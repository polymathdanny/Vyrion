# Vyrion (VYRN) — exchange listing package

Fill in the `TBD` items before sending. Exchanges' application forms map
almost one-to-one onto these sections.

## 1. Asset profile

| Field | Value |
|---|---|
| Project name | Vyrion |
| Ticker | VYRN |
| Asset type | Native coin of the Vyrion blockchain (own Layer-1, proof of work) |
| Decimals | 8 |
| Max supply | 21,000,000 VYRN (consensus rule) |
| Initial / premine supply | **0**: fair launch, no premine, no presale, no team allocation |
| Circulating supply | = total minted (`getblockchaininfo` → `supply.minted`); no locked or vested allocations exist |
| Consensus / algorithm | Proof of work, **Scrypt** (N=1024, r=1, p=1), Litecoin-compatible (CPU, GPU, Scrypt ASIC) |
| Block time / difficulty | 60 s / LWMA, retargets every block |
| Emission | 4.99087383 VYRN per block in era 0, halving every 2,103,840 blocks (~4 years) |
| Reward split | 50% to the miner (+ all fees) from block 1. Launch period (first ~90 days): dev 10% + exchange liquidity 10%. Then dev 5% (to year 4), 2.5% (4–8), 0% after. Dev ≈ 3.3% and liquidity ≈ 0.3% of max supply in total, both taken from the AI-work reserve (minted only for verified AI work), never from miners |
| Signatures / addresses | Ed25519; Base58Check addresses starting with `V` |
| Use of the coin | AgentSpace marketplace (hire agents, jobs with on-chain escrow); paying on-chain AI agents per token (agents are registered on-chain, run on owners' own computers with open-source AI, and pay each other); agent registration fees; mining rewards; fees |
| Logo | `assets/vyrion-icon-512.png`, `vyrion-icon-256.png`, `vyrion-icon-128.png`; full logo `assets/vyrion-logo-full.jpg` |
| Website / whitepaper / socials | TBD |
| Source code | TBD (GitHub organization) |

## 2. Technical integration

| Item | Value |
|---|---|
| Node software | `vyriond` + `vyrion-cli` (Linux / macOS / Windows), bitcoind-style JSON-RPC |
| Model | Account-based (balance + nonce per address), not UTXO |
| Deposits | One address per customer (`getnewaddress`), confirmations via `gettransaction` / `listtransactions` |
| Withdrawals | `sendfrom` / `sendtoaddress`; `sendrawtransaction` for externally signed txs |
| Recommended confirmations | 30 (~30 minutes) |
| Fee | Market fee rate per byte (default minimum ≈ 0.000015 VYRN per transfer) |
| Memo | optional, up to 64 bytes |
| Explorer | built into the wallet (`listblocks`, `getblock`, `getaddressinfo`); public explorer TBD |
| Mining pools | built-in Stratum server in every node; standard Scrypt pool software needs a Vyrion coin adapter (TBD) |
| Integration guide | `docs/EXCHANGE_INTEGRATION.md` |

## 3. Supply verification (send these to the exchange)

1. Genesis block pays nobody: `getblock $(getblockhash 0) 2`.
2. `getblockchaininfo` → `supply` (minted, dev share paid, AI-work reserve), recomputed by each node from all blocks.
3. Consensus constants (cap, subsidy, split, dev address) in `node/internal/blockchain/params.go`.
4. Dev fund and liquidity addresses are public and fixed at genesis; balances visible with `getaddressinfo`. Nothing was created before block 1.

## 4. Security & legal

| Item | Status |
|---|---|
| Node / consensus code audit | TBD (required) |
| Agents (preview) audit before they go on-chain | TBD |
| Bug bounty | TBD |
| Legal opinion (token classification) in launch jurisdictions | TBD — required by most tier-1/2 exchanges |
| Team KYC with exchange | TBD |
| Company / foundation entity | TBD |

## 5. Market

| Item | Notes |
|---|---|
| Existing liquidity | TBD |
| Market maker | TBD (many exchanges require one) |
| Network hash rate / holders | `getnetworkhashps`, `getblockchaininfo` → `supply.accounts` |
| Community size | TBD |
| Listing fee / budget | TBD — varies widely by exchange |

## 6. Risk disclosures exchanges expect

- Early-stage network; consensus code under audit (state status).
- A new proof-of-work chain with low hash rate early on is more exposed to 51% attacks; exchanges should use deep confirmations at first.
- Proof of Inference (AI-work rewards) is a preview and not yet part of consensus.
- Not an investment product; no promises of returns are made by the project.

> Not legal or financial advice. Have counsel review all exchange
> representations before submission.
