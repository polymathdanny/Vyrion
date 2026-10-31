# SOC 2 and Vyrion: what is possible

## Short answer

SOC 2 is not a certificate for software or a network. It is an independent
CPA firm's report on an **organisation's** controls, measured against the AICPA
Trust Services Criteria (Security is required; Availability, Processing
Integrity, Confidentiality and Privacy are optional). So:

- **The Vyrion network as a whole cannot be "SOC 2 compliant".** It has no single
  operator: thousands of independent people run nodes and host agents.
- **The organisation behind Vyrion can get a SOC 2 report** covering what it
  operates: release builds and downloads, seed nodes, the website, support, and
  any **managed service** it offers.
- **A managed "Vyrion for Business" agent hosting service** (agents run by the
  company for business customers, with the same software) is the natural thing
  to put in scope. That is what enterprise buyers would ask a SOC 2 report for.
- **Independent agent owners** are outside any Vyrion report. A business that
  hosts its own agents can include them in its own SOC 2 scope; the software's
  controls below help it pass.

## What the software already provides (maps to the criteria)

| Criteria | In the software (0.7.0) |
|---|---|
| CC6 Logical access | Wallet passphrase encryption (PBKDF2 + AES-GCM), RPC cookie/password auth, localhost-only RPC and wallet UI, CSRF/DNS-rebinding protection, data folder locked to the account, signed and replay-protected agent requests |
| CC6.7 Data in transit | End-to-end encrypted agent chats; signed key exchange tied to on-chain identity |
| CC7 System operations | Structured logs, crash-safe state files with automatic recovery, single-instance lock, reachability monitoring |
| CC8 Change management | Versioned releases, reproducible builds with SHA256SUMS, automated tests (unit, race detector, fuzzing, multi-node end-to-end) |
| Confidentiality (C1) | Private-by-default documents, conversation isolation, redaction, retention and deletion |
| Privacy (P-series) | Notice (signed data cards), choice (owner settings), retention limits, "forget me" deletion, collection minimisation (nothing sent to the developers), access by data subjects to their own conversation only |
| Processing integrity (PI1) | Consensus-enforced payments, escrow and fees; per-token metering by the model's own tokenizer |

## What the organisation must add (not software)

- A legal entity and named owners for security and privacy.
- Written policies: information security, access control, acceptable use,
  incident response, business continuity / disaster recovery, vendor management,
  data retention, change management, risk assessment.
- People controls: background checks, security training, onboarding/offboarding.
- Device and account controls for staff: MFA everywhere, disk encryption, MDM,
  quarterly access reviews.
- For anything it operates (seed nodes, managed hosting): monitoring and
  alerting, backups with restore tests, vulnerability scanning, logging retention.
- An annual independent penetration test, and the independent code audit
  already planned before exchange listings.
- Evidence collection over time (a compliance platform automates most of it).

## Typical path and budget (check current quotes)

Industry guides for 2026 put a SOC 2 Type 1 at roughly **$15k–$40k all-in**, a
Type 2 at roughly **$30k–$80k+**, and compliance-automation platforms at
**$10k–$30k a year**; a Type 2 needs an observation window of at least 3 months
(6 is common) after about a month of readiness work, and it is renewed yearly.
These are published estimates, not quotes.

1. Decide the scope (recommended: the organisation + a managed agent hosting service).
2. Readiness: policies, controls, compliance platform, gap assessment.
3. Type 1 report (controls designed correctly at a point in time).
4. Type 2 report after the observation window (controls worked over time).

Until then, the honest claim is: *"Agents run on their owners' machines;
conversations are end-to-end encrypted, isolated per person and never sent to
the Vyrion developers; owners control retention and deletion. SOC 2 for a
managed business offering is on the roadmap."*
