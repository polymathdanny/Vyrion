# VYRN tokenomics: fair launch

**One line:** no premine, no presale, no team allocation. Nothing exists
before block 1: every VYRN is created block by block as the chain is mined.
A dev share and a launch-liquidity share are part of each block's subsidy,
written into consensus at genesis, visible on-chain, and they never reduce
what miners get.

## Supply

| | |
|---|---|
| Max supply | 21,000,000 VYRN (8 decimals), enforced by every node |
| Genesis supply | **0**: the genesis block pays nobody |
| Proof of work | Scrypt (N=1024, r=1, p=1), Litecoin-compatible |
| Block time | 60 seconds (LWMA difficulty, retargets every block) |
| Block subsidy | 4.99087383 VYRN in era 0 |
| Halving | every 2,103,840 blocks (~4 years) |
| Emission | 50% of the cap in years 0–4, 75% by year 8, approaching 100% |
| Coinbase maturity | 100 blocks (~1 h 40 min) |
| Fee | Market fee rate per byte, paid to the block's miner (default relay minimum 0.0001 VYRN/kB ≈ 0.000015 VYRN per transfer). Miners fill blocks highest rate first |

## Where each block's subsidy goes

| Share | Launch (first 129,600 blocks, ~90 days) | Rest of era 0 (to year 4) | Era 1 (yrs 4–8) | Era 2+ |
|---|---|---|---|---|
| Miner who found the block (plus all fees) | 50% | 50% | 50% | 50% |
| Dev fund | 10% | 5% | 2.5% | **0%** |
| Exchange liquidity | 10% | 0% | 0% | 0% |
| AI-work reserve (Proof of Inference) | 30% | 45% | 47.5% | 50% |

- **The dev share** goes to one fixed address set in the genesis parameters and
  is subject to the same 100-block maturity as mining rewards. Over the whole
  emission it is about **689,000 VYRN (~3.3% of max supply)**, earned block by
  block over 8 years, never as a lump sum. For comparison, Decred and Dash
  route ~10% of every block to a treasury forever.
- **The launch-liquidity share** goes to a second fixed address for the first
  ~90 days only: about **64,700 VYRN (~0.3% of max supply)**, roughly 720 VYRN
  a day, to seed exchange and DEX liquidity. It is earned by the chain running,
  not created up front, so nobody holds coins before miners do.
- Dev and liquidity together: about **753,000 VYRN (~3.6%)**, all of it taken
  from the AI-work reserve, so miners receive exactly 50% of every block from
  block 1.
- **The AI-work reserve** is *not* minted with the block. It accrues as an
  allowance tracked by consensus and is only created when Proof-of-Inference
  work (AI inference for agents, checked by validators) is verified on-chain.
  Until that ships, it is simply not created, so real supply stays below the
  schedule. Anything never earned is never minted.

## AgentSpace fees (ongoing dev funding)

AgentSpace is the on-chain marketplace for AI agents. Its fees go to the same
public dev fund address as the block share and are enforced by consensus
(every node checks them; no server is involved):

| Action | Fee to the dev fund |
|---|---|
| List an agent for hire | 1 VYRN, once (updating the listing is free) |
| Post a job | 0.5 VYRN |
| Every payment to an agent (chat credit) | 2% of the payment |
| Every job payout (escrow released or claimed) | 2% of the payout |

Bids, reviews, delivery and refunds pay only the normal miner fee. The block
dev share ends after ~8 years; AgentSpace fees continue for as long as the
marketplace is used, so developer funding grows with real usage instead of
with inflation. Totals are visible on-chain (`getagentspace` →
`dev_fund_received`).

## Rules enforced in code (`node/internal/blockchain`)

- A coinbase may claim at most the miner share plus fees; blocks claiming more
  are rejected by every node.
- The dev and liquidity shares are credited by consensus to genesis-fixed
  addresses, with the same 100-block maturity as mining rewards; there is no
  other way to create coins.
- Total minted can never exceed 21,000,000 VYRN.
- Every node can check supply: `vyrion-cli getblockchaininfo` reports minted
  coins, dev share paid and the AI-work reserve, recomputed from the blocks.

## Launching without a premine

Mining needs no stake and no permission: anyone can mine from block 1 with a
CPU, GPU or Scrypt ASIC. The first miners receive the first coins. The team
mines on the same terms as everyone else.

## Not financial or legal advice

This describes a mechanism, not an investment. Mining rewards and dev funds can
have tax and securities implications depending on jurisdiction; get a crypto
lawyer's review before launch.
