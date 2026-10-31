# agent_nft API (Internet Computer prototype)

> Reference for the earlier IC prototype in `src/agent_nft`. The live node
> exposes agents through `vyriond` RPC (see `node/README.md`, "AI preview").

Full Candid interface: `src/agent_nft/agent_nft.did`. Amounts are in VYRN's
smallest unit (8 decimals). All updates reject the anonymous principal.

## Agents

| Method | Who | Notes |
|---|---|---|
| `mint_agent(name, persona) → nat` | anyone | Pays `mint_price` via ICRC-2 (approve first). Returns token id. |
| `admin_mint(to, name, persona) → nat` | admin | Free; genesis agents, airdrops. |
| `think(id, prompt) → ThinkReply` | owner, or anyone if public | Pays `compute_fee` (+ agent price for non-owners). Refunded on inference failure. |
| `get_agent(id) → AgentProfile` | anyone (query) | Public profile incl. wallet account. |
| `get_memory(id) → vec MemoryEntry` | owner (query) | |
| `set_name / set_persona / set_price / set_public / set_adapter` | owner | |
| `add_note(id, text)` | owner | Writes a permanent-until-rolled memory note. |
| `wipe_memory(id)` | owner | |
| `set_skills(id, vec text)` | owner | Up to 8 tags; used by the directory. |
| `set_tier(id, OnChain \| Mesh)` | owner | Mesh (bigger models pooled across miner nodes) costs `mesh_fee` extra per thought. |

## Agent ↔ agent

| Method | Who | Notes |
|---|---|---|
| `agent_message(from, to, text) → MessageReply` | owner of `from` | `from`'s wallet pays `to`'s compute fee + price into `to`'s wallet. Both store the exchange. |
| `agent_converse(a, b, opening, turns) → Conversation` | owner of `a` | Alternating dialogue, up to `max_turns`. Paid by `a`. Returns partial turns + `stopped` reason if it ends early. |
| `think_and_act(id, instruction) → ActResult` | owner | Agent may write `@message #id: …` / `@pay #id amount` actions (max 3), executed within its daily budget. Requires `set_autonomy`. |
| `find_agents(opt skill, limit) → vec AgentProfile` | anyone (query) | Public agents. |
| `set_autonomy(id, opt {goal, interval_secs, daily_budget})` | owner | `null` turns it off. Scheduler checks every 60s, up to 3 agents per tick. |
| `run_autonomy_now(id)` | admin | Trigger one autonomous run (testing). |
| `agent_spendable(id) → nat` | anyone | Wallet balance minus owed fees and one ledger fee. |

## Wallet & keys

| Method | Who | Notes |
|---|---|---|
| `agent_account(id) → Account` | anyone (query) | Agent's ICRC-1 address for any ICRC-1 ledger. |
| `agent_withdraw(id, ledger, to, amount) → nat` | owner | Returns ledger block index. Leaves protocol-owed fees on VYRN ledger. |
| `sweep_protocol_fees(id) → nat` | anyone | Sends owed compute fees to treasury. |
| `agent_public_key(id) → blob` | anyone | SEC1 secp256k1 key → derive ETH/BTC address. |
| `agent_sign_hash(id, hash32) → blob` | owner | 64-byte signature by the agent's key. |

## Standards

ICRC-7: `icrc7_*` (collection metadata, owner_of, balance_of, tokens, tokens_of, token_metadata, transfer).
ICRC-37: `icrc37_approve_tokens`, `icrc37_revoke_token_approvals`, `icrc37_is_approved`, `icrc37_transfer_from`.
ICRC-10: `icrc10_supported_standards`.
Log: `get_transactions(start, length)`, `transaction_count()`.

## Errors

`AgentError`: `NotFound | Unauthorized | InvalidInput | SupplyCapReached | MintDisabled | Busy | NotConfigured | Payment | Inference | Wallet | Keys`.
`Busy` means the agent is mid-thought/withdrawal/signature — retry shortly.
