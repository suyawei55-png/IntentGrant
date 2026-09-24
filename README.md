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

IG-Lite v0.2 composes with AP2 Checkout Mandates (`mandate_ref` → `checkout_hash`, semantic
mapping) and consumes the A2A `auth-required` task state; both are semantic mappings, not
interop claims (see IG-Lite §2.2–2.3).

## Consumer Profile — IG-Lite v0.2 (draft)

`IG-LITE.md` specifies the **consumer-facing profile** of the protocol family. v0.1.1
established the informed-consent evidence baseline (disclosure objects, physical
confirmation capture, grants, mechanism-graded assurance, neutral hash custody).
**v0.2 closes the two remaining gaps** in that baseline:

- **Disclosure sufficiency (§6)** — a machine-readable **Disclosure Baseline Schema
  Registry** translating current statutory disclosure obligations into `required` fields
  recomputed by the verifier; non-compliant disclosures fail closed (`rejected_schema`).
  Normative force comes from the statutory sources, published with per-item verification
  status (§6.6).
- **Delivery evidence (§4.7)** — a **Delivery Receipt (D1)** capturing the procedural fact
  of "a conspicuous viewing opportunity was offered" on weak surfaces, and a **Transit
  Fidelity Receipt (D2)** proving the relay did not swap the disclosure between storage
  and display. Delivery tiers are orthogonal to assurance levels (§5.6: combined level =
  max).

The complete evidence chain in one sentence: *everything that had to be said was said;
what was said was not altered; what cannot be altered was actually offered to the user;
and what the user saw was authorized by the user* (§6.1).

Three design positions distinguish it from the core specification (unchanged from v0.1):

| | Core (v2.0) | Consumer (IG-Lite v0.2) |
|---|---|---|
| **Assurance anchor** | enterprise / financial-grade paths | customer-service dispute-resolution grade |
| **Confirmation endpoint** | independent endpoints incl. dedicated hardware | in-band first (phone / app / wearable); hardware optional, graded by capability |
| **Custody model** | in-domain audit trail | neutral custodian as a hash log — *hash now, reveal on dispute* |

The profile grades confirmation endpoints by **Device Capability Class (DCC)**, and now
grades *delivery* by the same honesty discipline: what a device can attest (including
`secure_clock`) decides what evidence level it may claim. Overstating either downgrades
the evidence to the actual mechanism — the claim takes the hit, not the user.

**Status:** draft v0.2, authored in Chinese, open for community review. The Registry
schemas and D1/D2 receipts are **specification-defined**; the pseudocode-level reference
covers the issuance fail branch, fidelity checks and nonce-hash computation. Device-signed
delivery channels await DCC-B+ hardware. **"Specified" ≠ "implemented"** — the
implementation-status statement (§12) is normative for all external claims. The
neutral-custodian role has **no** production implementation yet.

## Repository contents

```
intentgrant/
├── README.md                       this file — English entry point
├── IP-STATEMENT.md                 licensing scope, trademark reservation, exclusions (v1.1)
├── DISCLOSURE-BOUNDARY.md          what is deliberately not published, and why
├── CONTRIBUTING.md                 how to file issues and pull requests (DCO)
├── LICENSE                         Apache License 2.0
├── NOTICE                          provenance, neutrality, and implementation-status notice
├── core/
│   ├── SPECIFICATION.md            normative Core specification, v2.0 Neutral Release (Chinese)
│   ├── SPECIFICATION.docx          specification, editable
│   └── SPECIFICATION.pdf           specification, for external distribution
├── profiles/
│   └── lite/
│       ├── IG-LITE.md              IG-Lite consumer profile, v0.2 draft (Chinese)
│       └── IG-LITE-PSEUDOCODE.md   pseudocode-level reference (v0.2: required-fail branch,
│                                   fidelity checks, nonce-hash)
└── bindings/
    └── README.md                   transport bindings (IG-MCP interface: to be added)
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

- **Status:** Public as of **2026-09-06**. Active development. The Apache License 2.0 grant
  set out in the `LICENSE` file is operative.
- **Role:** This repo serves both as *proof-of-record* for the protocol's original definition
  and implementation, and as the open review channel for drafts, including the Consumer
  Profile (IG-Lite) v0.1–v0.2.
- **Language:** The normative specs (`core/SPECIFICATION.md`, `profiles/lite/IG-LITE.md`) are
  authored in Chinese. This README and the `NOTICE` file are provided in English for
  international review.
- **Public disclosure:** An initial framework was published on the TRAE community forum in
  **Aug 2026**; the full Neutral Release (v2.0) of the specification was completed
  **2026-09-06**.
- **Honesty:** Implementation-status claims are graded in the specifications themselves;
  nothing in this repository should be read as a claim of production deployment, external
  customer validation, or standards-body recognition. The IntentGrant Foundation referenced
  in project communications is in setup, not incorporated.

## License / provenance

Protocol IP is independently managed. This is a *proof-of-record* repository.

- The repository is public as of **2026-09-06**; the **Apache License 2.0** grant set out in
  the `LICENSE` file is operative. Contributions are accepted under Apache-2.0,
  inbound = outbound, with DCO sign-off — see `CONTRIBUTING.md`.
- Licensing scope, trademark reservation, and scope exclusions: `IP-STATEMENT.md`. What is
  deliberately not published, and why: `DISCLOSURE-BOUNDARY.md`.
- Initial public disclosure: TRAE community forum, **2026-08**.

See the `NOTICE` file for the provenance statement, the specification-neutrality statement,
and the implementation-status notice.

## Contact

Su Yawei (苏亚伟) — specification author, IntentGrant.
