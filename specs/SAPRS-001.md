# SAPRS-001: Solana Agent Policy Runtime Specification

**Version:** 0.1.0-draft
**Author:** Diego Guedes — MIND Protocol
**Date:** 2026-07-01
**Status:** Draft — Open for ecosystem feedback
**License:** CC BY 4.0

---

## Abstract

Solana's agentic economy has solved two of three foundational primitives:

- **Identity** — AG9 / Know Your Agent / SATI
- **Payment** — x402, Pay.sh, Machine Payments Protocol

The third primitive — **policy authorization and proof of execution** — does not yet exist as a standard. Without it, autonomous agents can identify themselves and move money, but no one can verify *what they were authorized to do* or *prove that rules were followed*.

This specification defines SAPRS: a policy runtime standard for autonomous agents on Solana. It establishes a schema for declaring agent spending policies, a protocol for on-chain policy registration, and a proof standard for verifiable execution evidence.

Reference implementation: [MIND Protocol](https://mindprotocol.xyz)

---

## 1. Problem

### 1.1 The Missing Layer

The current Solana agent stack:

```
┌─────────────────────────────────────┐
│  AG9 / SATI — Agent Identity        │  ← solved
├─────────────────────────────────────┤
│  ??? — Policy & Authorization       │  ← missing
├─────────────────────────────────────┤
│  x402 / Pay.sh / MPP — Payment      │  ← solved
├─────────────────────────────────────┤
│  ??? — Proof of Execution           │  ← missing
├─────────────────────────────────────┤
│  Open Transaction Layer — Compliance│  ← forming
└─────────────────────────────────────┘
```

An agent today can prove *who it is* (identity) and *move money* (payment). It cannot prove *what it was authorized to spend*, *which rules governed that spending*, or *that those rules were actually applied*.

### 1.2 Why This Matters Now

**Regulatory pressure:** EU AI Act and emerging US frameworks are converging on accountability requirements for autonomous financial agents. Enterprises deploying agents via Pay.sh or x402 need a compliance artifact per transaction — not just a receipt.

**Institutional adoption:** State Street, JPMorgan, Western Union are now on Solana. Their compliance, risk, and legal teams will not approve agent spending without policy documentation and audit trail.

**Scale risk:** At $850M today and projected $10B by 2027 in agentic transaction volume, undocumented agent spending is a systemic risk vector. Policy runtime is not a feature — it is infrastructure.

### 1.3 What Breaks Without It

- An agent authorized for $5 API calls executes a $500 swap — no policy blocked it.
- An enterprise audits agent activity — no proof that rules were applied.
- A regulator asks what governed an agent's financial decision — no standard answer exists.
- An agent is compromised — no policy firewall between identity and payment rails.

---

## 2. Specification

### 2.1 Core Concepts

**Policy Runtime:** The execution layer that evaluates an agent's declared intent against a set of rules before any payment or action is authorized.

**Intent:** A structured declaration of what an agent wants to do, how much it wants to spend, and why.

**Policy:** A versioned, on-chain-registered rule set that defines allowed intents, hard limits, deny conditions, and proof requirements.

**Proof of Execution (PoE):** A cryptographic artifact proving that a specific policy was evaluated against a specific intent and produced a specific result — before the action was taken.

---

### 2.2 Intent Schema

Every agent action that involves payment or external state change MUST declare an intent.

```json
{
  "saprs_version": "0.1.0",
  "intent_id": "uuid-v4",
  "agent_id": "ag9:solana:<public_key>",
  "action": "call_api | execute_swap | purchase_compute | transfer",
  "amount_usdc": 0.0120,
  "recipient": "<solana_address_or_endpoint>",
  "policy_account": "<solana_pda>",
  "timestamp": "<iso8601>",
  "metadata": {
    "purpose": "fetch_wallet_intelligence",
    "session_id": "<optional>"
  }
}
```

**Required fields:** `saprs_version`, `intent_id`, `agent_id`, `action`, `amount_usdc`, `policy_account`, `timestamp`

---

### 2.3 Policy Schema

A policy is a versioned document declaring what an agent may and may not do.

```yaml
saprs_version: "0.1.0"
policy_id: "<uuid>"
agent_id: "ag9:solana:<public_key>"
version: "1.0.0"
registered_at: "<solana_tx_signature>"

# Allowed action types
allowed_intents:
  - call_api
  - purchase_compute
  - fetch_data

# Hard limits — enforced before any execution
limits:
  max_amount_usdc_per_call: 0.05
  max_amount_usdc_per_day: 10.00
  max_calls_per_minute: 60
  allowed_recipients:         # optional allowlist
    - "<solana_address>"

# Automatic deny — no override
deny:
  anonymous_agent: true
  missing_intent_declaration: true
  amount_exceeds_limit: true
  action_not_in_allowed_intents: true
  duplicate_intent_within_ms: 500

# Human approval gate
human_approval:
  required_above_usdc: 1.00
  required_for_actions:
    - execute_swap
    - transfer

# Proof requirements
proof:
  type: saprs_poe           # Proof of Execution
  algorithm: sha256
  emit_on: allow            # always | allow | deny
  chain: intent -> policy -> result -> signature
  on_chain_anchor: true
```

---

### 2.4 Policy Evaluation Flow

```
Agent declares intent
        │
        ▼
Policy Runtime loads policy_account (PDA)
        │
        ▼
Evaluate: is action in allowed_intents?     → NO  → DENY + emit PoE(deny)
        │ YES
        ▼
Evaluate: amount within limits?             → NO  → DENY + emit PoE(deny)
        │ YES
        ▼
Evaluate: deny conditions triggered?        → YES → DENY + emit PoE(deny)
        │ NO
        ▼
Evaluate: human approval required?         → YES → PAUSE + request approval
        │ NO / APPROVED
        ▼
ALLOW — execute x402 payment
        │
        ▼
Emit Proof of Execution (PoE)
        │
        ▼
Anchor PoE hash on Solana
```

---

### 2.5 Proof of Execution (PoE) Standard

Every policy evaluation MUST produce a PoE artifact.

```json
{
  "saprs_version": "0.1.0",
  "poe_id": "<uuid>",
  "intent_id": "<intent_id>",
  "policy_id": "<policy_id>",
  "agent_id": "ag9:solana:<public_key>",
  "result": "ALLOW | DENY",
  "reason_code": "WITHIN_POLICY | LIMIT_EXCEEDED | UNAUTHORIZED_ACTION | ...",
  "evaluated_at": "<iso8601>",
  "policy_hash": "sha256:<hash_of_policy_at_evaluation_time>",
  "intent_hash": "sha256:<hash_of_intent>",
  "delivery_hash": "sha256:<hash_of_poe>",
  "solana_anchor": "<tx_signature>",
  "verify_endpoint": "https://<provider>/v1/proof/<delivery_hash>"
}
```

**The chain:**
`sha256(intent) + sha256(policy) + result → delivery_hash → anchored on Solana`

This means: anyone with the `delivery_hash` can verify that a specific policy governed a specific intent, producing a specific result — without trusting the provider.

---

### 2.6 On-Chain Policy Registry

Policies MUST be registered as PDAs on Solana before they can be referenced in intents.

```
PDA seeds: ["saprs_policy", agent_pubkey, policy_version]

Policy Account data:
  - policy_id: [32]bytes
  - policy_hash: [32]bytes       // sha256 of policy document
  - registered_by: Pubkey
  - registered_at: i64
  - active: bool
  - version: u32
```

Benefits:
- Policy is immutable after registration
- Anyone can verify the policy that governed a transaction
- Policy rotation requires new PDA + explicit agent migration
- Regulators can audit policy history on-chain without trusting any provider

---

## 3. Reason Codes

| Code | Meaning |
|---|---|
| `WITHIN_POLICY` | Intent evaluated, all conditions met |
| `LIMIT_EXCEEDED` | Amount or rate above declared limit |
| `UNAUTHORIZED_ACTION` | Action not in allowed_intents |
| `MISSING_POLICY_ACCOUNT` | Agent has no registered policy |
| `POLICY_EXPIRED` | Policy registration is inactive |
| `HUMAN_APPROVAL_PENDING` | Awaiting human gate |
| `DUPLICATE_INTENT` | Same intent within dedup window |
| `DENY_CONDITION_TRIGGERED` | Hard deny condition matched |

---

## 4. Ecosystem Integration

### 4.1 With AG9 / Know Your Agent

SAPRS uses `agent_id` in AG9 format: `ag9:solana:<public_key>`. An agent with an AG9 identity can register a SAPRS policy PDA. Identity precedes policy — you cannot register a policy without a verifiable agent identity.

### 4.2 With x402

Before executing an x402 payment, the agent MUST present a valid `poe_id` from a SAPRS-compliant runtime. The x402 receipt and PoE form a combined compliance artifact: *payment happened* (x402) + *policy allowed it* (SAPRS PoE).

### 4.3 With Pay.sh

Pay.sh agents can declare a `policy_account` in their agent manifest. Pay.sh infrastructure MAY enforce SAPRS evaluation before forwarding payment requests, enabling enterprise-grade governance for Google Cloud agents.

### 4.4 With Open Transaction Layer

SAPRS PoE artifacts are structured to map to IVMS101 and ISO 20022 fields, enabling the Open Transaction Layer compliance coalition to include agent policy evidence in institutional transaction records.

---

## 5. Reference Implementation

**MIND Protocol** provides the reference implementation of SAPRS-001:

- Policy Runtime: `mind-policy-runtime` (production)
- Proof of Execution: Mindprint (SAPRS-compatible PoE)
- On-chain anchor: Solana devnet (mainnet Q3 2026)
- MCP endpoint: `https://api.mindprotocol.xyz/mcp`
- Agent Artifact format: compatible with Hermes Agent Skills

Live artifacts:
- `agent-artifacts/mind-risk-scoring/` — risk-gated policy example
- `agent-artifacts/mind-signals/` — read-only policy example
- `agent-artifacts/mind-market-intelligence/` — credential-tiered policy example

---

## 6. What This Specification Does Not Cover

- Agent-to-agent delegation chains (future: SAPRS-002)
- Cross-chain policy portability (future: SAPRS-003)
- ML-based risk scoring integration (future: SAPRS-004)
- ZK-proof of policy evaluation for private spending (future: SAPRS-005)

---

## 7. Feedback and Adoption

This specification is open for ecosystem feedback.

To adopt SAPRS-001 in your project:
1. Register your agent identity via AG9
2. Declare a policy document following Section 2.3
3. Register the policy as a PDA on Solana
4. Emit PoE artifacts following Section 2.5
5. Optionally: use MIND Protocol as managed runtime

To contribute to the specification:
- Open a discussion at `github.com/mind-protocol/saprs`
- Tag `@MINDProtocol` on X with feedback
- Contact: `hello@mindprotocol.xyz`

---

## 8. Path to Solana Foundation Proposal

This specification is intended for submission as a Solana ecosystem primitive following community validation. The proposed path:

1. **Now:** Publish SAPRS-001 draft publicly
2. **Month 1:** 2-3 ecosystem projects adopt the intent schema
3. **Month 2:** Submit to Solana Foundation as ecosystem primitive proposal
4. **Month 3:** SIMD proposal for on-chain Policy Registry program
5. **Q4 2026:** SAPRS as native Solana agent infrastructure

---

## Appendix A: Comparison with Existing Standards

| Feature | x402 | AG9 | OKX APP | SAPRS-001 |
|---|---|---|---|---|
| Payment execution | Yes | No | Partial | No (pre-payment) |
| Agent identity | No | Yes | No | Reference only |
| Policy declaration | No | No | No | Yes |
| Spending limits | No | No | Partial | Yes |
| Proof of execution | No | No | No | Yes |
| On-chain audit trail | Partial | No | No | Yes |
| EU AI Act alignment | No | No | No | Yes |
| Open standard | Yes | Yes | Yes | Yes |

---

*SAPRS-001 is authored by Diego Guedes (MIND Protocol) and released under CC BY 4.0.
Fork it. Build on it. Cite it. The ecosystem wins when the standard exists.*
