# IntentGrant

> A physical-intent authorization protocol for the AI-agent execution loop.
> "OAuth answers *who you are*. IntentGrant answers *is this what you intend to do, right now*."

IntentGrant is a protocol layer that turns a human "yes" during an AI-agent action into a
time-limited, auditable, hardware-signed authorization credential. As agents move from
understanding to *executing* in the real world, someone has to answer three questions that
today's stacks leave open: **Who confirms, who holds the evidence, and how is liability
resolved when the action goes wrong?**

IntentGrant is our answer to that gap.

## The core idea

| | Today's answers | IntentGrant's answer |
|---|---|---|
| **Identity** | OAuth / SSO — *who you are* | kept as-is |
| **Intent** | **undefined** — no one checks *that you really mean this* | a physical-confirmation layer |
| **Authorization scope** | coarse, long-lived tokens | per-task, time-limited, minimal grants |

> Existing protocols (OAuth2, MCP, A2A) describe *how* an agent may act on your behalf.
> They do not capture *that you actively approved this specific action, right now* — with an
> unforgeable link to a human action. That missing step is what IntentGrant formalizes.

## Architecture (4 layers)

```
Layer 3  Audit        Full-journey traceability, immutable ledger
Layer 2  Grant        Credential issuance / signing / expiry  (intent grant tokens)
Layer 1  Physical     BLE event parsing — press, double-click, long-press, triple-click
Layer 0  Activation   Device binding (biometric / PIN / 2FA) — "physical hold ≠ permission"
```

## Assurance levels & physical triggers

Because confirmation strength should match action risk:

| Risk | Physical trigger | Example | Grant TTL | Scope |
|---|---|---|---|---|
| **Low** | double-click | read docs, AI chat | 10 min | read-only, no export |
| **High** | long-press | export / share / send out | 5 min | limited actions, self-destruct |
| **Critical** | triple-click | emergency / admin override | 3 min | single-shot, one-time |

## A lifecycle in one line

```
Activation check → parse physical action → grant requested
→ generate & sign credential (SE-chip) → service validates signature, expiry & scope
→ execute → credential destroyed → audit log closed
```

## Hardware-form agnostic

IntentGrant is not married to any one device. A ring, watch, band, glasses, or a physical
button on a phone can be the confirmation endpoint — whichever fits the scenario. The
**protocol** is the product, not the hardware.

## Aligned with existing standards

IntentGrant does not attempt to replace OAuth 2.0 — it defines the task-level human
confirmation layer that sits *on top* of it. It is positioned as an implementation profile
aligned with:

- **RFC 9396** (OAuth 2.0 Rich Authorization Requests) — fine-grained authorization detail
- **IETF Secure Intent Protocol** (draft) — intent binding, agent identity, delegation control
- **RFC 8693** (OAuth 2.0 Token Exchange) — intent → action credential exchange
- **RFC 9449 / RFC 8705** (DPoP / mTLS) — replay protection and device binding

See Chapter 2 of the specification for the full mapping.

## Repository contents

```
intentgrant-spec/
├── README.md             this file — English entry point
├── SPECIFICATION.md      normative specification, v2.0 Neutral Release (Chinese)
├── SPECIFICATION.docx    specification, editable
├── SPECIFICATION.pdf     specification, for external distribution
├── LICENSE               Apache License 2.0
└── NOTICE                provenance, neutrality, and implementation-status notice
```

## Implementation status — read before evaluating

**All capability claims in this repository are graded.** Chapter 12 of the specification
("Implementation Status Statement") classifies every capability into one of three levels:

| Level | Meaning |
|---|---|
| **Implemented (prototype)** | demonstrated in the working prototype |
| **Design intent** | depends on the mass-production secure version; not yet implemented |
| **Future work** | not started |

Claims about secure-element (SE) chip signing, physical tamper response, and biometric
binding are **design intent** at the prototype stage. They are not to be represented as
implemented capabilities. A working prototype demonstrates execution capability; it does not
constitute commercial validation.

## Repository status

- **Status:** Private · active development. This repo serves as *proof-of-record* for the
  protocol's original definition and implementation.
- **Language:** The normative spec (`SPECIFICATION.md`) is authored in Chinese. This README
  and the `NOTICE` file are provided in English for international review.
- **Public disclosure:** An initial framework was published on the TRAE community forum in
  **Aug 2026**; the full Neutral Release (v2.0) of the specification was completed
  **2026-09-06**.

## License / provenance

Protocol IP is independently managed. This is a *proof-of-record* repository.

- The `LICENSE` file is pre-set to **Apache License 2.0** and becomes operative upon public
  release of this repository. While the repository remains private, no rights to use,
  reproduce, or distribute the contents are granted.
- Initial public disclosure: TRAE community forum, **2026-08**.
- Export & open-sourcing decisions are made separately, once legal/business conditions allow.

See the `NOTICE` file for the provenance statement, the specification-neutrality statement,
and the implementation-status notice.

## Contact

Su Yawei (苏亚伟) — specification author, IntentGrant.
