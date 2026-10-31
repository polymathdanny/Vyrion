# Privacy and data protection in Vyrion agents

Vyrion agents run on their owners' own computers. Nobody (not the Vyrion
developers, not miners, not relay nodes) receives the documents an agent is
trained on or the conversations people have with it. This page lists what the
software guarantees, what each agent owner controls, and what stays the owner's
responsibility.

## What the software does for everyone (always on)

| Protection | How |
|---|---|
| **Conversations are isolated** | The AI only ever sees the memory of the person it is talking to. The owner's chats, the owner's private notes and other visitors' conversations are never placed in the model's context for someone else. |
| **End-to-end encryption between nodes** | Messages to an agent on another node are encrypted (X25519 + HKDF-SHA256 + AES-256-GCM) to a key the host signs with the agent's on-chain wallet key. Relays and anyone on the network see only ciphertext; a substituted key is detected and the message is not sent. |
| **Private by default documents** | Documents added to an agent's knowledge base are private (used only when the owner chats) unless the owner marks them "for customers". |
| **Personal data scanning** | Every document is scanned for emails, phone numbers, card numbers, national ids, IBANs, API keys, private keys and passwords. A document containing personal data cannot be used for customers until it is redacted. |
| **Redaction of replies** | Replies to anyone but the owner are scrubbed of personal data (on by default). |
| **Learning only from the owner** | "Learn from my conversations" uses only the owner's own chats, never other people's (who could otherwise poison or extract it). |
| **Retention limits and "forget me"** | Owners choose whether conversations with others are kept and for how long (default 30 days). Anyone can send a signed "forget me" request; the host deletes everything it remembers about that address. |
| **Signed data cards** | Each agent publishes what it is trained on (titles, sizes, SHA-256 fingerprints and licences of customer documents; private documents only counted), which model it uses, and how it treats your data, signed by the agent's on-chain wallet key. The wallet shows it on the agent's profile and flags any card that does not verify. |
| **Local storage only** | Knowledge files live in the owner's data folder (`agents/<id>/knowledge`), which the Windows installer locks to the owner's account; files are written crash-safely. |

## What the owner controls (Train an Agent page)

- Which documents customers may benefit from; redaction; declared licence.
- Learning from their own conversations; teaching facts; using their own model.
- Keeping visitor conversations, retention days, reply redaction.
- Deleting a person's conversations (`agentforget <agent> <address>`).

## Limits, stated plainly

- **The owner can read conversations with their agent.** It runs on their computer. The data card says so; don't send an agent something you would not tell its owner.
- **A data card is a signed declaration, not a proof.** The signature proves which agent (and so which owner) made the statement; it cannot prove a model was not trained on something else. Fingerprints let anyone who has a document check it is the one listed.
- **Pattern-based redaction catches common personal data, not all of it** (for example, names alone or free-text addresses).
- **Prompt-injection is reduced, not eliminated.** Visitors are treated as untrusted and the model is told never to reveal personal or private data, but models can be tricked. Keep secrets out of customer documents; private documents never reach visitors' conversations at all.
- **Disk encryption** is the operating system's job: turn on BitLocker / FileVault on computers hosting agents.
- **Payments are public** on the blockchain (addresses, amounts, which agent was paid); message content never is.

## Legal roles

An agent's owner decides what their agent does with people's data, which
under laws such as the GDPR makes them the data controller for it. The
software gives them the tools (minimisation, isolation, retention, deletion,
encryption, redaction, transparency); compliance with the laws that apply to
them remains their responsibility. Not legal advice.
