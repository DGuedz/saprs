# SAPRS — Solana Agent Policy Runtime Specification

> The missing layer between agent identity and agent payment.

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)
[![Status: Draft](https://img.shields.io/badge/Status-Draft-orange.svg)]()
[![Version: 0.1.0](https://img.shields.io/badge/Version-0.1.0-blue.svg)]()

---

## The Problem

Solana's agentic economy has two of three foundational primitives:

| Layer | Standard | Status |
|---|---|---|
| Agent Identity | AG9 / Know Your Agent | Solved |
| **Policy & Authorization** | **???** | **Missing** |
| Payment | x402 / Pay.sh / MPP | Solved |
| **Proof of Execution** | **???** | **Missing** |
| Compliance | Open Transaction Layer | Forming |

An agent can prove *who it is* and *move money*. It cannot prove *what it was authorized to spend* or *that rules were actually applied*.

At $850M in agentic transaction volume today — projected $10B by 2027 — undocumented agent spending is a systemic risk. Policy runtime is not a feature. It is infrastructure.

---

## What SAPRS Defines

**1. Intent Schema** — a structured declaration of what an agent wants to do before it acts.

**2. Policy Schema** — a versioned, on-chain-registered rule set defining allowed actions, hard limits, and deny conditions.

**3. Policy Evaluation Flow** — the runtime that validates intent against policy before any x402 payment fires.

**4. Proof of Execution (PoE)** — a cryptographic artifact proving a policy was evaluated against an intent, producing a verifiable result, anchored on Solana.

**5. On-Chain Policy Registry** — PDAs on Solana where policies are registered immutably and referenced by agents at execution time.

---

## How It Fits

```
AG9 / Know Your Agent       →  who is the agent
           ↓
  SAPRS Policy Registry      →  what is it authorized to do
           ↓
  SAPRS Policy Runtime       →  was the rule applied?
           ↓
  SAPRS Proof of Execution   →  cryptographic evidence
           ↓
x402 / Pay.sh / MPP         →  payment executes
           ↓
Open Transaction Layer       →  institutional compliance
```

---

## Read the Specification

→ [specs/SAPRS-001.md](specs/SAPRS-001.md)

---

## Reference Implementation

[MIND Protocol](https://mindprotocol.xyz) provides the reference implementation of SAPRS-001:

- **Policy Runtime:** `mind-policy-runtime` (production)
- **Proof of Execution:** Mindprint (SAPRS-compatible PoE)
- **On-chain anchor:** Solana devnet → mainnet Q3 2026
- **MCP endpoint:** `https://api.mindprotocol.xyz/mcp`

Try it:
```bash
npx -y @mind_protocol/mcp-server@latest
```

---

## Roadmap

| Spec | Topic | Status |
|---|---|---|
| SAPRS-001 | Core: Intent, Policy, PoE, Registry | Draft |
| SAPRS-002 | Agent-to-agent delegation chains | Planned |
| SAPRS-003 | Cross-chain policy portability | Planned |
| SAPRS-004 | ML-based risk scoring integration | Planned |
| SAPRS-005 | ZK proof of policy evaluation | Planned |

---

## Adopt SAPRS-001

To implement SAPRS in your project:

1. Register your agent identity via [AG9](https://ag9.xyz)
2. Declare a policy document following [Section 2.3](specs/SAPRS-001.md#23-policy-schema)
3. Register the policy as a PDA on Solana
4. Emit PoE artifacts following [Section 2.5](specs/SAPRS-001.md#25-proof-of-execution-poe-standard)
5. Optional: use [MIND Protocol](https://mindprotocol.xyz) as managed runtime

---

## Contribute

This specification is open for ecosystem feedback.

- Open an issue or PR in this repo
- Tag [@MINDProtocol](https://x.com/Mind_Protocol) on X with feedback
- Email: `hello@mindprotocol.xyz`

**Fork it. Build on it. Cite it. The ecosystem wins when the standard exists.**

---

## Path to Solana Foundation

| Phase | Action | Timeline |
|---|---|---|
| Now | Publish SAPRS-001 draft | July 2026 |
| Month 1 | 2-3 ecosystem projects adopt intent schema | Aug 2026 |
| Month 2 | Submit to Solana Foundation as ecosystem primitive | Sep 2026 |
| Month 3 | SIMD proposal for on-chain Policy Registry program | Oct 2026 |
| Q4 2026 | SAPRS as native Solana agent infrastructure | Dec 2026 |

---

## License

[CC BY 4.0](LICENSE) — Diego Guedes, MIND Protocol, 2026.
