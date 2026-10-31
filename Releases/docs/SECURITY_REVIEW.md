# Vyrion node: internal security review

**Date:** 25 Oct 2026 · **Version reviewed:** vyriond 0.3.3 (Go, `node/`) ·
**Reviewer:** Claude (AI-assisted internal review)

> This is an internal, AI-assisted code review. It is **not** a substitute for an
> independent third-party audit. Exchanges and investors will ask for one;
> commission it after the fixes below are in.

## Verdict

**Update, 25 Oct 2026 (vyriond 0.3.5): all code findings are fixed**
(F-8, coinbase maturity 100, shipped in 0.3.5 with a testnet restart). Each fixed
High/Medium finding now has a regression test that reproduces the original
attack and must stay passing. The fixes do not change which blocks are valid,
so 0.3.3 and 0.3.4 nodes stay on the same chain (tested), and testnet keeps
running without a restart.

Original verdict (0.3.3): not ready for mainnet value until 2 High and 4
Medium findings were fixed; the consensus rules, supply cap, key handling,
wallet encryption and RPC/UI protections held up.

## Scope note for 0.8.0

0.8.0 adds in-wallet software updates. Nodes only accept a release whose
manifest (`vyrion-update.json`) carries an ed25519 signature from a key
embedded in the node; every download must match the sha256 in that signed
manifest; archives are unpacked with path-traversal checks into the data
folder's `updates/` area; the new program must run and report the expected
version before anything is replaced; wallet and agent files are copied to
`backups/` first; the old programs are kept for `vyriond -rollback`. Required
(consensus) updates only pause mining and sending on the networks and heights
named in the signed manifest. The main risk moves to the release signing key:
whoever holds it can ship code to every node that installs updates, so it must
be kept offline (docs/RELEASING.md). Tested (forged and altered manifests,
tampered downloads, unsafe archive paths, a real running node updating,
restarting, halting and rolling back), not yet independently reviewed.

## Scope note for 0.7.0

0.7.0 adds privacy protections: per-person conversation isolation (a fix: until
0.6.x a visitor's request could reach the model together with the owner's and
other visitors' recent messages), end-to-end encrypted agent chats (X25519,
HKDF-SHA256, AES-256-GCM; host key signed by the agent's on-chain wallet key),
private-by-default knowledge documents, pattern-based personal-data detection
and redaction, retention limits, signed "forget me" requests and signed data
cards. Tested (wire capture shows only ciphertext; key substitution and card
tampering detected; isolation and redaction tests), not yet independently
reviewed. See docs/PRIVACY.md.

## Scope note for 0.6.0

0.6.0 adds AgentSpace, a consensus-level marketplace (transaction kind 3:
listings, jobs, bids, escrow, delivery, disputes, reviews, payments with a 2%
dev-fund share) and per-agent local training (knowledge base, own model URL).
Block validation was refactored into simulate + commit so block templates can
drop transactions that conflict inside one block. Covered by consensus tests
(escrow conservation, review eligibility, reorg undo vs. full rebuild,
template conflicts), agent tests and a two-node end-to-end test, but **not yet
reviewed line by line**; include it in the independent audit. Escrow has no
arbiter by design: a dispute freezes funds until one party gives way.

## Scope note for 0.5.0

0.5.0 makes fees a market (no fixed fee in consensus; per-byte relay minimum,
fee-rate block selection, mempool eviction, `estimatesmartfee`), prices agents
per token with prepaid, refundable credit and demand-based pricing, and adds
NAT traversal (UPnP/NAT-PMP, peer dial-back check, relay nodes). These have
unit, race and multi-node end-to-end tests but **no line-by-line review yet**;
include them in the independent audit. Credit held by an agent is trusted to
its host (like a prepaid account), so amounts should stay small.

## Scope note for 0.4.0

0.4.0 adds the on-chain agent registry (transaction kind 2), the agent runtime
(paid cross-node chat, agent-to-agent payments, autonomy) and the local AI
engine installer. These were built after this review, with security tests
(signed and replay-protected chat requests, payment matching, rate limits,
zip-slip and checksum checks, reorg undo of registry changes), but they have
**not** had the line-by-line review above. Include them in the independent audit.

## Remediation status (0.3.4)

| ID | Status | What changed | Regression test |
|---|---|---|---|
| F-1 | **Fixed** | Duplicate transactions are rejected in `checkBody`, before a block is cached, so a mutated copy can no longer poison the real block's id | `TestAuditMerkleMutation` |
| F-2 | **Fixed** | Mempool keeps a per-sender, nonce-ordered index; signatures are checked once on entry; revalidation is linear. 4,000 pending txs: 3.0 s → 1.2 ms. Max 64 pending txs per sender | `TestAuditMempoolQuadratic` |
| F-3 | **Fixed** | Blocks forking more than 100 blocks below the tip are refused; checkpoint support added (`Params.Checkpoints`) | `TestAuditDeepForkStored` |
| F-4 | **Fixed** | Orphan-triggered sync only contacts peers already in the peer list; one sync per peer at a time | `TestAuditPeerHeaderSSRF` |
| F-5 | **Fixed** | Stratum: 1,024 connections total, 64 per IP; a miner sending more than 32 invalid shares (and more than twice its valid ones) is disconnected and its IP banned for 10 minutes. Stale shares never count | `TestAuditShareFloodCost`, `TestConnectionCapPerIP` |
| F-6 | **Fixed** | Reorgs undo blocks back to the fork point and apply the new branch (undo data kept for the last 100 blocks); a failed reorg restores the old chain. Blocks are read from `blocks.dat` on demand instead of being held in RAM. Startup replay of the node's own file skips re-hashing proof of work | `TestReorgUndoMatchesRebuild`, `TestReorgInvalidBranchRestores` |
| F-7 | **Fixed** | Read, write and idle timeouts on the RPC and P2P servers; at most 8 bulk block downloads served at once | — |
| F-8 | **Fixed (0.3.5)** | Coinbase maturity raised to 100 blocks, as Bitcoin and Litecoin; testnet restarted with a new genesis block | chain tests |
| F-9 | **Fixed** | Replace-by-fee needs +10% and at least another minimum relay fee (0.5.0: fee rates) | `TestMempoolFeeMarket` |
| F-10 | **Fixed** | Stratum extranonce1 = random base + per-connection counter (unique) | — |
| F-11 | Open (release step) | Build releases with the newest Go patch release and run `govulncheck` | — |

Also fixed in 0.3.4 (agents preview, reported during testing): chat works
without a model server (clearly labelled automatic reply instead of an error);
"Let it act" works before autonomy is configured (default 1-credit daily
budget); creating an agent tops up free preview credits automatically, and
messages say "credits" instead of "VYRN".

## Scope and method

| | |
|---|---|
| Code | 35 Go source files, 9,575 lines + 1,818 lines of tests; wallet UI (935 lines JS) |
| Components | consensus (`internal/blockchain`), sync/relay/RPC (`internal/chainnode`), Stratum (`internal/stratum`), wallet (`internal/wallet`), RPC server, wallet UI server, daemon wiring, Windows installer |
| Out of scope | AI-agents preview (devnet, no real value), the Internet Computer prototype (`src/`), third-party miners |
| Method | line-by-line manual review of consensus, P2P and Stratum paths; threat model (remote peer, Stratum client, malicious web page, local attacker); proof-of-concept tests; Go race detector over the full suite; 60 s fuzzing of the block and transaction decoders (~3 million inputs, no crashes) |
| Not run | `govulncheck` (no network access to the vulnerability database here) |

## Findings

| ID | Severity | Finding | Location | PoC |
|---|---|---|---|---|
| F-1 | **High** | Merkle-tree mutation (CVE-2012-2459 class) splits the chain | `blockchain/chain.go:333` (`checkBody`), `:368` (`process`) | Reproduced |
| F-2 | **High** | Quadratic mempool revalidation: cheap CPU denial of service on every node | `blockchain/mempool.go:28,41,114` | Reproduced |
| F-3 | Medium | Deep low-difficulty forks are accepted and stored (memory/disk exhaustion) | `blockchain/chain.go:368` | Reproduced |
| F-4 | Medium | Server-side request forgery through the `X-Vyrion-Peer` header | `chainnode/service.go:170` | Reproduced |
| F-5 | Medium | Stratum: bogus shares cost a full scrypt hash each; no connection cap or ban | `stratum/stratum.go:151,479` | Reproduced |
| F-6 | Medium | Every reorg replays the whole chain from genesis; all blocks kept in RAM | `blockchain/chain.go:418` | Code review |
| F-7 | Low | HTTP servers set only a header timeout; `getblocks` returns up to 32 MB per request | `cmd/vyriond/main.go:285`, `chainnode/service.go` | Code review |
| F-8 | Low | Coinbase maturity of 20 blocks is shallow for a young chain | `blockchain/params.go:33` | Code review |
| F-9 | Low | Replace-by-fee needs only +1 base unit, so replacements can be spammed | `blockchain/mempool.go:76` | Code review |
| F-10 | Low | Stratum extranonce1 can collide between miners (wasted work) | `stratum/stratum.go:236` | Code review |
| F-11 | Low | Built with Go 1.24.7; rebuild with the latest Go patch release | toolchain | n/a |
| O-1 | Ops | Dev-fund key is in the everyday node wallet on an online PC; wallets are unencrypted until the user encrypts them | operations | n/a |
| O-2 | Ops | A new PoW chain has little hash rate behind it at launch (51% risk) | launch | n/a |

### F-1 (High): merkle mutation splits the chain

Bitcoin-style merkle trees duplicate the last hash on odd levels, so a block
with its last transaction repeated has the **same merkle root and header
hash** as the honest block. `checkBody` does not reject duplicate
transactions; the duplicate is only caught later in `applyBlock`, which marks
that header hash invalid. When the honest block arrives it is refused as
"block already known".

*Impact:* anyone who relays a mutated copy of a fresh block first can make
nodes permanently reject the real block and fork off the network.
*PoC:* `TestAuditMerkleMutation`: honest block rejected with
`block already known`.
*Fix:* reject duplicate transaction ids in `checkBody` (before the block is
cached), as Bitcoin Core does. About 10 lines.

### F-2 (High): quadratic mempool revalidation

`Mempool.Add` rebuilds the per-sender index over the whole pool each call, and
`Revalidate` (run after every block) re-adds every pending transaction,
re-verifying each signature. Measured: **3.0 s for 4,000 pending
transactions**; at the 50,000-transaction cap this grows to minutes per block
(quadratic extrapolation). Filling the pool costs an attacker only normal fees.

*Impact:* nodes and miners stall after each block; Stratum work and sync fall behind.
*Fix:* keep a persistent per-sender index, skip signature re-checks for
transactions already verified, and revalidate only senders touched by the new
block. About 60 lines.

### F-3 (Medium): deep low-difficulty forks are stored

Any block that extends any known block at its expected difficulty is stored,
however far below the tip it forks. Forking near genesis needs only the
minimum difficulty (~1 million scrypt hashes per block on mainnet), and each
such block can carry up to 1 MB of signed-but-unfunded transfers. All of it is
kept on disk and in RAM.
*PoC:* `TestAuditDeepForkStored`: a fork from genesis is stored while the tip is at 50.
*Fix:* refuse blocks whose fork point is more than a set depth (e.g. 100
blocks) below the tip, and add checkpoints at releases. About 15 lines.

### F-4 (Medium): SSRF via `X-Vyrion-Peer`

When a POSTed block is an orphan, the node syncs from whatever URL the sender
put in the `X-Vyrion-Peer` header, so anyone on the internet can make the
node send requests into its local network.
*PoC:* `TestAuditPeerHeaderSSRF`: the node requested `GET /p2p/chain/tip` from an attacker-chosen server.
*Fix:* sync only from peers already in the peer list. About 5 lines.

### F-5 (Medium): Stratum share flood

Each submitted share is assembled and scrypt-hashed before it can be rejected
(~0.5 ms of CPU). One connection kept a core busy (2,000 bogus shares/s), and
there is no connection cap, rate limit or ban. The installer opens port 3333.
*Fix:* cap connections (total and per IP), disconnect after repeated invalid
shares, and apply cheap checks (job, ntime, duplicates) first. About 30 lines.

### F-6 (Medium): full-chain replay on reorg, all blocks in RAM

Any reorg, including the routine one-block races between miners, rebuilds the
state from genesis and re-verifies every signature in the chain. All raw
blocks also stay in memory. Fine at today's size, but cost grows with the chain.
*Fix:* undo/redo only the blocks between the fork point and the new tip, and
read old blocks from disk instead of RAM.

### Low and operational

- **F-7:** add read/write/idle timeouts to the RPC and P2P HTTP servers, and rate-limit `getblocks`.
- **F-8:** consider maturity of 100 blocks (Bitcoin/Litecoin) before launch; it is a consensus change, so decide before 24 Nov.
- **F-9:** require replacements to raise the fee meaningfully (e.g. +10% and at least the minimum fee).
- **F-10:** derive extranonce1 from a per-server counter only, so it is unique.
- **F-11:** rebuild releases with the newest Go patch release and run `govulncheck` in CI.
- **O-1:** encrypt both fund wallets, keep offline backups, move the dev fund out of the daily node wallet.
- **O-2:** ask exchanges for deep confirmations (30+) at first, and publish the launch time well ahead so the start is fair.

## What passed

| Area | Result |
|---|---|
| Scrypt PoW | Matches RFC 7914 test vectors; verified against the real `cpuminer` |
| Header checks | Difficulty bits must match LWMA; median-time-past and +5 min future limit enforced |
| Supply | Coinbase cannot claim more than miner share + fees; dev/liquidity shares fixed by consensus; 21M cap checked on every block |
| Transactions | Ed25519 with strict key/signature lengths; signature covers the network id (no cross-network replay); per-address nonces (no replay); amount/fee overflow bounded |
| Decoders | Length-checked reader; 60 s fuzzing (~3 million inputs), no panics |
| Reorg correctness | Reorg and persistence tests pass; state after reorg matches the winning chain |
| Concurrency | Full test suite clean under the Go race detector |
| RPC | Localhost by default; cookie or password auth with constant-time comparison; remote bind refused without a password |
| Wallet UI | Localhost-only Host check (blocks DNS rebinding); one-time login token; HttpOnly + SameSite=Strict session; custom-header CSRF guard; strict Content-Security-Policy; all untrusted text escaped |
| Wallet | PBKDF2-SHA256 (210,000 rounds) + AES-256-GCM; auto-relock; per-address keys from one seed; data folder locked to the user on Windows |
