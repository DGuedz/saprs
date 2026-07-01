<div align="center">

# SAPRS

### Solana Agent Policy Runtime Specification

**The missing layer between agent identity and agent payment.**

<br/>

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)
[![Status](https://img.shields.io/badge/Status-Draft%20v0.1.0-orange.svg)]()
[![Solana](https://img.shields.io/badge/Chain-Solana-9945FF?logo=solana&logoColor=white)](https://solana.com)
[![x402](https://img.shields.io/badge/Compatible-x402-blue)](https://x402.org)
[![AG9](https://img.shields.io/badge/Compatible-AG9%20Identity-green)](https://ag9.xyz)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

<br/>

[Read the Spec](specs/SAPRS-001.md) · [Reference Implementation](https://mindprotocol.xyz) · [Contribute](CONTRIBUTING.md) · [Discuss](https://github.com/DGuedz/saprs/issues)

</div>

---

## Why This Exists

Solana's agentic economy reached **$850M in transaction volume** in 2026 — projected **$10B by 2027**.  
65% of all agentic AI payments already run on Solana.

Two foundational primitives exist. One does not.

| Layer | Standard | Status |
|---|---|---|
| Agent Identity | AG9 / Know Your Agent / SATI | ✅ Solved |
| **Policy & Authorization** | — | ❌ Missing |
| Payment Rails | x402 / Pay.sh / MPP | ✅ Solved |
| **Proof of Execution** | — | ❌ Missing |
| Institutional Compliance | Open Transaction Layer | 🔄 Forming |

An agent can prove **who it is** and **move money**.  
It cannot prove **what it was authorized to spend** — or that any rule was applied before the payment fired.

At this scale, undocumented agent spending is a systemic risk vector.  
SAPRS closes that gap.

---

## What SAPRS Defines

SAPRS is an open specification — not a product — that any Solana project can implement.

```
AG9 / Know Your Agent         →  who is the agent
             ↓
  SAPRS Policy Registry       →  what is it authorized to do
             ↓
  SAPRS Policy Runtime        →  was the rule applied?
             ↓
  SAPRS Proof of Execution    →  cryptographic evidence
             ↓
x402 / Pay.sh / MPP           →  payment executes
             ↓
Open Transaction Layer        →  institutional compliance
```

**Five components:**

| Component | What it does |
|---|---|
| **Intent Schema** | Structured declaration of what an agent wants to do before it acts |
| **Policy Schema** | Versioned, on-chain rule set: allowed actions, hard limits, deny conditions |
| **Policy Evaluation Flow** | Runtime that validates intent against policy before x402 fires |
| **Proof of Execution (PoE)** | Cryptographic artifact proving a policy governed a specific action |
| **On-Chain Policy Registry** | Solana PDAs where policies live — immutable, auditable, trustless |

---

## Quick Example

**Declare an intent:**
```json
{
  "saprs_version": "0.1.0",
  "intent_id": "uuid-v4",
  "agent_id": "ag9:solana:<public_key>",
  "action": "call_api",
  "amount_usdc": 0.0120,
  "policy_account": "<solana_pda>",
  "timestamp": "2026-07-01T00:00:00Z"
}
```

**Register a policy:**
```yaml
saprs_version: "0.1.0"
allowed_intents:
  - call_api
  - purchase_compute
limits:
  max_amount_usdc_per_call: 0.05
  max_amount_usdc_per_day: 10.00
human_approval:
  required_above_usdc: 1.00
proof:
  type: saprs_poe
  on_chain_anchor: true
```

**Receive proof:**
```json
{
  "poe_id": "uuid",
  "result": "ALLOW",
  "policy_hash": "sha256:...",
  "intent_hash": "sha256:...",
  "delivery_hash": "sha256:...",
  "solana_anchor": "<tx_signature>"
}
```

→ Full specification: [specs/SAPRS-001.md](specs/SAPRS-001.md)

---

## Reference Implementation

[**MIND Protocol**](https://mindprotocol.xyz) is the reference implementation of SAPRS-001 — in production today.

| Component | Implementation | Status |
|---|---|---|
| Policy Runtime | `mind-policy-runtime` | Production |
| Proof of Execution | Mindprint | Production |
| On-chain anchor | Solana devnet | Active |
| Mainnet | Solana mainnet | Q3 2026 |
| MCP endpoint | `https://api.mindprotocol.xyz/mcp` | Live |

```bash
# Try the reference implementation
npx -y @mind_protocol/mcp-server@latest
```

---

## Adopt SAPRS-001

```
1. Register your agent identity via AG9
2. Declare a policy document → specs/SAPRS-001.md#23-policy-schema
3. Register policy as a PDA on Solana
4. Emit PoE artifacts → specs/SAPRS-001.md#25-proof-of-execution
5. Optional: use MIND Protocol as managed runtime
```

Added your project? Open a PR to [ADOPTERS.md](ADOPTERS.md).

---

## Specification Roadmap

| Spec | Topic | Status |
|---|---|---|
| **SAPRS-001** | Core: Intent, Policy, PoE, Registry | 🟠 Draft |
| SAPRS-002 | Agent-to-agent delegation chains | 📋 Planned |
| SAPRS-003 | Cross-chain policy portability | 📋 Planned |
| SAPRS-004 | ML-based risk scoring integration | 📋 Planned |
| SAPRS-005 | ZK proof of policy evaluation | 📋 Planned |

---

## Path to Solana Foundation

| Phase | Milestone | Timeline |
|---|---|---|
| **Now** | Publish SAPRS-001 draft | July 2026 |
| Month 1 | 2–3 ecosystem projects adopt intent schema | Aug 2026 |
| Month 2 | Submit to Solana Foundation as ecosystem primitive | Sep 2026 |
| Month 3 | SIMD proposal for on-chain Policy Registry program | Oct 2026 |
| Q4 2026 | SAPRS as native Solana agent infrastructure | Dec 2026 |

---

## Contribute

This specification is open for ecosystem feedback. Governance happens in the open.

- **Issues** — feedback on the spec, edge cases, corrections
- **PRs** — proposed changes to schema or protocol
- **Discussions** — broader design questions

See [CONTRIBUTING.md](CONTRIBUTING.md) for the full process.

---

## Community

| Channel | Link |
|---|---|
| X / Twitter | [@Mind_Protocol](https://x.com/Mind_Protocol) |
| Website | [mindprotocol.xyz](https://mindprotocol.xyz) |
| Email | hello@mindprotocol.xyz |
| GitHub Issues | [DGuedz/saprs/issues](https://github.com/DGuedz/saprs/issues) |

---

<div align="center">

**Fork it. Build on it. Cite it.**  
The ecosystem wins when the standard exists.

<br/>

[CC BY 4.0](LICENSE) · Diego Guedes · MIND Protocol · 2026

</div>
