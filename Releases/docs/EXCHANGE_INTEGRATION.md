# Exchange & wallet integration

Vyrion is its own proof-of-work chain. The node, `vyriond`, speaks
bitcoind-style JSON-RPC, so the usual exchange workflow (one deposit address
per customer, poll for confirmations, send withdrawals from a hot wallet)
carries over almost unchanged. The main difference from Bitcoin is that Vyrion
uses **accounts** (balance + nonce per address) instead of UTXOs.

## Running a node

```bash
vyriond -network main -rpcuser exchange -rpcpassword <long-random> -rpcbind 127.0.0.1
```

Keep RPC on localhost or a private network. Data: `blocks-main-<id>/`, `wallet.dat`
(back it up; encrypt with `encryptwallet`).

## Facts for the integration team

| Item | Value |
|---|---|
| Ticker / decimals | VYRN / 8 (1 VYRN = 100,000,000 base units) |
| Address format | Base58Check, version byte 70 → starts with `V` (testnet `t`) |
| Block time | 60 s |
| Recommended confirmations | 30 deposits (about 30 minutes); mined coins mature after 100 blocks |
| Fee | Market fee rate per byte (bitcoind style). Default relay minimum 0.0001 VYRN/kB (≈ 0.000015 VYRN for a plain transfer); `estimatesmartfee`, `settxfee`, `paytxfee=` work as in bitcoind. Not a consensus rule |
| Memo | optional, up to 64 bytes, public |
| Tx replay protection | network id in every tx + per-address nonce |
| Reorgs | possible (most-work chain), as with any PoW chain |
| Ports | P2P 9737, RPC 9736 |

## Deposits

1. `getnewaddress "customer-123"` → a `V…` address per customer (each has its
   own key, all derived from the wallet seed).
2. Poll `listtransactions 1000` (category `receive`) or, per address,
   `getaddresstxids <address>` and `gettransaction <txid>`.
3. Credit when `confirmations >= 30`. Re-check before crediting large amounts:
   `confirmations` drops to 0 or the tx disappears if a reorg removes it.

## Withdrawals

- `sendfrom <hot-wallet-address> <customer-address> <amount> ( "memo" )`
  sends from one specific address (recommended for exchanges), or
  `sendtoaddress <address> <amount>` lets the wallet pick the source.
- The returned txid confirms in the next block in normal conditions.
- Several withdrawals from the same address in one block are fine: the wallet
  assigns consecutive nonces automatically.
- To raise the fee of a stuck tx, resend with the same nonce and a higher fee
  (`sendrawtransaction`; replace-by-fee).

## Sweeping deposits

Because balances live on addresses, move deposits to the hot wallet with
`sendfrom <deposit-address> <hot-address> <amount-minus-fee>`. Sweeping is
optional; `sendtoaddress` can also pay from several addresses at once (one
transaction per source address).

## Useful RPCs

| Need | Method |
|---|---|
| Height / sync | `getblockchaininfo` (`blocks`, `headers`, `initialblockdownload`) |
| Balance of any address | `getaddressinfo <address>` |
| Wallet balances | `getbalances` (spendable, immature, pending in/out) |
| Validate | `validateaddress <address>` |
| Tx by id | `gettransaction <txid>`, `getrawtransaction <txid> true` |
| Blocks | `getblockhash <height>`, `getblock <hash> 2` |
| Broadcast signed tx | `sendrawtransaction <hex>` |
| Supply | `getblockchaininfo` → `supply.minted` (circulating = minted; no locked allocations) |

## Supply verification for listing reviews

- Genesis pays nobody (`getblock <genesis-hash> 2`).
- `getblockchaininfo` → `supply` reports minted coins, dev share paid and the
  AI-work reserve, recomputed by the node from every block.
- Max supply 21,000,000 VYRN and the reward split are consensus rules in
  `node/internal/blockchain/params.go`.
