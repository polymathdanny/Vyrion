# Launching Vyrion: testnet now, mainnet next

Vyrion is its own blockchain, so "deploying" means running nodes. There are no
contracts to upload and no cycles or gas to buy. The network exists as soon as
two nodes that know each other are running.

## 1. Testnet (today)

Testnet is the default network (`network=test`). Coins have no value, so it is
safe for anyone to try mining, sending and the wallet.

**Seed nodes.** Run 2–3 always-on nodes with public IPs (any small VPS, 1 vCPU,
1 GB RAM is enough). On each (Linux):

```bash
tar xzf vyrion-0.3.0-linux-amd64.tar.gz && cd vyrion-0.3.0-linux-amd64
./vyriond -externalurl http://<this-server-ip>:9737 -addnode <other-seed-ip> -printtoconsole=false &
```

Open TCP **9737** (peers) and, if people will point miners at it, **3333**
(Stratum). For a permanent service use `contrib/systemd/vyriond.service`.

**Everyone else** installs the node (Windows: `install.bat`) and adds a seed:
`addnode=<seed-ip>` in `vyrion.conf`, or Peers → Add peer in the wallet. Then
Mining → Start mining, or point a Scrypt miner at a node:

```
minerd -a scrypt -o stratum+tcp://<node-ip>:3333 -u <your t... address> -p x
```

Check health from any node: `vyrion-cli getblockchaininfo`, `getpeerinfo`,
`getchaintips` (each peer's height).

## 2. Mainnet checklist

Mainnet launches **24 Nov 2026 00:00 UTC** (genesis block already mined).
Both genesis-fixed addresses are set in `node/internal/blockchain/params.go`:

| Role | Address | Wallet |
|---|---|---|
| Dev fund (10% for ~90 days, then 5% → 2.5% → 0) | `VXuG3Km3Y4PAx5FYHUUxY2QyTS8jwpQa75` | `%APPDATA%\Vyrion\wallet.dat` |
| Launch liquidity (10% of blocks 1–129,600) | `VHs8buiDnzX7htNPKdvDyczCwZGMCuRMHJ` | `E:\vyrion-liquidy\wallet.dat` |

1. **Protect both wallets:** encrypt them and keep copies of both `wallet.dat`
   files off this computer. Losing one loses those coins for good.
2. **To change an address before launch:** edit `DevAddress` / `LiquidityAddr`
   in the `Mainnet` block of `params.go` and rebuild (`build-windows.bat`).
   After launch they can never change.
3. **Optional: move the launch date.** If you change `GenesisTime` or
   `GenesisMessage`, re-mine the genesis block and record the new nonce:
   ```bash
   vyriond -minegenesis main     # prints nonce= and hash=; takes a few minutes
   ```
   Put the nonce into `GenesisNonce` and update the expected hash in
   `internal/blockchain/genesis_test.go`.
4. **Seed nodes:** add the seed IPs to the release notes (and later as
   built-in defaults) so new nodes find the network.
5. **Build and publish:** `make release VERSION=1.0.0` → zips/tarballs plus
   `SHA256SUMS` in `node/dist/`. Publish the source and checksums together.
6. **Launch:** seed nodes start with `network=main`; miners and wallets switch
   by setting `network=main` in `vyrion.conf`. Mainnet data lives in
   `blocks-main-<id>/`, separate from testnet.

## 3. Before real value is at stake

- Independent review of the consensus code (`node/internal/blockchain`) and
  the Stratum server.
- At least 3 seed nodes run by different people, in different places.
- A public block explorer and a status page (both can be built on the node's
  RPC: `listblocks`, `getblock`, `getaddressinfo`).
- Early on, ask exchanges for deep confirmations (30+), because a young
  proof-of-work chain has little hash rate behind it.

## 4. AI agents and Proof of Inference

Agents and AI-work rewards run as a preview inside each node. The AI-work
reserve already accrues in consensus (`getblockchaininfo` → `supply.poi_reserved`)
and is minted only once verified AI work becomes part of consensus in a later
release.
