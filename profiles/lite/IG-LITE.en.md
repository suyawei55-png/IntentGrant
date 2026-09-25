# IntentGrant IG-Lite

> **Language authority.** This document is a non-normative English translation of
> the Chinese `IG-LITE.md`. The Chinese v0.2.1 specification is the normative
> source of record. This translation reflects the same 2026-09-25 cross-reference
> errata, with no change to mechanism semantics. If this translation conflicts
> with the Chinese source, the Chinese source controls; the inconsistency is a
> translation defect, not an alternative interpretation or a divergence between
> co-equal language editions. **MUST**, **SHOULD**, and **MAY** reproduce the
> requirements of the Chinese source and do not give this translation independent
> normative status. Section numbering remains identical in both languages, so
> cross-references resolve in either.
> Chinese statutory provisions are cited as **Article N**; the **§N** notation is
> reserved for sections of this specification.


## Consumer Physical Intent Authorization Profile Specification

**IntentGrant Consumer Profile Specification (IG-Lite) v0.2.1**

| Item | Content |
| --- | --- |
| Specification name | IntentGrant Consumer Physical Intent Authorization Profile (IG-Lite) Specification |
| Version | v0.2.1 English translation (cross-reference errata); v0.2 (Disclosure Baseline and Delivery Proof edition, 2026-09-20); v0.1.1 (open-source release, 2026-09-19); v0.1 (first edition, 2026-09-12) |
| Publication date | 2026-09-25 (draft) |
| Protocol designer | Su Yawei |
| Upstream specification | *IntentGrant Physical Intent Authorization Protocol Specification* v2.0 (neutralized edition, 2026-09-06), hereinafter **Core v2.0** |
| Document status | Non-normative English translation of a Draft — Standards Track specification; open for international review |
| License | Apache License 2.0 (specification text and reference implementation) |
| Companion releases | *IG-Lite Reference Implementation — Pseudocode* (specification-level reference implementation, synchronized with v0.2.1); *IntentGrant MCP Server Interface Definition v0.1* (IG-MCP, interface-layer implementation; a separate document — this specification remains transport-neutral, see §2.4) |
| Keywords | Consumer Profile, weak surface, Disclosure Object, Presentation Commitment, physical confirmation, LoA, DCC, Accountability Chain, Disclosure Baseline, Schema Registry, Delivery Receipt, A2A auth-required, AP2 mandate, FIDO UP/UV |
| Scope | Informed-consent capture, credential issuance, and evidence retention before high-consequence actions by consumer-grade AI agents |

> This specification is the **Consumer Profile** of Core v2.0. For the relationship between the Physical Authorization Endpoint (PAE) and Core v2.0's four-layer architecture (Layer 0–Layer 3), see §1.2 and Appendix C. Working definitions needed by this Profile are stated self-sufficiently in §1.5. All capability claims are governed by Chapter 12, "Implementation Status Statement" — **what the protocol can claim ≠ what it can execute**.

---

## Abstract

The question Core v2.0 answers is: how do you prove, when an AI agent performs a high-consequence operation on a human's behalf, that "this is in fact what the person actively wants to do right now"? Core gives a complete answer in the enterprise context (administrative control plane, role binding, and hardware-endpoint private-key signatures).

The consumer context exhibits four structural differences that Core v2.0 does not cover:

1. **Surface difference.** Consumer-grade agents run en masse on weak surfaces (screenless or small-screen earbuds, glasses, and rings, plus voice-first in-vehicle and AIOS scenarios). AP2's answer to "the user approves the Mandate" is the Trusted Surface — which presupposes a rich-UI client; that premise does not hold on weak surfaces.
2. **Voice difference.** Voice responses of the "OK, confirm" kind cannot serve as high-assurance confirmation evidence: per multiple 2026 empirical findings, a 5-second voice sample suffices to clone a voiceprint.
3. **Adjudication difference.** Consumer disputes are settled mainly through customer-service adjudication and civil litigation, and the basis for adjudication today is chiefly the platform's own system records — the "refund-only" mechanism came to an industry-wide end in 2025, marking the industry's self-repudiation of unilateral adjudication; meanwhile, judicial rules (the Supreme People's Court judicial interpretation on bank cards) require the party asserting an "authorized transaction" to prove **the verification information at the time the transaction occurred**.
4. **Device-capability difference.** Consumers refuse to pay uniformly for security hardware: the mechanisms span an enormous range, from generic Bluetooth buttons with no security boundary to wearables with an SE/TEE — any blanket claim that "physical button = high assurance" is overstatement (the origin of mechanism-graded assurance in §5).

This specification defines IG-Lite: on top of the Core v2.0 four-layer architecture, it defines for the consumer context the complete semantics of **disclosure (Disclosure Object and Presentation Commitment) → physical confirmation capture (challenge binding) → credentials (Confirm Grant / Access Grant) → evidence custody (neutral Custodian hash log)**, and adapts to the full spectrum of consumer hardware, from generic Bluetooth buttons to security-chip wearables, via a mechanism-graded Level of Assurance (LoA). The legal track adopts the **contractual-form reliable electronic signature** of Article 13(2) of the *Electronic Signature Law*, producing a customer-service-adjudication-grade evidence chain. The financial-grade track is deferred to a future Enterprise Profile; Core v2.0 does not define a CA-certified e-signature path for legal effect (see §9.1).

**v0.2 additions (relative to the v0.1.1 open-source release).** v0.1.1 already provided a complete evidentiary foundation for informed consent: the disclosure system (Disclosure Object and Presentation Commitment, §3.1–3.2), physical confirmation capture (IntentGrant, §3.3), the credential model (Confirm/Access Grant, §4), deliverability traces (D0 rendering traces, §3.3, audit grade), the mechanism-graded assurance level (LoA/DCC, §5), the four-segment Accountability Chain (§3.4), and neutral hash custody (§8). The evidence of informed consent can thus be grouped into four mutually independent elements — sufficiency (the disclosure was complete), integrity (the disclosure remained intact), deliverability (that same disclosure was actually presented for review), and authenticity of intent (the person affirmatively authorized the action as presented): **v0.1.1 covers integrity, authenticity of intent, and the audit-grade form of deliverability; v0.2 supplies sufficiency and upgrades deliverability from audit grade to device-signature grade**:

- **Sufficiency → Disclosure Baseline: Schema Registry (§6)**: current statutory disclosure obligations are made machine-readable per transaction type as schemas, and the `required` fields are recomputed for completeness by the verifier — a credential whose structure is non-compliant is judged invalid. The source of enforcement power is not an initiative of Yishan but the statutory source itself (non-compliant disclosure ought by rights to be excluded from valid evidence in the first place); the legitimacy of the baseline comes from statutory translation, not from a self-imposed standard.
- **Deliverability → Delivery Receipt / Transit Fidelity Receipt (§4.7)**: the weak-surface Delivery Receipt (D1, the procedural semantics of "a prominent review opportunity was given") and the Transit Fidelity Receipt (D2, a three-segment chain proving that the transit party did not swap the disclosed content), covering the gap of "stored version ≠ displayed version".
- **Composition semantics**: the complete evidence chain in one sentence — **the disclosure was complete; it remained intact; that same disclosure was actually presented for review; and the person affirmatively authorized the action as presented** (§6.1, the four-element model).

Ecosystem position: IG-Lite consumes the **auth-required state** of the A2A task protocol and defines its capture semantics; it **composes** with AP2 (a Confirm Grant references the checkout_hash of a Checkout Mandate); its terminology aligns with the FIDO UP/UV primitives. Payment rails are outside the scope of this specification (§1.4, Principle 1, "does not touch money").

---

## 1. Overview and Scope

### 1.1 Problem Statement

In the consumer context, the capture of "informed consent" when an agent initiates a high-consequence action (placing an order, deletion, outbound transmission, subscription change, spatial-device operation) currently exhibits five gaps that cannot remedy one another:

- **Gap 1: rich-UI confirmation is unavailable on weak surfaces.** AP2's Mandate Delegation requires the user to approve on a Trusted Surface [S2]; earbuds, glasses, rings, and in-vehicle voice have no clickable rich UI. The A2A task lifecycle already defines the `auth-required` state [S1], but it does not specify who captures the confirmation action once that state is entered, at what assurance level it is captured, or what credential the confirmation produces.
- **Gap 2: voice confirmation is not admissible.** Voiceprint cloning has entered a low-cost gray-market stage: a 5-second voice sample suffices to clone [S20]. "The user said 'confirm'" cannot serve as high-assurance evidence — this is not a product defect, it is an attack surface.
- **Gap 3: adjudication evidence is controlled solely by the platform.** In disputes such as agent-mediated purchasing, "whether the user gave informed consent" can only be checked against the platform's own logs: the party asserting authorization and the party holding the evidence are one and the same (the self-attestation trap). "Refund-only" (the platform unilaterally deciding a refund) was pioneered by Pinduoduo in 2021 and was fully abolished by Taobao and Douyin on the same day, 2025-04-22 [S35] — the unilateral adjudication model is systematically unbalanced when dispute volume is high, and has been self-repudiated by the industry.
- **Gap 4: consumer device capabilities span an extremely wide range.** From generic Bluetooth buttons with no security boundary to wearables with an SE/TEE, the security mechanisms are entirely different. Any blanket claim that "physical button = high assurance" is overstatement.
- **Gap 5 (closed in v0.2): there is no evidentiary path for disclosure sufficiency and deliverability.** Who determines whether the platform's disclosure "said everything", and how is "whether a review opportunity was given" proven — today only by the platform's self-attested logs. In an auto-renewal dispute, "extremely inconspicuous small gray type + pre-checked box" was found by a court to infringe the right to be informed (2025-08 Guangzhou Internet Court case [S46]); ex post review is costly and uncertain. Schematization and delivery receipts move these two links from ex post review forward into the credential structure layer.

### 1.2 Positioning of the Consumer Profile (Division of Labor with Core)

**Core v2.0** (Enterprise Profile): the four-layer architecture of the activation layer (Layer 0) / physical-action layer (Layer 1) / credential layer (Layer 2) / audit layer (Layer 3); the enterprise-administrator control plane; biometric binding and loss-reporting procedures; and endpoint private-key signatures (with in-SE signing as production design intent in Core Chapter 12). A financial-grade CA-certified e-signature track is a **future extension not defined by Core v2.0** (see §9.1).

**IG-Lite v0.2.1** (this specification, Consumer Profile): inherits the four-layer architecture and the credential/audit model, and performs six redefinitions for the consumer context:

| Dimension | Core v2.0 | IG-Lite v0.2.1 | Clause |
| --- | --- | --- | --- |
| Control plane | Enterprise administrator | Controller app + the user themself (no administrator review path) | §7.4 |
| Verifiability of consent | The credential itself | Disclosure first: Disclosure Object + Presentation Commitment + Timing Binding | §3 |
| Disclosure sufficiency | Undefined | Disclosure Baseline: Schema Registry (statutory source made machine-readable) | §6 |
| Deliverability | Undefined | Delivery / Transit Fidelity Receipt | §4.7, §5.6 |
| Assurance level | Implied by risk tiering | Mechanism-graded LoA × Device Capability Class (DCC) | §5 |
| Evidence attribution | Enterprise-side audit log | Neutral third-party custody hash log + Chinese-law anchors | §8, §9 |

In one sentence: **Core answers "is the operator actively authorizing at this moment" at enterprise grade; IG-Lite gives the answer to the same question in the consumer context — a customer-service-adjudication-grade evidence chain.**

### 1.3 Scenario Pillars (Illustrative; the Specification Is Not Limited to These)

**P1 On-person intent attestation**: high-consequence confirmation for wearables / AIOS proactive services. As an agent moves from "generating" to "executing on behalf" (a reminder turned into an order, confirmation of a schedule change, spatial-device operation), it captures the physical evidence that "the person wants this right now".

**P2 Personal data and memory access authorization**: before an agent accesses personal data such as the user's calendar, notes, and health data, it obtains a constrained standing authorization through an Access Grant (scope, quota, revocable), and the LoA rules of §5 determine which actions require per-instance confirmation.

**P3 Commercial-transaction disclosure confirmation**: order confirmation for instant retail / agent-mediated purchasing. Disclosure (merchant, item, amount, timeliness) → physical confirmation → four-segment Accountability Chain → evidentiary adjudication in dispute. This pillar composes with the AP2 Checkout Mandate (§2.3).

### 1.4 Design Principles

1. **"Does not touch money"**: payment authorization and clearing run through the platform's existing payment capability / AP2 Payment Mandate [S3]; IG-Lite captures only the "moment of consent" (including awareness of the amount) and neither touches payment rails nor handles payment credentials.
2. **Mechanism-graded**: LoA is a mechanism property, not a label (§5); device capability claims MUST be honest (§7.2), and a claim that diverges from reality is a violation (the overstatement prohibition, Chapter 12).
3. **Disclosure precedes confirmation**: a confirmation without verifiable disclosure does not constitute informed consent (§3); the confirmation action itself MUST have anti-fatigue and mis-touch-prevention design (§10.1).
4. **Tiered retention of evidence**: hash chain permanent (not personal data) + disclosure text time-limited (N parameter) + dispute freeze (the "stop processing" semantics, §9.4) — the tension between traces and the right to erasure is dissolved by mechanism, not by marketing language.
5. **Composition at the ecosystem position, not substitution**: it hangs off A2A states, AP2 mandates, FIDO terminology, and existing Chinese-law articles; it claims to replace no existing standard and claims no unverified interoperability.
6. **Statutory translation, not invention (new in v0.2)**: every `required` field of the Disclosure Baseline (§6) **MUST** be anchored to a current statutory source (article number + effective date); a field with no public statutory support MUST NOT enter `required` — the protocol translates obligations, it does not create them (§6.3).

### 1.5 Normative Language and Terminology Conventions

- **MUST** / **MUST NOT**: mandatory requirements of this specification; non-compliance means the implementation does not conform to this specification.
- **SHOULD**: recommended requirement; deviation is permitted only where a known reason exists, and any deviation MUST be explainable.
- **MAY**: optional behavior.
- Terms are consistent with the Core v2.0 terminology system: **Physical Authorization Endpoint (PAE)** means a form-factor-agnostic hardware endpoint that carries physical confirmation actions; **Controller App** means the application that manages PAE binding and assembles/submits credentials; **Control Plane** means the enterprise administrative plane, whose consumer-context responsibilities are borne by the Controller App (§7.4). These are self-contained working definitions for this Profile. Terms newly introduced by this Profile are listed in Appendix A.
- **Numbering disambiguation**: Core architecture layers are always written in full as **Layer 0–Layer 3** (or by layer name). The tokens **L0 / L1 / L1.5 / L2 / L3** are reserved exclusively for the Levels of Assurance defined in §5. Core Chapter 13's acceptance levels are referred to only as acceptance levels, never as LoA.
- Citation markers [S#] correspond to the "Sources" list at the end of the document. Statutory citations follow the text currently in force and use **Article N**; **§N** is reserved for sections of this specification.

---

## 2. Relationship to Existing Standards

### 2.1 Ecosystem Position

```
┌──────────────────────────────────────────────────────────────┐
│ Execution layer: platform business systems (commerce /       │
│ data services / payment rails, AP2 Payment Mandate) —        │
│ out of scope of this specification (§1.4, Principle 1)       │
└───────────────▲───────────────────────────────────────────────┘
                │ execution receipt hash alignment (§4.5)
┌───────────────┴───────────────────────────────────────────────┐
│ Credential layer: IG-Lite Grant (Confirm / Access)           │
│   ├─ mandate_ref ──references──▶ AP2 Checkout Mandate        │
│   │                               (checkout_hash)            │
│   ├─ schema_ref ──validates────▶ Schema Registry             │
│   │                               (§6, new in v0.2)          │
│   └─ constraint syntax semantically aligned with the         │
│      open Checkout Mandate (allowed_* / line_items /         │
│      quota)                                                  │
└───────────────▲───────────────────────────────────────────────┘
                │ physical event signature (DCC-graded, §5)
┌───────────────┴───────────────────────────────────────────────┐
│ Capture layer: PAE physical confirmation (defined here)      │
│   ├─ terminology aligned with FIDO UP/UV (AA TWG agenda)     │
│   ├─ disclosure first: Disclosure Object + Presentation      │
│      Commitment (§3)                                         │
│   └─ delivery proof: D1/D2 receipts (§4.7, new in v0.2)      │
└───────────────▲───────────────────────────────────────────────┘
                │ consumes the auth-required state and defines its semantics
┌───────────────┴───────────────────────────────────────────────┐
│ Task layer: A2A task lifecycle (auth-required) [S1]          │
│ MCP tool calls (elicitation = structured input collection,   │
│ orthogonal) [S5]                                             │
└───────────────▲───────────────────────────────────────────────┘
                │
┌───────────────┴───────────────────────────────────────────────┐
│ Shared core: IntentGrant Core v2.0 (Layer 0–Layer 3          │
│ architecture)                                                │
└──────────────────────────────────────────────────────────────┘
```

### 2.2 A2A: Semantics and Capture of auth-required

The A2A task lifecycle (hosted by the Linux Foundation) already contains the `auth-required` state among its seven states [S1]; the semantics and the capture of that state are undefined. IG-Lite supplies three things:

1. **Capture routing**: once a task enters auth-required, the confirmation request is routed by the AIOS to an available PAE (§7.3);
2. **Credential semantics**: confirmation produces a Confirm Grant (§4.2), whose LoA and evidence attribution are defined by this specification;
3. **State exit**: once the Grant is issued and consumed, the task resumes execution; if the Grant expires or is rejected, the task fails or is cancelled per A2A semantics.

Where a consuming task model does not adopt the A2A lifecycle (e.g. MCP tool calls, platform settlement flows), this specification defines a **confirm-required** state as an equivalent entry point: the Agent enters that state before performing a high-consequence action, and its capture semantics (routing §7.3, disclosure §3, physical confirmation §10.1, credential issuance §4) are identical to those of auth-required.

### 2.3 AP2: Commercial Slice Composition

AP2 (released 2025-09, donated to the FIDO Alliance Agentic Authentication Technical Working Group in 2026-04 [S4]) defines W3C VC-based Mandates: closed / open Checkout Mandate and Payment Mandate [S3]. Determination of the relationship: **not competition, but composition**.

- AP2's Mandate Delegation takes place on a Trusted Surface [S2]; weak-surface scenarios (earbuds / AIOS with no screen to tap) are unanswered by AP2 — IG-Lite grades surfaces instead: a Trusted Surface is a strong surface, and weak surfaces go through this specification's Presentation Commitment and physical capture.
- **Composition syntax**: the `mandate_ref` field of a Confirm Grant references the `checkout_hash` of a closed Checkout Mandate (SD-JWT VC, `vct=mandate.checkout.1`); the constraint syntax of an Access Grant aligns with the open Checkout Mandate (`allowed_merchants` / `line_items` / `quota`) [S3].
- **Wording discipline**: this specification claims only **semantic compatibility and field mappability**, not interoperability — the AP2 ecosystem does not know about IG-Lite, and genuine interoperability is ecosystem engineering for a later version.
- AP2's own text, in a NOTE, lists "a directly trusted user key, such as a passkey or hardware-attested key" as a **future exploration** for the Mandate delegation trust model (the current definition covers only User Credential and Trusted Agent Provider) [S2] — the PAE is one instantiation of that future direction (the hardware-attested key path). The citation here is marked as exploratory, as it in fact is.

### 2.4 MCP Elicitation: Orthogonal

MCP elicitation (specification revision 2025-06-18) is the server requesting **structured input** from the client (a JSON Schema form, with a three-state response) [S5]; it states that it mandates no particular user-interaction model, and it MUST NOT be used to request sensitive information. Characterization: **MCP solves "the Agent asking the user for parameters"; IG-Lite solves "the user giving the Agent consent"** — orthogonal and complementary, with no conflict.

IG-Lite's MCP tool-layer mapping is carried by a separate document, *IntentGrant MCP Server Interface Definition v0.1*, and is not defined in this specification — **this specification remains transport-neutral**: MCP is a binding layer, not a protocol layer, and the specification text is not specialized to any single binding form (for the version relationship between the interface definition document and this specification, see §13.1).

### 2.5 FIDO: Terminology Alignment

- The PAE physical event cites FIDO's **User Presence (UP)** concept; biometric verification in Core's activation layer (Layer 0) corresponds to **User Verification (UV)**; the DCC grading and capability claims (§5.1, §7.2) borrow the phrasing of authenticator attestation capability claims [S6][S7].
- One deliberate divergence: CTAP2.3 allows a single touch to constitute UP [S6]; IG-Lite captures a **consent signal**, not a presence signal, and a single click does not constitute consent (§10.1.1). The rationale and precedents supporting this distinction are in §10.1.1.
- The FIDO Agentic Authentication TWG (established 2026-04-28 [S4]) has an agenda covering the authentication assurance of "how a user delegates actions to an Agent"; IG-Lite is an early answer to that agenda in the Chinese AIOS and Chinese-law context, and will submit comments to it after publication.

### 2.6 The OAuth Family (Inherited from Core)

Core v2.0 Chapter 2's alignment relationships with RFC 9396 (RAR) and RFC 8693 / 9449 / 8705 are inherited in full; IG-Lite's Grant structure adds disclosure and LoA semantics on top, without changing Core's alignment conclusions.

### 2.7 Ecosystem Position Among Chinese Standards

The domestic agent standards system is taking shape. At the policy layer, the *Implementation Opinions on the Regulated Application and Innovative Development of Agents* (2026-05) requires the establishment of an agent standards system [S43]; the mandatory national standard *Basic Security Requirements for Agent Applications* has been formally initiated (plan number 20263116-Q-252, issued 2026-07, the world's first mandatory agent security standard) [S41]; national standards on agent identity management have also been reported as initiated [S44]. At the consortia-standards layer, *Technical Framework for Agent Identity Authentication and Authorization* and *Technical Requirements for Agent Runtime Security*, proposed by the China Cyberspace Security Association with participation from Ant Group, research institutes, carriers, and internet vendors, were officially launched on 2026-09-09 at the Bund Conference [S42].

Ecosystem determination (as of this specification's publication date): the layout above covers the agent's own identity authentication, permission control, runtime security, and application security baseline; **the disclosure semantics of the moment of confirmation and the retention of adjudication evidence** (this specification's §3, §4, §6, §8, §9) have no counterpart in public standard texts. The characterization is the same as in the other sections of this chapter: **orthogonal and composable** — identity authentication answers "who the Agent is and what permissions it holds", whereas this specification answers "how the moment of human consent is credibly captured and retained"; runtime-security techniques do not define evidentiary-law anchors, and this specification does not touch the agent's own security. After publication, comparison opinions and comments will be submitted to the domestic standard drafters in the same manner as to the FIDO AA TWG (§2.5). **v0.2 supplement**: the current-law disclosure obligations in the consumer-protection direction (Article 10 of the *Consumer Protection Regulation* (《消保条例》), Article 18 of the *Measures for the Supervision and Administration of Online Transactions* (《网络交易监督管理办法》), Article 17 of the *E-Commerce Law* (《电商法》), and others) are the direct statutory source of this specification's §6 Disclosure Baseline — the domestic ecosystem position extends from "no one covers it" to "translating current law" (§6.6, statutory mapping).

---

## 3. Disclosure and Intent Capture

> This chapter is the semantic extension of Core v2.0 Chapters 4–6 (activation layer / physical-action layer / credential layer, i.e. Layer 0 / Layer 1 / Layer 2) into the consumer context: Core defines "how a physical event becomes a credential", while this chapter defines "how informed consent is established before the credential". As of v0.2, this chapter also carries the interface to the §6 Disclosure Baseline: the Disclosure Object gains the `schema_ref` field (§3.1).

### 3.1 Disclosure Object

Before requesting confirmation, the Agent **MUST** construct a Disclosure Object and complete its presentation. Minimum fields:

```json
{
  "do_id": "do_01hv3k9x2q8c4rm7m3w2e6k9zm",
  "schema_ref": "ig:disclosure:T-0@1.0.0",
  "context_template": "Placing an order with {merchant}: {item} ({attrs}) x{qty}, total {amount}, {eta}",
  "rendered": "Placing an order with Example Coffee: Americano (no ice) x1, total CNY 16.90, delivery in about 15 minutes",
  "scope": "commerce.order.create",
  "counterparty": "Example Coffee (Example Merchant)",
  "amount": { "value": "16.90", "currency": "CNY" },
  "attrs": { "item": "Americano", "qty": 1, "ice": "no ice" },
  "risks": ["Made-to-order beverages are not eligible for the seven-day no-reason return"],
  "sensitive": false,
  "loa_target": "L1.5",
  "delivery_tier": "D0",
  "confirm_window": 60,
  "issued_at": "2026-09-12T10:29:00+08:00",
  "evidence_rules_version": "h(b64url)…",
  "disclosure_hash": "b64url(SHA-256(canonical_json))"
}
```

- `do_id`: the disclosure object's unique identifier, which **MUST** be generated by a ≥128-bit CSPRNG (UUIDv4 / ULID-26) and MUST form part of the hash input. Purpose: the disclosure's core fields occupy a low-entropy combinatorial space (merchant × item × amount × attributes are enumerable), so if the hash input carried no high-entropy random value, the `disclosure_hash` on the public custody log (§8 admission audit) could be used to reconstruct the transaction content by dictionary enumeration — and the characterization "a hash is not personal data" (§9.3) would lose its cryptographic support. With `do_id` in the hash, enumeration is computationally infeasible.
- `schema_ref` (new in v0.2): the identifier of the Disclosure Baseline schema applicable to this disclosure (including the version, `ig:disclosure:<type>@<version>`). It **SHOULD** be populated; once populated, the verifier recomputes the completeness of the `required` fields against that schema (§6.3 enforcement chain), and a credential that does not supply complete `required` fields per the schema is judged invalid. Schema type selection and the type registry are in §6.4.
- `disclosure_hash` is computed over the canonicalized JSON of the **core fields** (do_id, schema_ref (if present), scope, counterparty, amount, attrs, risks, sensitive, loa_target, delivery_tier, evidence_rules_version); canonicalization **MUST** follow RFC 8785 (JCS) [S15] — this is the basis of hash consistency across devices. `do_id` **MUST NOT** appear in the public custody log (§8).
- In scenarios involving **sensitive personal information** such as health data (P2), the Disclosure Object **MUST** set `"sensitive": true`, which triggers the separate-consent semantics of PIPL Article 29 (statutory mapping in §9.2).
- `evidence_rules_version`: the version hash of the platform evidence rules cited by this disclosure (the reliable-electronic-signature conditions agreed in the user agreement, §9.1), used to prevent "unilateral revision of the rules by the platform causing existing evidence to drift in what it points to".
- `delivery_tier` (new in v0.2): the declared level of delivery proof for this disclosure (D0/D1/D2, §5.6). It enters the hash to prevent downgrade rewriting, and interlocks with the per-type default combinations in §5.6 and the schema output in §6.4.
- `rendered` is the text actually presented on the weak surface (the announcement script / on-screen content) and **MUST** be semantically consistent with the core fields; rendering traces are in §3.3.
- Mapping for non-transaction scenarios: in P2 data-access scenarios, `counterparty` = the data source and `amount` is replaced by `rate` (a rate limit); in P1 intent scenarios, `amount` / `counterparty` are replaced according to the action semantics (e.g. target device name + operation). The core field structure is unchanged.

### 3.2 Presentation Commitment

When a weak surface (voice announcement, ≤1-inch screen) presents binding information to the user, it **MUST** use one of the following two:

- **a) Cross-surface tail-code comparison**: a trusted presentation surface (the Controller app's screen, or the screen of a paired second device) displays the full disclosure together with the **last 8 hex digits** of `disclosure_hash`; the weak surface announces the same tail code; the user compares the two for consistency. This mode **MUST** have a second trusted presentation surface independent of the announcement channel; otherwise it MUST NOT be used. The second trusted presentation surface SHOULD be **host-isolated** from the weak surface (a different device): in configurations where only the channel is separated while the rendering controller is the same entity (e.g. the same pair of glasses or the same phone rendering both the projection and the announcement), the tail code carries attention-binding semantics only, and cross-surface integrity verification MUST NOT be claimed (the determination criteria match §11.1). Its semantics are the same as Microsoft Authenticator's number matching: binding the user's attention **to the content of the trusted surface** (the official countermeasure to MFA fatigue bombing [S23]).
- **b) Structured triple**: the key amount (or rate limit, **including the currency**, or a single currency declared by an industry profile) + the counterparty's name + the last 4 digits of `disclosure_hash` (a spoken reference code), announced by the weak surface. The triple carries the transaction semantics itself — what the user hears is the core element of what they sign, and pressing the button is consent to that content.

A weak surface **MUST NOT** announce only a bare tail code (of any number of digits) without satisfying either a)'s second-trusted-surface requirement or b)'s structured content.

Rationale: the user cannot compute SHA-256 mentally — on a **single** weak surface, no tail code constitutes a cryptographic binding the user can verify independently. The verification semantics of a tail code can only be **cross-surface comparison** (a); in the single-surface case, the only thing that carries semantics is **the content itself** (b). The goal of tail-code defense is "blind approval without attention" (the fatigue attack surface); "announced content differing from signed content" is instead caught by the audit-layer rendering trace and the ex post consistency comparison of core fields (§3.1, §4.5). Residual risk of a): the last 8 digits = 32-bit collision resistance, and GPU offline enumeration is feasible within minutes — its security premise is that the second presentation surface and the challenge issuer are not compromised at the same time (§11.1).

### 3.3 Timing Binding

- The PAE signature **MUST** bind a real-time challenge: `nonce + disclosure_hash + expiry`; the confirmation window defaults to ≤60 seconds.
- **A physical event for which no challenge was received produces no Grant**: an offline press is discarded, neither cached nor forwarded (cache-and-forward with a downgrade marker is a configurable item for a later version).
- Audit-layer timing verification: the physical event timestamp **MUST** be later than the start of the disclosure announcement and earlier than the expiry of the challenge.
- **Rendering trace (D0, the current boundary of dual-modal evidence)**: this mechanism is **D0** among the §5.6 delivery tiers — the rendering party's self-attested, audit-grade trace. The hash of the announcement text enters the audit-layer log (**MUST**; the announcement text is disclosure source text, and its retention is subject to the §9.3 N parameter, with only its hash retained after N expires); the hash of the TTS audio track enters the audit-layer log (**MUST** where the interaction includes an audio announcement channel; exempt for screen-only interaction). The two have different verification semantics: **the announcement-text hash carries the consistency comparison against the signed disclosure** (§3.1 / §4.5 ex post audit anchor); **the audio-track hash carries only a single-instance trace of "the fact and the timing of the announcement"** — an audio byte stream is not reproducible and carries no consistency semantics. Bringing the canonicalized announcement script into the Grant's signed object (signature-grade audio binding) is a later agenda item (§11.2). The audit layer's ≥6-month floor applies to mechanism data such as the audit index and the hash chain, and does not include disclosure source text. v0.2's D1/D2 (§4.7) upgrade delivery proof from D0's self-attested grade to device-signature grade.
- Announcement-to-press window binding (§10.1.3): a press within ≤60 seconds after the announcement completes is valid; **a press while the announcement is in progress is invalid** (a silent period that prevents confirmation before the announcement finishes).

Informative: the complete confirmation message sequence (happy path, with delivery receipts as of v0.2) —

```
Agent → Controller app: ConfirmationRequest(scope, counterparty, amount, attrs…)
Controller app: construct Disclosure Object (with do_id/schema_ref/disclosure_hash)
             → recompute required completeness per schema_ref (§6.3; missing field = no challenge issued)
             → rendering trace into the audit layer → submit disclosure_hash to the Custodian
Controller app → weak surface: announce rendered + binding information (§3.2 a/b)
Controller app/renderer: generate Delivery Receipt D1/D2 (§4.7, per delivery_tier)
Controller app → PAE: challenge{nonce, disclosure_hash, exp}
User → PAE: confirmation action (double-click / long-press, §10.1.1)
PAE → Controller app: event signature (per attested_by subject, §4.2 signature rules)
Controller app: verify signature → issue Confirm Grant → Agent consumes credential → execute
Executor → Controller app: Receipt{executed_action_hash} → alignment check → receipted / VIOLATED
```

### 3.4 Accountability Chain (Four-Segment Hash Chain)

```
intent capture (Disclosure Object issued)
   → disclosure (Presentation Commitment + rendering trace [D0] / delivery receipt [D1/D2])
   → confirmation (PAE physical event signature)
   → execution (receipt hash alignment, §4.5)
```

Each of the four segments is signed and joined to the next; they **MUST NOT** be merged into a single aggregate hash.

Vocabulary correspondence: the four functional segments in the Abstract (disclosure → physical confirmation capture → credential → evidence custody) are the functional view of this chain — "credential" is the Grant produced by the "confirmation" segment, and the objects of "evidence custody" are the per-segment hashes (§8) together with the evidence object after alignment in the "execution" segment.

Rationale: the *Online Litigation Rules of the People's Courts* (《人民法院在线诉讼规则》) hold that blockchain evidence "is presumed authentic after being recorded on-chain, while the authenticity of pre-chain data is reviewed separately" [S31]; Article 93 of the *Provisions of the Supreme People's Court on Evidence in Civil Proceedings* (《最高人民法院关于民事诉讼证据的若干规定》) locates the determination of authenticity in the integrity and reliability of the system environment on which generation, storage, and transmission depend [S30]. Chained custody preserves only "after on-chain"; **the pre-chain links MUST therefore be independently verifiable in segments** — this is the evidentiary-law basis for not reducing the four segments to a single aggregate hash.

### 3.5 Channel and Pairing Security (Profile Delta over Activation-Layer Binding)

**Pairing association model**:

- Activation-layer binding **MUST** use LE Secure Connections (ECDH P-256); the association model **MUST** be Numeric Comparison (six-digit comparison at both ends) or OOB (e.g. QR scan).
- Just Works is permitted only as the fallback path for screenless devices; for a device bound via JW, the LoA of its Grants is **permanently capped at L1** (JW's TK=0-grade protection, with no MITM protection — confirmed by the original official documentation from Nordic and TI [S17][S18]).
- **Obligation to record the binding fact (Profile delta)**: the association model actually used **MUST** be recorded in the Profile delta field `association_model` of the activation-layer binding record (values `numeric_comparison` / `oob` / `just_works`) and included in the §7.3 registry snapshot — that record is the evidence for the LoA cap determination of "bound via JW"; where a registry entry lacks the field, it is treated as `just_works` (the least favorable model).
- **Post-binding verification**: where a screenless PAE falls back to JW, manual verification **MUST** be completed before the first Grant is issued — the Controller app displays the device fingerprint (serial number / public-key tail code) and the user confirms "this is the device in my hand".

**Event signature and conflict of interest**:

- The physical events underlying **L2 and above** evidence **MUST** be produced by the PAE's device-side key signing the challenge (`attested_by: "device"`); a credential carrying an app self-attested signature (`attested_by: "app"`) is **capped at L1.5**.
- For DCC-C devices (§5.1), the signature in fact occurs in the Controller app (the phone is simultaneously the BLE central and the authorized party): such signatures **MUST** be marked `attested_by: "app"` (app-attested) in the audit-layer record, strictly distinguished from `attested_by: "device"` (device-attested) — the phrasing aligns with FIDO authenticator attestation capability claims [S7].

**Relay**:

- Primary mitigation = the challenge-response Timing Binding of §3.3: a relay cannot answer a real-time challenge and can only forward a stale signature (stale = invalid).
- **Proximity MUST NOT be relied upon as a security property** (the car-key relay precedent: an attacker within a few meters of the victim can relay an unlock; Tesla counters with UWB ranging [S19]); precise UWB ranging is listed as an optional enhancement for a later version (aligning with the CCC Digital Key path).

---

## 4. Credential Model

### 4.1 Credential Type Overview

| Credential | Semantics | Consumption Mode | Core Counterpart |
| --- | --- | --- | --- |
| **Confirm Grant** | One-time confirmation: "I agree to this one, at this moment" | Single use, default expiry ≤300 seconds | Consumer refinement of the Core credential-layer Grant (operation-level) |
| **Access Grant** | Constrained standing authorization: "this class of thing is allowed, within these bounds" | Standing; no per-instance confirmation within quota | Extension of Core Permission_Scope |
| **Receipt** (§4.5) | Execution receipt: what was executed, and whether it aligns with the disclosure | Produced on each execution | Structured form of the Core audit-layer Result |
| **Delivery Receipt** (§4.7, v0.2) | Delivery receipt: the weak surface did render the disclosure (D1) | Produced on each disclosure delivery (per delivery_tier) | No Core counterpart — new in v0.2 of this Profile |
| **Transit Fidelity Receipt** (§4.7, v0.2) | Transit fidelity receipt: the transit rendering matches the source disclosure (D2) | Produced on each transit rendering (per delivery_tier) | No Core counterpart — new in v0.2 of this Profile |

**Boundary rule**: Irreversible actions (memory deletion, outbound transmission, deregistration-class operations) are **never** covered by an Access Grant — a Confirm Grant is REQUIRED every time (§10.2 scenario table). **A delivery receipt carries no consent semantics**: no field in D1/D2 and no trace of user behavior constitutes a consent signal in the sense of §10.1.1; consent is carried solely by the IntentGrant (Confirm Grant) (§4.7).

### 4.2 Confirm Grant

| Field | Type | Description |
| --- | --- | --- |
| `grant_id` | UUID | Inherits Core `Grant_ID`; globally unique |
| `grant_type` | `"confirm"` | One-time confirmation credential |
| `pae_id` | String | Inherits Core `Device_ID` |
| `dcc_class` | `"A" / "B" / "C"` | §5.1 Device Capability Class |
| `attested_by` | `"device" / "app"` | §3.5 event-signing subject |
| `disclosure_hash` | String | Hash of the bound Disclosure Object (§3.1) |
| `schema_ref` | String (optional) | Same source as the Disclosure Object (§3.1/§6.4); when present, the verifier recomputes completeness against it (§6.3) |
| `delivery_receipt_ref` | Object (optional) | Delivery receipt reference (§4.7): `{receipt_type, receipt_id}`; for high-value transactions the Accountability Chain includes a delivery segment |
| `mandate_ref` | Object (optional) | §2.3: references an AP2 closed Checkout Mandate |
| `loa_target` | `"L1" / "L1.5" / "L2"` | Target assurance level; actual LoA below target → Grant invalid (§5.3) |
| `confirm_modality` | `"double_click" / "long_press" / "declared_equivalent"` | §10.1.1 action modality; accessible alternatives MUST be declared (§5.5) |
| `challenge` | Object | `{nonce, disclosure_hash, issued_at, expires_at}`, §3.3 |
| `evidence_rules_version` | String | Same source as the Disclosure Object |
| `session_ref` | UUID | Owning session (used for rate-limit counting, §10.1.2) |
| `issue_time` / `expire_time` | Timestamp | Single use; default ≤300 seconds (aligned with the Core High tier) |
| `signature` | String | PAE key signature (by the `attested_by` subject) |

**Signature computation object (normative requirement)**:

- `signature` covers the RFC 8785 (JCS) canonical byte string [S15] of **all fields of this credential except `signature`** (including `challenge`). Signing only the challenge does not cover the credential body: if the device signs only the challenge, the controller app can assemble arbitrary Grant fields outside the device signature, and the anti-forgery value of the device-side signature is lost.
- When `attested_by: "device"`, that signature **MUST be computed by the PAE device side** — the Controller App assembles the credential draft and sends it over the activation-layer channel for signing, and the device returns `signature`; the app **MUST NOT sign in its place** (the DCC-C case is `attested_by: "app"`, §3.5).
- The verification public key is distributed with the credential: a Grant copy **SHOULD** embed the PAE public key or its fingerprint, so that a verifier in a dispute (platform / judiciary / user) can verify the signature independently without calling back to the controller registry. Core v2.0 does not define a standalone key-rotation procedure. If an implementation rotates a key, it **MUST** update the activation-layer binding before issuing subsequent Grants and retain the public keys or fingerprints needed to verify historical Grants.

**Pre-issuance validation (added in v0.2, required-fail branch)**: Before assembling the credential draft, the controller app **MUST** validate the consistency of the Disclosure Object with `schema_ref` — if `schema_ref` is present, it recomputes required completeness per §6.3: **missing, empty string, whitespace-only, or a not_applicable declaration with no stated reason → refuse to generate the challenge (the transaction does not reach the confirmation step at the protocol layer)**. This validation guards against "unintentional non-compliance" (a developer omitting a field); against deliberate bypass, the defense lies at verification-side arbitration and at the legal layer — **the two layers each guard a different class of object, and the protocol layer MUST NOT claim "admission blocking"** (§6.3 ecosystem-position boundary).

Example (P3 commercial-transaction pillar, the same order as the §3.1 example):

```json
{
  "grant_id": "urn:uuid:0d1f9a4c-…",
  "grant_type": "confirm",
  "pae_id": "PAE-2026-0001",
  "dcc_class": "B",
  "attested_by": "device",
  "disclosure_hash": "b64url(SHA-256(JCS(§3.1 core fields, incl. do_id)))",
  "schema_ref": "ig:disclosure:T-0@1.0.0",
  "delivery_receipt_ref": {"receipt_type": "delivery", "receipt_id": "urn:uuid:…"},
  "mandate_ref": {
    "protocol": "ap2",
    "vct": "mandate.checkout.1",
    "checkout_hash": "b64url(SHA256(checkout_jwt))"
  },
  "loa_target": "L1.5",
  "confirm_modality": "double_click",
  "challenge": { "nonce": "9f3a…", "disclosure_hash": "(same as above)", "issued_at": "…", "expires_at": "…+60s" },
  "evidence_rules_version": "h(b64url)…",
  "session_ref": "urn:uuid:session-…",
  "issue_time": "2026-09-12T10:30:12+08:00",
  "expire_time": "2026-09-12T10:35:12+08:00"
}
```

### 4.3 Access Grant

| Field | Type | Description |
| --- | --- | --- |
| `grant_id` / `pae_id` / `dcc_class` / `attested_by` | Same as Confirm | — |
| `grant_type` | `"access"` | Standing authorization credential |
| `scope_constraints` | Object | Constraint syntax aligns with the AP2 open Checkout Mandate [S3] |
| `per_action_policy` | Object | Per-action LoA routing (see below) |
| `quota` | Object | Frequency and amount limits (reused by §10.1.2) |
| `validity` | Object | `{not_before, not_after, renewal}`; standing or session-level |
| `revocation` | Object | Loss report / revocation aligns with the Core Chapter 10 emergency procedure |
| `signature` | String | Signing rules as in §4.2: a JCS-canonical signature over all fields of this credential except `signature`, computed by the `attested_by` subject (when `attested_by: "device"`, it MUST be computed device-side) — preventing the app side from tampering with the quota and constraints after assembly |

`scope_constraints` example (P3, semantics aligned with the open mandate field names):

```json
{
  "allowed_actions": ["commerce.order.create"],
  "allowed_merchants": ["Example Coffee"],
  "line_items": [ { "category": "made-to-order beverage", "max_qty_per_order": 3 } ],
  "per_order_amount_max": "30.00",
  "data_retention": "session_only"
}
```

`per_action_policy` example (P2, data access):

```json
{
  "default": { "loa": "L1", "quota": { "calls_per_hour": 20 } },
  "escalate": [
    { "action": "memory.read.recent_7d", "loa": "L1" },
    { "action": "memory.export", "loa": "L2", "grant": "confirm" },
    { "action": "memory.delete", "grant": "confirm_always" }
  ]
}
```

- `grant: "confirm_always"` = a Confirm Grant is always REQUIRED (irreversible actions, §4.1 boundary rule).
- Issuing an Access Grant itself **MUST** go through one full Confirm flow (including disclosure and physical confirmation); when the quota is exhausted or the policy escalates, it **MUST** return to per-instance confirmation.
- The Disclosure Object of that Confirm flow **SHOULD** present in full the constraints that are to be standing-authorized (scope, quota, limits) — the constraint semantics thereby enter the disclosure hash and the Accountability Chain (§3.4), and can be reconciled after the fact.

### 4.4 Lifecycle and State Machine

```
drafted (disclosure construction)
  → presented (Presentation Commitment executed, rendering trace [D0] / delivery receipt [D1/D2])
  → challenge_issued (≤60s window)
  → confirmed (physical event signature valid → Grant issued)
      → consumed (single use, execution triggered)
      → receipted (receipt hash aligned, closed loop)
  / expired (window or credential expired)
  / revoked (loss report / revocation, aligned with Core Chapter 10)
  / rejected_bind (activation-layer verification failed, aligned with Core Chapter 8)
  / rejected_schema (required completeness validation failed, new in v0.2 — no credential is issued, the transaction does not proceed)
```

The state semantics follow the intent lifecycle of Microsoft Agent Governance (Declare → Approve → Execute; verified against the official blog post of 2026-05-14, "Governance at the Speed of Agents"); this specification refines the terminal state into `receipted` / **VIOLATED (unauthorized execution)** (where a hash mismatch is recorded as VIOLATED, see §4.5); v0.2 adds the pre-state `rejected_schema` (§6.3).

### 4.5 Execution Receipt and TOCTOU Alignment

The gap between confirming X and executing Y (TOCTOU) MUST be closed:

- After completing the action, the executor **MUST** produce a Receipt: `{grant_id, executed_action_hash, disclosure_hash, executed_at, result}`;
- `executed_action_hash` **MUST** match the alignment rules of the scope declared by `disclosure_hash` (action, subject, and amount/limit agree within tolerance);
- **Non-alignment = the credential chain provides no proof of authorization for it**: the execution is recorded in the audit layer as unauthorized execution (VIOLATED). This clause adjudicates only at the evidence-chain layer — **it does not adjudicate contract validity**: whether a contract is formed is determined separately under Article 48 of the *E-Commerce Law* (《电子商务法》) and Article 491 of the *Civil Code* (《民法典》) (§9.2).
- Pattern corroboration: the AP2 Action Authorization Verifier returns a Receipt [S2]; IG-Lite generalizes "receipt alignment" from the payment domain to all confirmation scenarios.

### 4.6 Design Rationale: Cross-Device WYSIWYS (Informative)

The gold standard for display integrity in hardware wallets is WYSIWYS — a secure screen attached directly to a secure chip, "what you see is what you sign" (Ledger/Trezor's own phrasing [S10][S11]). But Ledger achieves integrity through **integrated hardware**; the AIOS scenario is cross-device (rendering on the AIOS/controller side, signing on the PAE), with no integrated hardware.

IG-Lite's answer is **Timing Binding + third-party custody**: the disclosure hash is generated before the physical event and trace-logged over an independent channel (the Custodian, §8); the signed Grant binds that hash; the audit layer verifies the timing that "the event is later than the announcement and earlier than the challenge expiry". "What was confirmed" is not self-attested by the signing device, but jointly proven by **disclosure-signing timing + third-party custody**. This is this specification's technical contribution beyond existing precedents, and the answer to "how to prevent misdirection on a small-screen or screenless earphone".

### 4.7 Delivery Receipt and Transit Fidelity Receipt (New in v0.2)

> This section defines two delivery-proof credentials, corresponding to the D1/D2 tiers in §5.6. The two have different proof objects and **MUST NOT be cited interchangeably**. Design lineage: the design draft *DisplayReceipt: Delivery Receipt and Transit Fidelity* (2026-09-16) plus red-team revisions (2026-09-20, redteam/模拟评审-v0.2-2026-09-20.md).

#### 4.7.1 Mode A: Weak-Surface Delivery Receipt (D1)

**Semantics**: proves that **the weak surface of the paired device rendered the complete disclosure of the bound content within the window**. Does not prove: that the user read it, understood it, was present, or consented (the "prominent manner" requirement of *Consumer Protection Regulation* (《消保条例》) stops at opportunity; consent is carried solely by the IntentGrant).

**The probative force of dwell duration is graded by device capability (red-team E-2/P-2 revision)**: a device signature proves the signing instant and the signed object, but cannot prove that "rendering lasted N seconds" — if a self-reported duration entered the proof object directly, the first thing challenged would be "1 second of rendering signed as 8 seconds". Two paths:

- **Secure time-source path**: the DCC endpoint capability matrix includes a "secure clock" capability bit (§5.1, TEE-protected clock); on endpoints with that capability, the render_start/end dual timestamps are produced by the secure clock and enter the proof object (the procedural semantics of "dwell for the specified duration" holds);
- **claimed path**: on endpoints without a secure time source, `render_duration_ms` is a **claimed field** (self-reported by the rendering side, included in the hash for tamper-evidence but not in the proof), and the proof object narrows to "a rendering event occurred within the window".

**Credential structure (illustrative)**:

```json
{
  "receipt_type": "delivery",
  "receipt_id": "urn:uuid:…",
  "device_id_cert": "…(DCC-B or above device certificate chain)",
  "grant_ref": "urn:uuid:… | null",
  "content_hash": "b64url(SHA-256(canonical_json(rendered_disclosure) || per_receipt_nonce))",
  "nonce": "…(≥128bit device CSPRNG, included in the credential)",
  "schema_ref": "ig:disclosure:T-SUB@…(aligned with the Schema Registry)",
  "render_start": "2026-…T…Z",
  "render_duration_ms": 8000,
  "signature": "…(device private key, inside TEE/SE)"
}
```

- `grant_ref` **MAY be null**: it is null for announcement-type flows that announce first and confirm later, and after browsing without confirmation. A null value means "the disclosure was delivered but confirmation did not occur" — precisely the structural self-proof of "a delivery receipt carries no consent semantics".
- **Privacy red line (red-team P-1 revision)**: the computation of every content hash in this section (D1 `content_hash` / D2 `rendered_content_hash`) **MUST** mix in a per-receipt nonce (device CSPRNG, ≥128bit), stored alongside the credential; the Custodian **MUST NOT** record the mapping between nonce and rendered content (the same rule as the §3.1 prohibition on recording `do_id` — rendered disclosure content has low entropy, and a bare hash permits dictionary reconstruction of the transaction content from a public log).
- The signing endpoint floor is **DCC-B** (§5.1) — DCC-C has no independent security boundary, its signing subject is the controller app (`attested_by: "app"`), and it does not constitute device-signed delivery proof; in the DCC-C case, delivery proof falls back to D0 (§3.3 audit grade).

#### 4.7.2 Mode B: Transit Fidelity Receipt (D2)

**Semantics**: proves that **the content presented by the transit rendering party matches the disclosure generated by the source platform** — covering every path where "the party controlling rendering ≠ the party generating the disclosure" (including a phone's primary Agent relaying a platform disclosure and an enterprise-side Agent rendering one). Three-segment chain:

```
Source disclosure trace: the source platform generates source_disclosure_hash + custody anchor (timestamp or full custody, §8)
   ↓
Transit rendering: the rendering party renders from the source disclosure (paraphrasing format permitted, transaction elements verbatim)
   ↓
Rendering receipt: the rendering party's device signs rendered_content_hash, fidelity comparison → match / mismatch
```

**Credential structure (illustrative)**:

```json
{
  "receipt_type": "transit_fidelity",
  "receipt_id": "urn:uuid:…",
  "device_id_cert": "…(DCC-A/B)",
  "source_disclosure_hash": "b64url(…)(generated by the source platform)",
  "source_anchor_ref": "…(timestamp or custody-trace reference, §8)",
  "rendered_content_hash": "b64url(… || per_receipt_nonce)(computed by the rendering side)",
  "render_agent_id": "…(primary Agent identity, KYA integration point)",
  "render_timestamp": "…",
  "fidelity_verdict": "match | mismatch",
  "signature": "…(rendering-side device private key)"
}
```

**Boundaries**:

- The proof object is **rendering fidelity**, not proof that the user read or consented to the disclosure; in unattended or near-unattended scenarios, a Transit Fidelity Receipt **MUST** be combined with a Confirm Grant to form a complete evidence chain (fidelity + confirmation).
- Rendering is fixed at that point: `rendered_content_hash` covers the canonical content snapshot at the rendering instant (including the per-receipt nonce, see the 4.7.1 privacy red line); local state changes after rendering are outside the proof domain.
- **Re-rendering versioning (red-team E-3 revision)**: when dynamic content (inventory/price changes) triggers a re-render, a new receipt **MUST** be re-signed, and the new receipt references `prior_receipt_id` to chain; after a mismatch the chain's state is VIOLATED, and subsequent disclosures MUST open a new chain; partial field updates use "incremental re-render + full re-sign" (a receipt has no incremental structure).
- **Mismatch arbitration baseline (red-team L-5 revision)**: on the trusted-timestamp path alone, the source platform can re-mint a new-version source hash after the fact and disguise a substitution as a normal match — so arbitration **MUST** use **the most recent custody anchor before the rendering instant** as its baseline; a receipt on the timestamp-only path is limited to audit reference and **MUST NOT** support an admissibility-grade transit-fidelity claim, which presumes the full-custody path (§8).
- Attribution of fault: on mismatch, the direction of the adverse burden of proof is determined by chain role — the transit party holds the receipt, the source platform holds the source hash, and responsibility for an alignment failure rests with the party that changed content; the evidence structure points there naturally (subject to the arbitration baseline in the preceding clause).

#### 4.7.3 Capability Status Statement

The credential structures of both modes are **semantic designs** (defined in the v0.2 specification); the device-signing channel depends on DCC-B-or-above endpoints (mass-produced secure hardware, §12 implementation status); source-disclosure trace integration depends on commercial cooperation from the source platform (§11.2 reserved limitation). **Specification target state ≠ current implementation capability** (Chapter 12 discipline).

---

## 5. Level of Assurance (LoA)

### 5.1 Device Capability Class (DCC) — The Physical Premise of LoA

| Class | Definition | Example form factor |
| --- | --- | --- |
| DCC-A | Key stored inside the device security boundary (SE/TEE/secure enclave), with firmware signature verification | A ring/watch with a secure element |
| DCC-B | Key in device-dedicated storage, with no security boundary | Firmware key of a premium earbud |
| DCC-C | Key at the Controller app / phone layer; the device merely reports events | Generic Bluetooth remote button (the form factor of an off-the-shelf camera remote) |

DCC grading is a **claim + verification** problem: a capability claim (§7.2) that diverges from the actual mechanism is overstatement, handled under the Chapter 12 discipline.

**Capability matrix extension (v0.2)**: beyond DCC grading, an endpoint **MAY** declare additional capability bits, recorded in the Capability Descriptor (§7.2):

| Capability bit | Meaning | Unlocks |
| --- | --- | --- |
| `secure_clock` | A TEE/SE-protected clock (the timestamp cannot be tampered with at the application layer) | The "dwell duration" of a D1 Delivery Receipt enters the proof object (§4.7.1 secure time-source path); without this bit the duration field is claimed (it does not participate in the proof) |

(The remaining capability bits — sensor anchoring, display-channel confirmation, and so on — are left to a later version; capability-bit claims are equally subject to the overstatement prohibition.)

### 5.2 LoA Table = min(device mechanism ceiling, scenario requirement)

| LoA | Name | Mechanism requirement | Evidence produced |
| --- | --- | --- | --- |
| L0 | Notification | Any | None |
| L1 | Informed confirmation | Software confirmation; **a voice response is an auxiliary signal only and MUST NOT constitute confirmation on its own**; where voice serves as an auxiliary signal, **retaining the original audio is prohibited** (a voiceprint is sensitive personal information under PIPL Article 28 — the audit layer records only the determination result or its hash) | Software log |
| L1.5 | Physical confirmation (constrained) | Physical button + challenge-bound signature (app-attested for DCC-C); **the mechanism ceiling for DCC-B/C** (B may be `attested_by: "device"`, C only `attested_by: "app"`) | Physical event signature + disclosure hash binding |
| L2 | High-assurance confirmation | Physical event + **DCC-A** + Timing Binding + compliant Presentation Commitment | Complete four-segment Accountability Chain |
| L3 | Enhanced confirmation | L2 + CA-certified-form electronic signature + TSA timestamp + third-party custody (**target composition, not current capability**) | Financial grade (Enterprise Profile, planned; not defined in Core v2.0, see §9.1) |

- **A physical button does not by itself produce LoA; what produces LoA is the combination of "physical event + key mechanism". LoA is a property, not a label.**
- Basis for prohibiting standalone voice confirmation: a 5-second voice sample suffices to clone a voiceprint (2026 empirical finding [S20]).
- L3 is reserved for the Enterprise Profile; the highest target level for IG-Lite is L2.
- **Numbering note**: L0–L3 in this table are LoA values. Core architecture layers are written as Layer 0–Layer 3 (or by layer name) throughout this specification; the two dimensions MUST NOT be conflated.

### 5.3 Prohibition of Silent Downgrade

A Grant **MUST** declare `loa_target`; where the actual LoA at runtime is below `loa_target`, the Agent **MUST** re-request (re-disclose + re-confirm) or abandon execution. **Silent downgrade = the Grant is invalid.** The same rule applies to delivery tiers (v0.2): where a Disclosure Object declares `delivery_tier` and the actual delivery proof falls below that declaration (e.g. a D2 declaration downgraded to a D0 rendering trace), it is treated by analogy with silent downgrade — for a disclosure whose receipt is missing or whose tier does not match, the verifier accepts it at the actual tier and records this in the audit layer.

### 5.4 Shared-Device Constraints

A device shared by multiple people (no wear detection, or shared pairing) → LoA **capped at L1.5**; wear detection + single-person pairing can meet DCC requirements.

Rationale: earbuds are the most frequently shared category (family members picking up the wrong pair, children playing with them), and for them "physically in hand = authorized by the owner" does not hold. Household-scenario disclosure templates **SHOULD** state the identity of the current wearer (a best-effort declaration).

### 5.5 Accessible Alternative Channel

- The confirmation action pattern (§10.1.1) permits equivalent alternative channels (precedent: AssistiveTouch / Switch Control as alternatives to Apple Pay's side-button confirmation [S9]).
- The alternative channel **MUST** be declared in the Grant's `confirm_modality` field and recorded in the audit layer; **an alternative is an accessibility matter, not grounds for lowering LoA** — where an alternative channel is mechanically weaker than the standard action, it is graded by its actual mechanism.

### 5.6 Delivery Tier and LoA × Delivery Combinations (New in v0.2)

**The Delivery Tier (Tier D) is independent of LoA**; it describes only the strength of disclosure delivery, and the two axes are orthogonal:

| Tier D | Name | Mechanism | Proof object | Status |
| --- | --- | --- | --- | --- |
| D0 | Rendering trace (audit grade) | The §3.3 rendering trace, self-attested by the rendering party | Announcement-text hash + audio-track hash | Already in v0.1.1 |
| D1 | Weak-surface delivery receipt | §4.7.1, a DCC-B-or-above device signs the fact of rendering | Device-signed content_hash (including nonce) | Defined in v0.2 |
| D2 | Transit Fidelity Receipt | §4.7.2, the rendering end signs rendering fidelity | Three-segment chain: source-disclosure trace → transit rendering → rendering receipt | Defined in v0.2 |

**Orthogonality ruling (settled in v0.2)**: a schema declares the minimum LoA × D combination **by transaction type** (§6.4 type registry output); the scenario table (§10.2) requires LoA **by scenario**; at execution time **the maximum is taken** — the two are lower bounds on different dimensions, neither substituting for nor conflicting with the other.

**Type-level default combinations (the three launch schemas; full definitions in §6.4)**:

| Type | Default LoA × Delivery | Notes |
| --- | --- | --- |
| T-0 general | L1 × D0 | Platform risk control may raise it |
| T-AIR airline tickets | L1.5 × D1; above an amount threshold, L2 × D2 | The threshold is set by an industry profile; in the absence of a profile, it escalates at a face value of ≥¥5000 per ticket (a draft default, subject to revision in a subsequent v0.2 revision) |
| T-SUB subscription (first charge) | **L2 × D1 + an explicit confirmation action** | The first charge of an auto-renewal is a high-dispute point; renewal charges (non-first) are L1 × D1 + a 5-day-advance reminder trace |
| T-SUB subscription (renewal charge) | L1 × D1 | The consumer already has one complete disclosure record |

**Default recommendation for the phone-assistant scenario (from the consolidation decision, informative)**: attended = D2 alone (the user can compare it directly) + Confirm Grant (high risk); **unattended or near-unattended = D2 + L2 confirmation, mandatory**. The combination matrix is one of the schema's outputs — an industry profile declares the minimum combination by transaction type, and vendors may exceed it but not reduce it.

---

## 6. Disclosure Baseline: Schema Registry (New in v0.2)

> This chapter upgrades the "disclosure sufficiency is out of domain" entry in v0.1.1 §10.2 "Known Limitations" into a normative mechanism. Design upstream: the design draft *Disclosure Baseline Schema Registry* (v0.2, drafted 2026-09-16, merged after the 2026-09-20 source-verification batch and red-team revision).

### 6.1 Conceptual Model: The Disclosure Obligation Is Four Independent Elements

| Element | Question answered | Carrying mechanism | Version status |
| --- | --- | --- | --- |
| ① Sufficiency | Was the disclosure complete? | **The Schema Registry in this section** | New in v0.2 |
| ② Integrity | Did the disclosure remain intact? | disclosure_hash + signature + custody anchoring | Core of v0.1.1 |
| ③ Deliverability | Was that same disclosure actually presented for review? | D0 rendering trace (v0.1.1, audit grade) → D1/D2 delivery receipts (v0.2 §4.7, device-signature grade) | Upgraded in v0.2 |
| ④ Authenticity of intent | Did the person affirmatively authorize the action as presented? | IntentGrant (physical confirmation) | Core of v0.1.1 |

The complete evidence chain in one sentence: **the disclosure was complete; it remained intact; that same disclosure was actually presented for review; and the person affirmatively authorized the action as presented.**

Key point: the four elements are mutually independent, and if one link is missing the chain is blind on the corresponding question. What the schema supplies is link ① — it does not compete with the existing mechanisms; rather, it gives `disclosure_hash` a "content floor": v0.1.1's `disclosure_hash` solves **integrity** (the disclosure remained intact), not **sufficiency** (the disclosure was complete) — anchoring a hash to an insufficient disclosure is like stamping a seal across the pages of a non-compliant disclosure.

### 6.2 Registry Architecture and Governance

- **Form**: an open-source schema registry that registers a type schema by transaction type (§6.4 launch set: T-0/T-AIR/T-SUB). A schema is a data model (fields, types, the `required` set, N/A rules) plus a statutory mapping (§6.6); it is not code.
- **Versioning**: `ig:disclosure:<type>@<semantic version>`; the Registry retains the full version history (append-only), and version releases go through public review (consistent with the §13 version discipline).
- **Relationship to the §8 Custodian**: the Registry defines "what a complete disclosure is", while the Custodian anchors "that the disclosure hash did in fact exist before the transaction" — the Registry does not anchor evidence, and the Custodian does not interpret schemas; the two responsibilities are separated.
- **Admission audit**: a vendor's admission audit for joining the Registry shares its mechanism with the §8 custody admission (audit frequency, downgrade markers, public accountability). Spot checks for N/A abuse use the same channel (§6.3).
- **Open-source governance**: schema definition files are released in the same repository as this specification (Apache 2.0); external submissions of new type schemas go through PR review — **a new type MUST complete its statutory mapping before it can be merged** (a type with no public statutory source can only be registered at the recommended level).

### 6.3 Enforcement Chain and Ecosystem Position

**The enforcement chain (four links, each independent)**:

1. **Issuer**: when the PAE SDK / Controller app generates a Disclosure Object, it validates the `required` fields against `schema_ref` — **missing, empty string, or whitespace-only → refuse to generate a challenge** (at the protocol layer the transaction does not reach the confirmation step; §4.2 pre-issuance validation, §4.4 `rejected_schema` state).
2. **Integrity layer**: all fields (`required` and vendor alike) participate in canonical serialization (RFC 8785 JCS) and in the `disclosure_hash` computation — any subsequent alteration will cause hash verification to fail, at the same strength as the v0.1.1 tamper-resistance mechanism. **The division of labor MUST be kept clear**: the hash guarantees **integrity** (the disclosure the verifier sees = the disclosure as issued, treating all fields alike); **completeness validation is carried by the `schema_ref` recomputation in link 3** — the hash itself does not judge whether a required field is absent, and the two mechanisms **MUST NOT** be treated as interchangeable.
3. **Verifier**: during audit-layer review and Custodian verification, `required` completeness is recomputed after `schema_ref` version arbitration (§6.5) — **a structurally non-compliant Grant is judged invalid outright and does not enter any discussion of "strength of evidentiary effect"**.
4. **Legal consequence**: refusal to issue at the protocol layer is only a mechanism; the real enforcement power lies in the legal consequences of non-compliant disclosure itself (the legal-liability chapter of the *Consumer Protection Regulation* (《消保条例》), standard terms not being incorporated into the contract, and the compensation rules established by leading cases [S46][S47]). The protocol turns legal consequences into engineering consequences — this is "translation", not "invention".

> **Honest boundary of the ecosystem position (red-team L-2 revision)**: issuer-side validation guards against **unintentional non-compliance** (a developer omitting a field); it cannot guard against deliberate circumvention (assembling a credential by hand without going through the SDK) — it is a **quality filter**, not an admission gate. In a voluntary ecosystem, the real enforcement lies in verifier-side arbitration and legal consequences: a structurally non-compliant credential is judged invalid, and a non-compliant disclosure loses evidentiary effect by law. That is ex post arbitration and depends on courts and Custodians accepting and applying the rule. The two layers each guard against a different class of object and MUST NOT be conflated as "protocol-layer interception".

**Explicit N/A declaration rule**: where a `required` field does not apply, the implementation MUST explicitly declare `"applicability": "not_applicable"` **and attach the reason for that determination** (citing genuine legal doctrine; see the T-0 example in §6.4); silent omission, an empty string, and `null` are the three prohibited states. Abuse of `not_applicable` (claiming exemption in a scenario that plainly does not qualify) is subject to spot checks by the Custodian's admission audit (§6.2), and once confirmed, the vendor's subsequent transaction credentials carry a downgrade marker. **Article 19 of the *Consumer Protection Regulation* (《消保条例》), "the scope of goods not eligible for no-reason return MUST NOT be unilaterally expanded", constitutes a reverse statutory anchor for N/A truthfulness — the N/A reason MUST be genuine, and a fabricated exemption is precisely such a "unilateral expansion".**

**Vendor Namespace (add-only)**: a vendor may extend its own fields (`vendor.examplecorp.*`), and may **only add, never override `required`**; vendor fields participate in the hash (preventing ex post rewriting) but schema validation applies only to `required`. Direction: to make a vendor's disclosure "beyond the baseline" a comparable, promotable differentiator (turning disclosure sufficiency into a positive competitive dimension rather than a compliance burden). **Truthfulness boundary**: vendor fields are self-attested by the vendor plus spot-checked in admission audits, not warranted by the protocol — cross-vendor comparison holds only among audited vendors under the same Registry governance, preventing marketing abuse.

### 6.4 Type Schemas (Three at Launch)

#### 6.4.0 T-0 Fallback Set (Common to All Consumer Transactions)

| Field | spec | Statutory source (verification status in §6.6) |
| --- | --- | --- |
| `total_price` | Total transaction price + breakdown of its components (including separation of service fees / markups) | Article 13 of the *Price Law* (《价格法》) (clear price marking); Article 17 of the *E-Commerce Law* (《电商法》) (comprehensive, truthful, accurate, and timely disclosure) [S48] |
| `counterparty_full` | Full legal name of the actual operator (consistent with the entity's qualifications); in agent-mediated transactions, disclose the executing Agent identifier in parallel | Article 21 of the *Consumer Protection Law* (《消保法》) (obligation to mark the true name); the executing Agent identifier is a protocol-imposed requirement (a protocol-layer gain beyond statutory source, marked as such) [S49] |
| `return_policy` | Whether the seven-day no-reason return applies + the list of exclusions (where exceptions apply) | Article 25 of the *Consumer Protection Law* (《消保法》) and Article 19 of the *Consumer Protection Regulation* (《消保条例》) ("MUST NOT unilaterally expand the scope of exclusions") [S49][S50] |
| `dispute_entry` | (recommended, not required) Dispute-resolution entry point (a declaration of customer-service / complaint channels) | 2026-09-20 red-team evidence gathering: the dispute-resolution disclosure obligation under current law attaches only to the platform side (Article 63 of the *E-Commerce Law* (《电商法》) is an authorizing norm for platform operators; Article 21 of the *Measures for the Supervision and Administration of Online Trading Platform Rules* (《网络交易平台规则监督管理办法》) is likewise a platform obligation); an ordinary operator has no such disclosure obligation, so under the "no public statutory source → not in `required`" rule it is downgraded to `recommended`, pending a new statutory source [S51] |
| `risks[]` | Risk elements of this transaction type (reusing the §3.1 `risks` field semantics) | Article 20 of the *Consumer Protection Law* (《消保法》) (obligation of truthful and comprehensive information) [S49] |

JSON illustration (excerpt, showing the `required` structure and the N/A rule):

```json
{
  "schema_ref": "ig:disclosure:T-0@1.0.0",
  "required": {
    "total_price": {"amount": "3480.00", "currency": "CNY",
                    "breakdown": [{"item": "fare", "amount": "2980.00"},
                                   {"item": "platform service fee", "amount": "500.00"}]},
    "counterparty_full": {"name": "XX Air Passenger Transport Services (agent: XX Travel)",
                          "executing_agent": "did:ig:agent-7f3a"},
    "return_policy": {"applicability": "not_applicable",
                      "reason": "An air ticket is a contract for air carriage services and does not fall within the scope of Article 25 of the Consumer Protection Law (that article governs the sale of goods, and air tickets are not among its four listed exclusions); the right to a refund is instead realized under Article 23 of the Provisions on the Administration of Passenger Services in Public Air Transport, through the general conditions of carriage and the refund/change rules — see the T-AIR extended disclosure"}
  }
}
```

(The example above demonstrates the correct form of an explicit N/A declaration: **an exemption MUST cite a statutory reason, and that reason MUST be genuine legal doctrine** — the seven-day no-reason return does not apply to air tickets not because air tickets "are exempted under the Article 25 exclusions" (air tickets are not among the four listed exclusions in Article 25), but because an air ticket, as a carriage service contract, does not fall within Article 25's scope at all, and the refund right moves into the system of civil-aviation regulations plus the refund/change rules of the general conditions of carriage. Conflating "does not apply" with "is exempted" is the most common error in writing an N/A reason.)

#### 6.4.1 T-AIR Airline Tickets (High-Dispute Type One at Launch)

| Field | spec | Statutory source |
| --- | --- | --- |
| `refund_schedule[]` | Refund tiered rate table: by time interval before departure × fare level, listing the rate or amount for each tier | CAAC 2018 *Notice on Improving Civil Aviation Ticketing Services* (《关于改进民航票务服务工作的通知》) (Bureau-issued telegraph 〔2018〕No. 1952, "tiered rates") [S52]; the CATA refund/change consortium standard (**in preparation, recommended level**) — mandatory public disclosure of refund/change rates and free refund/change scenarios before payment [S53] |
| `change_rules[]` | Change rules using the same tier structure | As above |
| `free_refund_scenarios[]` | List of free refund/change scenarios (airline-caused, weather, force majeure, etc.) | A CATA consortium-standard disclosure item [S53] |
| `co_transfer_limit` | Whether endorsement/transfer is permitted, and its conditions | A CATA consortium-standard disclosure item [S53] |
| `carrier_identity` | The actual carrier (in code-share scenarios) | Article 23 of the *Provisions on the Administration of Passenger Services in Public Air Transport* (《公共航空运输旅客服务管理规定》) and the general conditions of carriage [S54] |

**Tense discipline**: as of 2026-09-20 the CATA refund/change consortium standard is "in preparation, planned for release within the year" (People's Daily, 2026-08-26 [S53]) — the related fields hang on `recommended_by` and **do not hang on `required`**; once the consortium standard is formally released, they are upgraded through Registry review (the N-day assessment mechanism is in §6.5). Precedent for industry self-initiation: China Eastern Airlines' new August 2026 *Implementation Rules for Voluntary Refunds and Voluntary Changes of Domestic Tickets* landed ahead of the consortium standard ([S53]) — the direction of making schema fields required already has spontaneous industry validation.

#### 6.4.2 T-SUB Subscription Auto-Renewal (High-Dispute Type Two at Launch)

| Field | spec | Statutory source |
| --- | --- | --- |
| `recurring_price` | Unit price per renewal period + period (month/year) | Article 10 of the *Consumer Protection Regulation* (《消保条例》) (obligation of prominent reminder for automatic extension / automatic renewal; verified verbatim across multiple official sources, 2026-09-20) [S55] |
| `first_charge_date` | Date of the first/renewal charge | As above ("prominently brought to attention before the date of automatic renewal, etc.") [S55] |
| `renewal_notice` | Pre-renewal reminder commitment: **≥5 days in advance** + reminder channel | Article 18 of the *Measures for the Supervision and Administration of Online Transactions* (《网络交易监督管理办法》) (**revised by SAIC Order No. 101 of 2025-03-18, effective 2025-05-01** — the revised text expressly requires prominent notice five days before the date of automatic renewal, etc., plus "a prominent and simple option to cancel or change at any time", and prohibits unreasonable fees) [S56] |
| `cancel_path` | Cancellation route (entry-point level, declared number of steps — "at the same level as sign-up") | **Article 18 of the *Measures for the Supervision and Administration of Online Transactions* (2025 revision): the direct regulatory basis** ("a prominent and simple option to cancel or change at any time") + a systematic interpretation of Article 10 of the *Consumer Protection Regulation* + the Supreme People's Court's leading consumer-protection case, Case 1, "Xie v. a video services company, a network service contract dispute" (auto-renewal without prominent reminder, **the operator was ordered to bear liability for the interest loss during the period the funds were occupied** — that case established the judicial review standard for "prominent") [S47] |
| `trial_terms` | Trial-period / low-price acquisition and automatic conversion clauses | The same case [S47] |

**Where the two judicial-admissibility questions land**: in a dispute, what the court asks is no longer "the platform says it reminded the user" (a unilateral record), but "whether the reminder action met the prominence standard defined by the schema and was proven by a v0.2 delivery receipt" (multi-party verifiable).

### 6.5 Versioning and Forensic Arbitration

`schema_ref` (including its version number) enters the hash as a core field of the Disclosure Object. In forensic evidence gathering, completeness is determined according to **the schema version applicable at the time of the transaction** — a new version after a statutory revision does not apply retroactively. The Registry retains the full version history, and the custody log can cross-verify version authenticity.

**T-SUB cross-version switching**: a renewal charge is recomputed against the schema version applicable at **the time of the charge** — the 5-day-advance reminder is a fresh obligation before each charge, and a single-transaction-time rule is insufficient for subscriptions, so renewal-charge credentials are validated independently. When a statutory revision triggers a version upgrade: existing subscriptions are not retroactively re-disclosed, but renewal-charge credentials generated after the upgrade takes effect are recomputed for completeness against the new version; credentials generated under the old version within the N-day assessment window retain their effect, and the assessment window itself serves as an honest-boundary declaration (the time lag between a new statute being knowable and the mechanism being upgraded is carried by the N-day mechanism).

**N-day assessment mechanism**: after a statutory revision, the Registry **SHOULD** complete the assessment of affected schemas and publish a version within 30 days (a default, adjustable by governance resolution); that window is the honest boundary for the period in which "the statute has changed but the schema has not yet caught up", and credentials under the old version retain their effect within the window (the §6.5 non-retroactivity principle); the fact that a window has passed without assessment is disclosed by the Registry on a public status page.

### 6.6 Statutory Mapping and Verification Status

**Rule (corresponding to §1.4 Principle 6)**: every `required` field of a schema **MUST** carry a statutory citation — article number + effective date + in-force status; a field with no public statutory support MUST NOT enter `required` (it may enter recommended); a case citation gives the issuing body + date, without inventing a docket number.

**Verification grading (2026-09-20 source-verification batch; see research/v02-factcheck-法源核验-2026-09-20.md for details)**:

- **Officially multi-source verbatim confirmation (≥3 official/authoritative sources cross-checked)**: Article 10 of the *Consumer Protection Regulation* [S55]; Article 18 of the *Measures for the Supervision and Administration of Online Transactions* [S56]; Article 17 of the *E-Commerce Law* [S48]; Articles 20, 21, and 25 of the *Consumer Protection Law* [S49]; Article 19 of the *Consumer Protection Regulation* [S50]; the 2018 CAAC notice (the caac.gov.cn official page, document number Bureau-issued telegraph 〔2018〕No. 1952) [S52]; the Supreme People's Court's 2026-03-15 leading consumer-protection case, Case 1 (court.gov.cn / Xinhua / People's Daily Online) [S47]; the Guangzhou Internet Court's August 2025 full-refund case (People's Daily, 2025-08-07, case commentary + the Ministry of Justice's smart legal-popularization platform) [S46].
- **Cleared (2026-09-20 final-verification batch, verbatim via curl of official full-text pages)**: the source text of Article 18 of the *Consumer Protection Regulation* (return/exchange/repair periods + one-time full refund at the price on the purchase receipt); the penalty for the obligation under Article 22 is in **Article 50(2)** (1–10 times + ¥500,000 + license revocation; paragraph 1 is the general penalty of 1–5 times for Articles 10–21); the version status of the *Measures for the Supervision and Administration of Online Transactions* = amended by SAIC Order No. 101 of 2025-03-18 (effective 2025-05-01), with the revised Article 18 adding the "option to cancel or change at any time" obligation — the direct regulatory basis for `cancel_path`; the operative holding of Supreme People's Court Case 1 = the operator was ordered to bear liability for **the interest loss during the period the funds were occupied** (verbatim from the court.gov.cn official page).
- **Remaining: official full-text comparison only**: the verbatim full text of the revised version of the *Measures for the Supervision and Administration of Online Transactions* under Order No. 101 (samr.gov.cn; the three key elements have already been confirmed by dual-source snapshots) [S56].
- **Tense-discipline items**: the CATA refund/change consortium standard is "in preparation" (§6.4.1) [S53]; its recommended-level attachment is maintained until formal release.

---

## 7. PAE: Terminology, Capability Descriptor, and Discovery

### 7.1 Terminology Alignment with FIDO

| This specification / Core term | FIDO counterpart | Notes |
| --- | --- | --- |
| PAE physical event (Core physical-action-layer event) | User Presence (UP) signal | UP = the minimum signal that a person is present; prevents malware from signing silently [S7] |
| Core activation-layer biometric verification | User Verification (UV) | Biometric verification of "who" |
| DCC grading + Capability Descriptor | authenticator attestation / metadata | How capabilities are stated [S7] |

Deliberate difference: CTAP2.3 allows a single touch to constitute UP [S6]; IG-Lite captures a **consent signal**, and a single click does not constitute consent (§10.1.1). The two have different security goals (presence vs. consent); the difference is a design choice, not an oversight.

### 7.2 Capability Descriptor

The PAE **MUST** provide a Capability Descriptor to the Controller app (GATT read or pairing registration information):

```json
{
  "pae_id": "PAE-2026-0001",
  "core_version": "2.0",
  "profile": "ig-lite",
  "profile_version": "0.2",
  "dcc_class": "B",
  "fw_version": "1.2.3",
  "confirm_modalities": ["double_click", "long_press"],
  "uv_available": false,
  "display": { "type": "none" },
  "pairing_models": ["numeric_comparison", "just_works"],
  "capability_flags": {"secure_clock": false},
  "battery_percent": 90
}
```

**Honest-claim discipline**: if `dcc_class`, `confirm_modalities`, `uv_available`, or `capability_flags` do not match the actual mechanism, this constitutes overstatement — the evidentiary weight of all historical Grants for that device is adjudicated as downgraded to the actual mechanism (Chapter 12).

Distinguishing capability advertisement from the binding fact: `pairing_models` states which association models the device **supports**; the model actually used for a binding is determined by the `association_model` in the activation-layer binding record (§3.5) — the LoA ceiling is determined by the latter, not by the capability claim.

### 7.3 Discovery and Confirmation Request Routing

- **Local discovery**: BLE advertising and GATT follow Core v2.0 Chapter 5 (Service/Characteristic UUIDs, 6-byte event packets); this specification does not redefine them.
- **Registry**: the Controller App maintains a registry of paired PAEs (including a Capability Descriptor snapshot and the activation-layer binding facts: `association_model` (§3.5), binding time, verification status).
- **Routing rules**: an Agent task enters A2A auth-required (or the equivalent confirm-required state defined by this specification) → AIOS queries the registry → routes by the shared-device constraints of §5.4 and the scenario's default PAE → when no PAE is available, handle per §5.3 (re-request or abandon); **MUST NOT** silently degrade to software-only confirmation and still claim the original `loa_target`.
- **Cross-device / cross-ecosystem cloud registration**: optional in later versions (future work, Chapter 12).

### 7.4 Binding and Verification

- Core v2.0 Chapter 4 (activation-layer flow, binding fields, locking and freeze policy) is executed by the Controller App in the consumer context; the administrator verification path is replaced by the post-binding human verification of §3.5.
- Loss reporting and revocation align with Core Chapter 10: one-tap loss reporting in the Controller app → `Bind_Status` freeze → forced revocation of all unexpired Grants for that `pae_id`.
- Simplification for the consumer context: Core's role-context handover (Chapter 11) is out of scope for this Profile.

---

## 8. Evidence Custody

> This chapter is the Consumer replacement for Core v2.0 Chapter 7 (audit layer, Layer 3): enterprise-side storage → neutral Custodian hash log.

- **Basis for independence (why the disclosure hash must be externally anchored)**: among the four segments of the Accountability Chain (§3.4), the confirmation segment is signed with a PAE-side key (§3.5) and the execution segment has Receipt alignment (§4.5), so the producer of each of these two segments can be separated from the interested party; **the disclosure segment, however, has the producer and the interested party as the same body** — the Disclosure Object is produced and presented by the Agent/platform (§3.1/§3.2), and if its retention and proof are likewise borne unilaterally by the presenting party, this constitutes self-attestation in a dispute: the user claims "I was not truthfully informed," and the platform proves that it informed by means of a log it generated and holds itself. Under Article 93 of the *Provisions of the Supreme People's Court on Evidence in Civil Proceedings* (《最高人民法院关于民事诉讼证据的若干规定》) [S30], review of the authenticity of electronic data rests on the integrity of the system environment on which generation/storage depends, and that environment is controlled by the interested party; Article 16 of the *Online Litigation Rules of the People's Courts* (《人民法院在线诉讼规则》), "presumed authentic after on-chain, separately reviewed before on-chain" [S31], precisely makes the unilateral generation step the weak point of review. Therefore neutral external anchoring of the disclosure hash is not a value-added feature but the condition for the evidentiary effect of this field in dispute scenarios — this is the common evidentiary-law basis for the independence requirement of this chapter and for the prohibition on merging the four segments in §3.4. **v0.2 extension**: the admissibility-grade claim for D2 source-disclosure traceability (§4.7.2) presupposes a complete custody path — the same argument covers the source-hash anchoring of the transit link.
- **Narrowed positioning**: the Custodian = an operator of a CT-style append-only hash log, performing only "admission + public audit"; it **does not store plaintext, is not a timestamp authority, and does not adjudicate liability**.
- **Timestamps**: overlay an RFC 3161 TSA [S13], preferring a domestic judicial certification service (e.g., UniTrust timestamp tsa.cn [S16]) — the TSA is a component supplier, not a competitor.
- **Independence**: the custody system **MUST** be mutually backed up by ≥2 independent operators (CT multi-log + gossip model [S14]); the relationship between the Custodian and the platform/device/OS parties **MUST** be publicly declared. During the pilot, a single log may go first, but that degraded state **MUST** be written into the Custodian's public declaration (consistent with the grading of the Implementation Status Statement in §12).
- **Admission audit**: following the CT inclusion proof model [S14], a third party can verify that "a given hash has indeed been placed in custody"; a Custodian refusing admission = externally detectable.
- **Lower bound on custody-log content**: only hashes and timestamps are recorded; `do_id`, per-receipt nonce, and any plaintext element **MUST NOT** be recorded — `do_id` exists only in the plaintext copies of the two parties facing off in a dispute; the correspondence between the nonce and the rendered content is likewise forbidden to record (the §4.7.1 privacy red line). This is the entropy premise for "the public log is auditable while the plaintext cannot be reconstructed by enumeration" (§3.1, §4.7.1; threat model in §11.1). **v0.2 extension (C.8 adopted)**: the delivery receipt hash (the final value of D1 `content_hash` / D2 `rendered_content_hash`) **SHOULD** be included in the custody log (receipt hashes in custody) — its entropy premise is the same as §3.1: the computation input contains a high-entropy `receipt_id` (UUID) and a per-receipt nonce, so after double high-entropy injection a low-entropy disclosure is not feasible to enumerate by dictionary.
- **Qualification path**: pilot custody of hashes (non-personal data) has no qualification threshold; a dispute-freeze evidence package can be sent at any time for notarization/judicial appraisal to upgrade its effect.
- **Architectural basis against covert modification**: the Custodian's trustworthiness does not rest on the operator's self-discipline but on the CT-style append-only + inclusion proof + gossip three-party-verifiable structure — this is the key statement of "trust-minimized custody" [S14].

---

## 9. Legal Effect and Rules of Evidence (Anchored in Chinese Law)

### 9.1 Two-Track Electronic Signature Regime

- **The IG-Lite track (Lite = customer-service adjudication grade)**: Article 13(2) of the *Electronic Signature Law* (《电子签名法》) — "the parties may also choose to use an electronic signature that meets the reliable conditions agreed upon between them" [S27]. A platform adopting IG-Lite **SHOULD** invoke this paragraph in its user agreement, stipulating this specification's rules of evidence as the "agreed reliable conditions" (and lock the version via `evidence_rules_version`, §3.1). CA certification is not a premise of reliability (Article 16: third-party certification is an optional service).
- **Core v2.0, current text**: endpoint private-key signatures (with in-SE signing as production design intent in Core Chapter 12). Core v2.0 does **not** define a CA-certified e-signature path for legal effect. The CA route under the PRC *Electronic Signature Law* (Article 13(1); third-party certification as an optional service under Article 16) is deferred to a future financial-grade Profile — until then, there is **no Core chapter to cite** for financial-grade legal effect, and readers MUST NOT be directed to "see Core".
- Article 14 of the *Electronic Signature Law*: a reliable electronic signature has the same effect as a handwritten signature or seal [S27].
- **Standard-form clause defense (R-19)**: the citation in the user agreement is a **standard-form clause**. Under Article 496(2) of the *Civil Code* (《民法典》), where the party providing the standard-form clause fails to perform its duty to prompt or explain, such that the other party does not notice or understand a clause that has a material interest to it, the other party may claim that the clause does not become part of the contract [S40]. The contractual-form track therefore requires:
  1. The agreed clause (the Article 13(2) citation + the evidence-rules version) **MUST** be prompted in a conspicuous manner — presented on a separate page or bold-marked + a standalone confirmation action; it **MUST NOT** be buried at the tail of a long agreement;
  2. The prompt-confirmation action itself **MUST** enter the audit-layer record — evidence that "the prompt occurred" is also carried by this specification's evidence chain (the agreement attests to its own legal foundation);
  3. Effect-attribution statement: any defect in the agreed effect caused by the platform's failure to perform its prompting duty is borne by the platform itself; this specification's evidence chain does not compensate for that defect.

### 9.2 Statutory Mapping Table

| Protocol mechanism | Statutory anchor | Mapping |
| --- | --- | --- |
| Activation-layer binding + exclusive key control (§3.5/§7.4) | Article 13(1), items (1) and (2), of the *Electronic Signature Law* | Exclusive control of the signature creation data; exclusive control at the time of signing |
| Grant signature + hash chain (§3.4) | Article 13(1), items (3) and (4), of the *Electronic Signature Law* | Changes to the signature and the data message are discoverable |
| User-agreement citation + version lock (§3.1) | Article 13(2) of the *Electronic Signature Law* | Agreed reliable conditions |
| Effect of an Agent concluding a contract | Article 48 of the *E-Commerce Law* (《电子商务法》) [S28] | The act of an automated information system is effective against the user |
| Consent-moment proof complements the contract-formation time | Article 491 of the *Civil Code* (《民法典》) [S29] | IG-Lite proves the authenticity of the authorization expression of intent before order submission (acceptance) |
| Disclosure of sensitive personal data access (P2 health, etc.) | Articles 28 and 29 of the *Personal Information Protection Law* (《个人信息保护法》) | `sensitive` marking (§3.1) + separate-consent semantics |
| Segment-by-segment traceability of the four-segment Accountability Chain (§3.4) | Article 93 of the *Provisions of the Supreme People's Court on Evidence in Civil Proceedings* (《民事证据规定》) | Comprehensive assessment of system-environment integrity |
| Custody hash log + pre-on-chain link (§8/§3.4) | Articles 16–19 of the *Online Litigation Rules* (《在线诉讼规则》) | Presumed authentic after on-chain; separately reviewed before on-chain |
| Resolution of the tension with the right to erasure (§9.4) | Article 4(1), Article 47 paragraph 3, and Article 73(4) of the *Personal Information Protection Law* | Stop processing; personal information does not include information after anonymization (Article 4(1)); the definition of anonymization is in Article 73(4) |
| Lower bound of audit-layer retention | Article 21 of the *Cybersecurity Law* (《网络安全法》) [S33] | Network logs ≥6 months |
| Retention of platform transaction information | Article 31 of the *E-Commerce Law* [S28] | ≥3 years from transaction completion |
| Burden of proof for authorized transactions | Supreme People's Court judicial interpretation on bank cards | The asserting party must submit the transaction-verification information at the time the transaction occurred [S34] |
| Voice auxiliary signal leaves no audio | Article 28 of the *Personal Information Protection Law* | Voiceprint is not a traceability subject (§5.2 L1 row) |
| **Disclosure-baseline `required` fields (§6.4, v0.2)** | Article 10 of the *Consumer Protection Regulation* (《消保条例》) [S55]; Article 18 of the *Measures for the Supervision and Administration of Online Transactions* (《网络交易监督管理办法》) [S56]; Article 17 of the *E-Commerce Law* [S48]; Article 13 of the *Price Law* (《价格法》) [S45]; Articles 20, 21, and 25 of the *Consumer Protection Law* (《消保法》) [S49] | Conspicuous reminder for auto-renewal; reminder five days before renewal; comprehensive, truthful, accurate, and timely disclosure; clearly marked prices; true-name labeling; true and comprehensive information; seven-day no-reason return (for §6.4.1 T-AIR, additionally cite Article 23 of the *Provisions on the Administration of Passenger Services in Public Air Transport* (《公共航空运输旅客服务管理规定》) [S54]) |
| **N/A truthfulness (§6.3, v0.2)** | Article 19 of the *Consumer Protection Regulation* [S50] | Must not unilaterally expand the range of goods not eligible for no-reason return — fabricating an exemption is "unilateral expansion" |
| **Judicial "conspicuousness" review standard (§6.4.2, v0.2)** | Supreme People's Court consumer-protection typical case No. 1 (2026-03-15) [S47]; Guangzhou Internet Court judgment (2025-08) [S46] | Adjudication rule for failure to give a conspicuous reminder for auto-renewal; precedents create no rights but provide the judicial review standard applied to schema fields |
| **Transit-fidelity admissibility (§4.7.2, v0.2)** | Articles 16–19 of the *Online Litigation Rules* (same as above) | Integrity-review path for source-disclosure custody anchoring |

### 9.3 Retention Tiers (N Parameter)

The mechanism and the lower bound are defined by this specification; the specific values are tuned by industry Profiles and locked via `evidence_rules_version`.

| Scenario | Default disclosure plaintext N | Basis for the lower bound |
| --- | --- | --- |
| General consumer confirmation | 90 days | Platform customer-service adjudication window |
| Durable consumer goods / three-guarantee items | ≥ three-guarantee validity period (typically 1–2 years) | Coverage of the three-guarantee dispute period |
| Financial confirmations | ≥ industry statutory retention period | Determined by regulatory requirements; the specification does not presume to set the number |

| Evidence layer | Retention | Basis |
| --- | --- | --- |
| hash + signature chain | Permanent (non-personal data) | hash is a one-way digest, and the computation object contains a ≥128-bit random `do_id` (§3.1) — after high-entropy injection, dictionary enumeration of low-entropy content is computationally infeasible; Article 4(1) of the *Personal Information Protection Law* explicitly provides that personal information **does not include** information after anonymization (definition of anonymization in Article 73(4)) — the main statutory anchor for "unidentifiable information is not personal information." The same argument in v0.2 extends to delivery receipt hashes (the computation input contains a high-entropy `receipt_id` and a per-receipt nonce, §4.7.1/§8) |
| Audit-layer log | ≥6 months (mechanism data such as the audit index and the Controller-local integrity hash chain — this chain is the audit layer's own integrity chain, not the §8 Custodian hash log, which is permanent, see the row above) | Article 21 of the *Cybersecurity Law*; the disclosure-plaintext portion (rendering traceability) is subject to the N parameter of this section, and after N expires only its hash remains (§3.3) |
| Platform transaction information | ≥3 years | Article 31 of the *E-Commerce Law* |

**Party responsible for storing the disclosure plaintext**: the platform side (order records, Article 31 of the *E-Commerce Law*) and the user side (the Grant copy containing the disclosure snapshot) are the principal responsible parties; retention of disclosure copies and rendering traceability in the AIOS/Controller app **SHOULD** be ≤N, and after expiry only the hash remains. When a deletion request under Article 47 of the *Personal Information Protection Law* arrives, what the Controller app deletes is the plaintext and the rendering traceability — the hash chain is unaffected.

**Evidence reconstruction in judicial scenarios (R-18)**: the design basis for the disclosure plaintext expiring after N days is the **customer-service adjudication window**, not the statute of limitations — the ordinary civil statute of limitations is three years (Article 188 of the *Civil Code* [S39]). For judicial disputes beyond the N window, evidence is reconstructed along the following path; this specification states this trade-off explicitly:

1. The disclosure plaintext held by the platform or Controller has expired and been destroyed under the N parameter, while the hash-only chain held by the Custodian is **retained permanently**;
2. In a dispute, the **party holding the plaintext** presents the content (platform order records are bound by the three-year retention requirement in Article 31 of the *E-Commerce Law*; the user-side Grant copy contains the disclosure snapshot), and its hash is computed and aligned against the `disclosure_hash` on the custody chain — a match proves that "the content disclosed at the time is consistent with what is presented today"; this is the "hash now, reveal later" architecture;
3. If the platform side also has no plaintext, or alignment fails, the party bearing the burden of proof bears the adverse consequence under the rules of evidence (the direction is consistent with the Supreme People's Court bank-card judicial interpretation: the party asserting an authorized transaction must prove it [S34]) — the burden of proof is pressed onto the party controlling the records, not the consumer.

### 9.4 Dispute Freeze (the "Stop Processing" Semantics)

When either party submits a claim to the platform's dispute process and the custody log receives a freeze request → the original disclosure segment enters a **stop-processing state** (Article 47 paragraph 3 of the *Personal Information Protection Law* [S32]): inaccessible except for judicial/arbitral invocation; the N expiry does not apply during the freeze; the freeze event itself enters the audit layer.

**This is not extra-legal traceability; it is lawful deactivation** — what is traced is the hash (non-personal data), and during the freeze the plaintext is only stored and security-protected, used for no purpose whatsoever.

---

## 10. Scenarios and Design Constraints

### 10.1 Confirmation Economics (Anti-Fatigue)

> This section is the foundation clause of IG-Lite: **the entire value of the protocol rests on "confirmation being meaningful."** If confirmation lacks frequency and content cost control, it is equivalent to an attack surface (Uber 2022 MFA fatigue compromise, 72% ignoring cookie banners; empirical evidence in 10.1.4).

#### 10.1.1 Confirmation Action Pattern

A valid consent signal **MUST** be one of the following patterns, and **a single click does not constitute a consent signal**:

- **Double click**: the interval between the two clicks ≤600ms (default; industry Profiles may tighten it; the Core v2.0 §5.5 firmware threshold is ≤350ms, which industry Profiles may tighten to align with);
- **Long press**: duration ≥800ms (default; aligned with the firmware recognition of Core v2.0 §5.5);
- Declared equivalent substitute (accessibility, etc., §5.5).

**Precedents for Presence Signals vs. Consent Signals**:

| Signal type | Precedent | Action | Prevents |
| --- | --- | --- | --- |
| presence | FIDO CTAP2.3 UP [S6] | single touch | malware silently signing |
| consent | Apple Pay side-button confirmation [S8][S9], Ledger dual-button simultaneous press [S10] | double click / dual button | accidental touch + fatigue-induced unconscious confirmation |
| consent | Microsoft Authenticator number matching [S23] | entering a mapped number | fatigue bombing (Uber 2022-style attacks [S21]) |

What IG-Lite captures is **consent**, so the action pattern **MUST** be higher than a presence signal — this is the normative basis for the difference from FIDO (single touch is compliant). The weak-surface equivalent of number matching = the structured triple announcement of §3.2: the content the user hears itself carries the transaction elements, and a press is consent to that content.

#### 10.1.2 Rate-Limiting Parameters (Defaults)

- `confirm_per_session` defaults to ≤5 (single Agent, single session); on exceeding the quota → the Agent **MUST** apply for a standing authorization via Access Grant or stop requesting;
- **Meta-confirmation exemption**: the Confirm Grant for an Access Grant / downgrade request / unlock-type request itself does **not** occupy the `confirm_per_session` quota (otherwise, once the quota is exhausted, the Agent could never apply for an upgrade, creating a deadlock) — but meta-confirmation still goes through the full disclosure-announcement-confirmation flow and is recorded in the audit layer;
- The quota counter is maintained locally by the Controller App and entered into the audit layer (to prevent self-deception on the Agent side); DCC-A/B devices **SHOULD** count synchronously on the PAE device side and report via signed events;
- For the same `disclosure_hash`, the exemption window T defaults to 10 minutes (for the semantics of the exemption, see 10.1.3);
- A repeated confirmation request for the same disclosure within 30 seconds → the Agent is in violation, recorded in the audit layer.

#### 10.1.3 Announcement-to-Press Window Binding

- A press is valid within ≤60 seconds after the disclosure announcement completes; a press during the announcement = invalid (silent period);
- **Repeated-disclosure exemption = exemption from repeated announcement, not from physical confirmation**: when the same `disclosure_hash` is requested for confirmation again within the T window, the Agent **MAY** omit the repeated announcement, but the user's physical confirmation action (§10.1.1) **MUST NOT** be omitted, and the audit layer **MUST** record a `repeat-announcement-exempt confirmation` (traceable, non-repudiable, not reduced). Rationale: what the exemption saves is the "cost of repeatedly presenting content"; if the consent signal itself could be exempted, that amounts to obtaining a standing-authorization effect by bypassing the Access Grant issuance flow of §4.3, and the rate-limiting system is hollowed out;
- Consequence of violation: confirmation-fatigue behavior enters the Agent reputation (a later-version field, reserved in v0.2).

#### 10.1.4 Design Basis (Empirical Anchors, Informative)

- **Real MFA fatigue compromise**: Uber, 2022-09 — the attacker bombarded MFA push and impersonated IT support; a contractor eventually approved, leading to a global compromise [S21]; the Akira ransomware gang was still using the same technique in 2025 [S22] — without frequency and content cost control, confirmation is equivalent to an attack surface.
- **Quantified consent fatigue**: 72% of users ignore or randomly click cookie banners (Flowconsent 2026) [S24]; about two-thirds of user behavior does not change with banner design (CookieYes) [S25]; average dwell time about 7 seconds (cited via [S26]) — high frequency + homogeneous + no content-mapping cost = confirmation by no one.
- **Concluding design proposition**: **the value density of confirmation = confirmation frequency⁻¹ × content-mapping cost**. IG-Lite controls from both ends at once: rate limiting (10.1.2) + content mapping (§3.2 triple) and attention binding (§3.2 cross-surface tail code) + action-pattern cost (10.1.1).

### 10.2 Scenario × Minimum LoA

| Scenario | Minimum LoA | Notes |
| --- | --- | --- |
| Instant retail ordering (P3) | L1.5 (default) | For amounts ≤ the order of the dual-exemption limit, may drop to L1 + platform risk control; over the limit, L2 |
| Memory/schedule read (P2) | L1 | Access Grant + quota limit |
| Memory export / outbound | L2 | DCC-A device |
| Memory deletion / irreversible operation | L2 + confirm_always | Never falls into Access Grant (§4.1) |
| Basic voice-scenario confirmation (P1) | L1 + Presentation Commitment | Voice is an auxiliary signal only (§5.2) |
| Subscription change / external authorization | L1.5 | The structured triple must include the rate limit |
| Financial payment | L3 (Enterprise) | Composes AP2 Payment Mandate; out of scope for this Profile |

**Orthogonal relationship with the disclosure baseline (v0.2)**: this table requires LoA by **scenario**; the §6.4 type schemas declare a minimum LoA×D combination by **transaction type**; at execution, take the max (§5.6) — the scenario table and the schema combination do not substitute for each other. This table does not repeat the D-level defaults; D-level requirements follow §5.6 and §6.4.

### 10.3 Precedent Citations (The "Non-Radical Innovation" Argument, Informative)

- **LoA × quota tiering**: UnionPay dual exemption (per-transaction limit + full compensation for risk), central bank Class III accounts (balance ceiling isolates risk exposure), digital RMB offline payment (direct debit within the no-password quota, password over the limit) [S36][S37] — in 20 years the financial system has never accepted the "no confirmation / all confirmation" binary; instead it is a three-layer design of "quota × confirmation method × compensation backstop." IG-Lite protocolizes this logic and extends it to non-payment scenarios.
- **Physical confirmation**: Apple Pay side-button double click [S8] (consumer-grade user education is already complete — the habit-migration cost of "physical button = I am confirming" ≈0); hardware-wallet WYSIWYS [S10][S11].
- **Pairing window opening**: Matter commissioning physical-action window [S12] (the prevailing anti-hijack mode for IoT, which hardware vendors already understand).
- **Machine-readability of disclosure obligations**: the "conspicuous reminder five days before" in Article 18 of the *Measures for the Supervision and Administration of Online Transactions* is itself the regulator making the reminder timing programmatic [S56]; the civil-aviation tiered-fare publicity is a regulatory tabular requirement on the disclosure structure [S52] — the schema extends the regulator's existing "programmatic disclosure" tradition to the machine-verifiable layer (§6.6).
- For financial and platform precedents on the adjudication side, see the companion document *Adjudication Economics (One Page)*.

---

## 11. Security Considerations and Known Limitations

### 11.1 Threat Model Summary (Attack Vector × Mitigation × Residual Risk)

| Attack vector | Mitigation clause | Residual risk (honest boundary) |
| --- | --- | --- |
| Disclosed content does not match signed content (tail-code collision / substitution of announced content) | Mandatory format of the Presentation Commitment: cross-surface tail-code verification (requires a second trusted surface) or a structured triple; a bare single-surface tail code MUST NOT be used (§3.2) | a) The 8-character tail provides 32-bit collision resistance; GPU enumeration is feasible at minute scale — provided the second presentation surface and the challenge issuer do not fall together. When the second surface and the issuer are the same entity (for example, when the Controller App screen serves as the second surface), the tail code retains only attention-binding semantics and does not constitute an integrity check against that party; content integrity is detected after the fact by audit-layer rendering traces and core-field consistency comparison (VIOLATED) |
| Dictionary enumeration of disclosure hashes (reconstructing transaction content from the public custody log) | A `do_id` of ≥128 bits of randomness is included in the hash computation; the Custodian MUST NOT record `do_id` (§3.1/§8) | None (computationally infeasible) |
| **Dictionary enumeration of delivery receipt hashes (v0.2)** | All content hashes (D1 `content_hash` / D2 `rendered_content_hash`) are mixed with a per-receipt nonce (≥128-bit device CSPRNG); `receipt_id` is high-entropy; the Custodian MUST NOT record the nonce↔content correspondence (§4.7.1/§8) | None (computationally infeasible — the same argument as for do_id) |
| **Renderer misreports dwell time ("rendered for 1 second, signed as 8 seconds", v0.2)** | Dwell-time probative weight is tiered: for endpoints with the secure-clock capability bit (`secure_clock`), both timestamps enter the proof object; for endpoints without that capability bit, dwell time degrades to a claimed field and does not participate in proof (§4.7.1/§5.1) | Under the claimed path, dwell time is not provable — the D1 proof of a "significant viewing opportunity" narrows to "the rendering event occurred within the window"; the relying party determines proof strength from the endpoint's capability bits |
| **Source platform re-forges the source hash to feign a match (v0.2)** | Mismatch arbitration is anchored to the latest custody anchor before the rendering time point; receipts on the trusted-timestamp-only path are limited to audit reference and MUST NOT support relying-party-grade assertions (§4.7.2) | TEE-side collusion (the source platform and the renderer colluding to forge) cannot be eliminated — an honest boundary; reliance on receipts depends on the multi-party independence of the chain |
| **Deliberately bypassing the SDK (hand-assembling credentials without issuer-side validation, v0.2)** | Verifier-side arbitration: a structurally non-conforming Grant is judged invalid (§6.3, ring 3); on the legal side, non-conforming disclosure loses evidentiary effect under law (§6.3, ring 4) | Issuer-side validation guards only against unintentional non-conformance — a quality filter, not an admission gate (§6.3, ecosystem-position boundary); verifier-side arbitration depends on courts and Custodians accepting it |
| Replay (forwarding a cached offline keypress) | Real-time challenge binding + window validation; offline keypresses are discarded (§3.3) | None (cached forwarding is an optional enhancement for a later version, with a downgrade marker) |
| BLE relay | challenge-response Timing Binding; proximity is not treated as a security property (§3.5) | No reliance on distance assumptions; UWB ranging is optional for a later version |
| BLE pairing hijack (Just Works has no MITM protection) | Activation-layer pairing MUST use LE Secure Connections + NC/OOB; the JW fallback caps at L1 + manual re-verification after binding (§3.5) | The existence of the JW fallback path is itself a usability concession; LoA is already capped |
| Controller app forges physical events (the phone is both the hub and the authorized party) | L2+ MUST use device-side key signatures; `attested_by: "app"` MUST be labeled explicitly, and DCC-C caps at L1.5 (§3.5/§5.1) | In DCC-C scenarios, compromise of the app domain means signatures can be forged; the LoA ceiling is honestly bounded |
| TOCTOU (confirm X, execute Y) | The receipt is re-hashed against the disclosure; a mismatch means the credential chain provides no proof of authorization plus a record of unauthorized execution (§4.5) | None (hard rule) |
| Confirmation fatigue (high-frequency bombardment → unconscious approval) | Action pattern (double press / long press; a single press is ineffective) + rate-limiting quota + announcement-to-press window binding (§10.1) | A user may still press without deliberation within the window — rate limiting compresses but does not reduce this to zero; Agent reputation is for a later version |
| **Statutory transition window (the schema lags statutory amendment, v0.2)** | An N-day evaluation mechanism + maintenance of prior-version credential effect within the window + disclosure of overdue items on the Registry public status page (§6.5) | Within the window, the lag between "the new law should be known" and "the schema has not yet followed" is borne by the mechanism and disclosed publicly — it is not denied by the protocol |
| Coercion | Not covered by this specification (§11.2) | All |
| Misattribution on shared devices | Shared pairing / absence of wear detection caps at L1.5 (§5.4) | In L1.5 scenarios, misauthorization arising from "the party physically holding the device ≠ the principal" cannot be eliminated |
| Device capability overstatement | Honest-declaration obligation for the Capability Descriptor + capability bits subject to the same overstatement prohibition + historical Grants from an overstated device are adjudicated downward (§7.2/§12) | Depends on the companion audit (not conducted, §12) |
| Custodian misbehavior / compromise | CT-style append-only + inclusion proof + multi-log redundancy + conflict-of-interest disclosure (§8) | Operator admission-level collusion cannot be fully excluded — "covert tampering is detectable" ≠ "no wrongdoing" |
| Invalidity of standard-form clauses (the agreement is held not to become part of the contract) | Prominent notice + independent confirmation + the notice action recorded in the audit layer (§9.1, R-19) | Execution quality on the platform side is not guaranteed by the protocol; the consequences of defects rest with the platform |
| Evidence gap (judicial reconstruction fails after the source text expires) | Permanent hash retention + platform §31 retention + hash alignment path + allocation of the burden of proof (§9.3, R-18) | If the source text is also missing on the platform side, the disclosed content cannot be reconstructed |

### 11.2 Known Limitations

- **Coerced keypress not covered**: physical possession ≠ free will; this specification defines no coercion signal (banks have no universal scheme either); a later version reserves a double-long-press coercion flag.
- **Unavailable offline**: confirmation depends on a real-time challenge; offline keypresses are discarded (cached forwarding + a downgrade marker are later-version topics).
- **Signature-grade binding on the audio side not covered**: audio evidence stops at the audit level (§3.3 rendering traces: the audio-track hash records the fact of announcement, while the text hash carries the consistency comparison); bringing a normalized announcement script into the Grant signing object requires first defining a TTS normalization object and engine cooperation requirements — a later-version topic.
- **~~Disclosure sufficiency (Adequacy) not in scope~~ → closed in v0.2 (§6)**: the Disclosure Baseline Schema Registry renders current statutory disclosure obligations machine-readable by transaction type; the `required` field is doubly enforced through issuer-side validation (issuance refusal) and verifier-side recomputation (judged invalid); statutory mapping and verification status are in §6.6. **Retained limitations**: ①Registry coverage is limited to registered types (T-0/T-AIR/T-SUB at launch; unregistered transaction types fall back to T-0 or a no-schema state); ②the transition window in which the schema lags statutory amendment (§6.5/§11.1); ③fields with no public statutory source, such as `dispute_entry`, are at the recommended level only (§6.4.0).
- **~~Deliverability stops at the audit level~~ → partially closed in v0.2 (§4.7/§5.6)**: D1/D2 device-signed delivery receipts raise the proof chain against the "storage ≠ display" attack from platform self-attestation (D0) to the device-signature level. **Retained limitations**: ①DCC-C endpoints have no device-signing capability, so delivery proof remains at the D0 audit level (§4.7.1); ②dwell time at endpoints without a secure clock is a claimed field and cannot be proven (§4.7.1); ③D2 source-disclosure tracing depends on commercial cooperation from the source platform; D2 is unavailable before that integration (§4.7.3); ④receipts on the timestamp-only path are limited to audit reference (§4.7.2). **Delivery receipts carry no consent semantics**: consent is carried solely by IntentGrant, and no user action in a receipt constitutes a consent signal within the meaning of §10.1.1.
- **DCC-C caps at L1.5**: an honest label that gives the supply chain an upgrade ladder ("DCC-B/A must be passed to reach L2"), not an overstatement.
- **Surveillance paradox**: personal information added by this protocol ≈ 0 (hashes are not personal data + source text retained for N days + the "stop processing" mechanism under Article 47 of the *Personal Information Protection Law*) — see §9.3/§9.4. The v0.2 delivery receipts and nonces follow the same argument (§8 Custodian non-recording clauses).
- **Fairness of disclosure templates is outside the protocol's domain**: whether rendered text constitutes a misleading statement (dark pattern) is a matter of platform governance and consumer-protection review; this protocol guarantees only that "what was presented at the time" can be proven after the fact. **v0.2 boundary clarification**: the schema's `required` fields specify that "what should be said was said" (completeness of the information set) and do not review "whether the manner of saying it misleads" — the two are complementary and non-overlapping.
- **Minors not covered**: a PAE holder may be a person with limited capacity for civil conduct (Article 19 of the *Civil Code* (《民法典》)); the shared-device caps of §5.4 compress LoA but do not resolve the attribution of capacity — a later version will consider UV/age gating (a guardian delegation chain is a v2.1 candidate and does not enter this version).
- **mandate_ref one-way reference**: the AP2 ecosystem does not recognize IG-Lite; semantics are compatible and fields are mappable, but interoperability is not claimed.
- **Standards-political risk**: if the FIDO AA TWG publishes a conflicting capture-assurance specification → ecosystem fragmentation risk; this specification's response is terminology alignment (not confrontation) + a Chinese-law anchor + differentiated positioning in weak-surface scenarios, and submission of comments to the TWG (future work).

---

## 12. Implementation Status Statement

This chapter is a material part of this specification. **All external materials, security reviews, and business communications are governed by this chapter** (aligned with the Chapter 12 discipline of Core v2.0: strict separation between the specification's target state and the current implementation scope).

| Capability item | Status | Notes |
| --- | --- | --- |
| Specification text (this document) | Available (this document) | v0.2.1 draft, open for community review; cross-reference errata only, with mechanism semantics unchanged from v0.2 |
| DCC-C reference implementation form | Available (prototype level) | Off-the-shelf general-purpose Bluetooth remote button + Controller app: `attested_by: "app"`, LoA capped at L1.5 (Appendix B) |
| Disclosure Object / Grant field structure and flows | Available (pseudocode level) | The field structure and flows are defined by the specification; the companion minimal reference implementation (pseudocode level) is in the companion file *IG-Lite Reference Implementation — Pseudocode* (《IG-Lite 参考实现-伪代码》) (extended in step with v0.2: `required` fail branch, fidelity validation, nonce hash computation) |
| IG-MCP Interface Definition v0.1 | Available (interface-definition level) | MCP tool-layer semantics and data model (a separate document). Tool implementation status is declared by grade (A/B/C) per its §9; some tools are already hosted by a live service (see next row) |
| ProofMesh origin-api v0.3 (Confirm Grant issuance / consumption / state machine) | Available (running live) | The four-state semantics of Ed25519 + RFC 8785 JCS + single-use TTL are implemented and independently verifiable; demo keys (derived from a fixed seed, publicly declared); in-process memory storage (lost on restart, declared as such) — productionization requires a switch to persistent storage and HSM/KMS |
| Schema Registry data model (§6) | Semantic design | The three schemas T-0/T-AIR/T-SUB are defined by the specification; statutory mapping has passed the 2026-09-20 source-text verification batch + the same-day final verification batch (8/10 confirmed verbatim from multiple official sources + four items cleared against official full-text pages; the remaining one compared against the official website full text; status in §6.6); open-source governance of the Registry has not started |
| Issuer-side fail branch for `required` validation | Pseudocode level (to be implemented) | Defined in §4.2/§6.3; extended with the companion pseudocode v0.2 |
| D1/D2 delivery receipt credentials (§4.7) | Semantic design | The device-signing channel depends on DCC-B or higher endpoints (mass-produced secure hardware); source-disclosure tracing integration depends on commercial cooperation from the source platform |
| DCC-A / DCC-B device paths (SE/TEE, device-side signing) | Design intent | Depends on mass-produced secure hardware |
| AP2 mapping of mandate_ref | Semantic design | No interoperability testing against an AP2 implementation has been performed; only semantic compatibility is claimed |
| Evidence Custodian (CT-style hash log) | Design intent | An independent operator has not been established; a pilot may start with a single log |
| A2A auth-required integration | Future work | Depends on adoption by Agent platforms |
| Submission of comments to the FIDO AA TWG / Chinese standards drafters | Future work | After publication |
| Third-party security audit and penetration testing | Not conducted | — |
| Large-scale deployment validation of DCC-A/B | Not conducted | No external POC or production deployment |

**Prohibition of overstatement**: for any device whose device capability claim (§7.2, including v0.2 capability bits) does not match its actual mechanism, the evidentiary weight of all of its historical Grants and receipts is adjudicated downward according to the actual mechanism; presenting design intent as an implemented capability constitutes a violation of this specification. **Capability status ≠ specification definition**: the credentials and schemas of §4.7/§6 are defined by the specification (target state), and their implementation status is governed by this chapter — external materials MUST NOT present "defined by the specification" as "implemented".

---

## 13. Version History

| Version | Date | Principal changes |
| --- | --- | --- |
| v0.1 | 2026-09-12 | First release: Consumer Profile positioning, Disclosure Object and Presentation Commitment, Confirm/Access Grant, mechanism-graded DCC/LoA, CT-style evidence custody, Chinese-law anchor, confirmation-economics constraints. Includes the 2026-09-09 red-team final-check revisions (R-18 judicial-scenario evidence reconstruction, R-19 standard-form-clause defense, R-20 security considerations chapter, correction of repeat-announcement-exemption semantics, exemption of meta-confirmation from quota); includes same-day revisions from three simulated reviews (Presentation Commitment semantics restructured into cross-surface tail-code verification / structured triple, high-entropy `do_id` introduced into hash computation to prevent enumerative reconstruction, Grant signing computation object and device-side computation rules, clarification of retention tiers for rendering traces, prohibition on storing voiceprints, clarification of unauthorized-execution wording, `sensitive` marking of sensitive data, allocation of responsibility for storing the disclosure source text, and others); includes same-day regression read-through revisions (alignment of two leftover wordings in Appendix C, consistency of the `challenge` object fields with §3.3, completion of Access Grant signing rules, `sensitive` added to hash core fields, clarification of the residual risk of same-entity Presentation Commitment, source citations appended to statutory mapping rows); includes the 2026-09-10 cold-read adjudication revisions (JW binding fact-record field `association_model`, normative definition of confirm-required, correction of the primary anchor to Article 4 of the Personal Information Protection Law, completion of source URLs and description of the verification channel, delivery of the companion pseudocode reference implementation); includes 2026-09-11 reinforcement: new §2.7 ecosystem position among Chinese standards (mandatory standards / group standards / policy, sources S41–S44), custody chapter supplemented with the basis for disclosure-hash independence (disclosure-segment generator and interested party being the same entity → self-attestation trap → external anchoring as the condition for evidentiary effect), and the §2.6 inheritance alignment list narrowed to verifiable RFCs. |
| v0.1.1 | 2026-09-19 | Open-source release. **Five 2026-09-16 revisions back-registered** (separation of the two hash semantics for rendering traces, host-isolation wording for Presentation Commitment, convergence of status-semantics references, bridging of double-press timing with Core firmware thresholds, the audio signature-grade binding topic); **changes in this revision**: header upgraded to v0.1.1, license settled as Apache 2.0, roadmap section added, implementation status supplemented with IG-MCP Interface Definition and ProofMesh origin-api. **No semantic changes.** |
| v0.2 | 2026-09-20 | **Disclosure Baseline and Delivery Proof edition**. Added: ①**§6 Disclosure Baseline Schema Registry** (four-element conceptual model, Registry architecture and governance, four-ring enforcement chain + honest boundaries of ecosystem position, N/A explicit-declaration rules, Vendor Namespace, three type schemas T-0/T-AIR/T-SUB, versioning and forensic arbitration including T-SUB cross-version switching, statutory mapping and verification status); ②**§4.7 Delivery Receipt (D1) and Transit Fidelity Receipt (D2)** (delivery receipt and transit fidelity receipt; dwell-time probative weight tiered by the secure-clock capability bit; per-receipt nonce privacy red line; re-rendering hash chaining; mismatch arbitration baseline); ③**§5.6 Delivery tier and LoA×D combination matrix** (D0/D1/D2 definitions, orthogonal adjudication taking max, type-level default combinations, default recommendation for phone-assistant scenarios); ④§3.1 `schema_ref`/`delivery_tier` fields added to the hash, §4.2 pre-issuance validation `required` fail branch, §4.4 `rejected_schema` state, §5.1 capability matrix extension (`secure_clock`), §5.3 clause applying the silent-downgrade rule by analogy to delivery tiers. Revised: ⑤§3.3 rendering traces annotated as D0-aligned; ⑥renumbering of §7/§8/§9/§10/§11/§12/§13 (formerly §6–§12); ⑦§9.2 statutory mapping table supplemented with v0.2 mechanism rows (statutory source of schema `required` fields, N/A authenticity, judicial standard of conspicuous review, transit fidelity reliance); ⑧§11.1 threat model gains four rows (receipt hash enumeration, dwell-time misreporting, source hash re-forging, SDK bypass) + a statutory transition window row; ⑨§11.2 known limitations rewritten (the disclosure-sufficiency / deliverability items marked as closed in v0.2 with retained limitations listed); ⑩new terms in Appendix A, two rows added to Appendix C, new Appendix C′ (v0.1.1 → v0.2 delta list), sources extended to S45–S56. **Upstream inputs**: the design document *Disclosure Baseline Schema Registry* (告知基线 Schema Registry) v0.2 (including the 2026-09-20 source-text verification batch, 8/10 confirmed verbatim from multiple official sources) + the design document *DisplayReceipt* v0.1 (including red-team revisions) + the 2026-09-20 three-party simulated review (all P0×1 / P1×7 / P2×6 items dispositioned) + adoption of IG-MCP Appendix C C.1/C.8. |
| v0.2.1 | 2026-09-25 | **Cross-reference errata; no mechanism-semantic change.** Corrected six statements that attributed a CA-certified legal-effect path to Core v2.0; Core's current capability is endpoint private-key signing, while a CA-certified financial-grade route remains a future Profile. Disambiguated Core architecture Layer 0–Layer 3 from LoA L0–L3, including audit-layer terminology and pseudocode helper names. Made the PAE, Controller App, and Control Plane definitions self-contained instead of relying on a non-normative upstream glossary. |

### 13.1 Roadmap and Public Implementation

**Adopted in v0.2 (from the eight IG-MCP Appendix C proposals)**: C.1 (deliverability → §4.7 D1/D2 and §5.6 delivery tier) is one of the v0.2 main lines; C.8 (receipt hashes into the custody log → §8) is adopted, with the entropy premise declared alongside (the computation input includes a high-entropy `receipt_id` and a per-receipt nonce, the same argument as for §3.1 `do_id`). The semantics of C.2/C.3/C.4/C.5 are absorbed or superseded by the v0.2 structures of §4.7/§5.6/§3.1 (the `delivery_tier` field replaces the C.5 proposal to add a column to the scenario table); C.6 (explicit error code for no available PAE) / C.7 (alignment tolerance locking) are later-version topics.

**Candidates for later versions**: presence_mode (0x01 local BLE + proximity / 0x02 remote / 0x03 remote + location proof) and a delegation chain are **v2.1 candidates** (registered 2026-09-17, credential field extensions); cached forwarding, UWB ranging, the TTS normalization object, the coercion flag, UV/age gating, and C.6/C.7 are to be reviewed in the same batch.

**Public implementations** (implementation status in §12):

- **IG-MCP Interface Definition v0.1**: MCP tool-layer semantics (interfacing the confirm-required state), dual form (HTTP as the primary form + stdio as the secondary form), a three-role authentication model, and the Resources surface. A separate document, not merged into this specification (the transport-neutral discipline, §2.4).
- **ProofMesh origin-api v0.3**: the live implementation of the chain-origin service (Confirm Grant issuance / query / consumption / the four-state state machine, Ed25519 + RFC 8785 JCS + single-use TTL), publicly and independently verifiable (the public key and verification material are distributed self-contained with the credential). Its demo keys are derived from a fixed seed (publicly declared via key_is_demo); a production implementation MUST replace key management (HSM/KMS) and disclose this truthfully.
- **Honest boundary (stated consistently with the companion demonstration system)**: the testable part of this specification and its implementation is **recomputability** (independent verification of the signed bytes, the disclosure hash, and the signature), not **adoption** (nothing has been submitted to any standards body; there is no third-party implementation); T2 (prevention of ex-ante fabrication) depends on an independent third-party disclosure source and has not yet been met — the neutral-custody requirements in §8 of this specification are reserved for exactly this.

---

## Appendix A: Glossary (Added by This Profile)

| Term | Definition |
| --- | --- |
| weak surface | Confirmation presentation environment with no screen, a small screen, or voice-first interaction (headsets, glasses, rings, in-vehicle); contrasted with the AP2 Trusted Surface (strong surface) |
| Disclosure Object | Carrier of the minimal information set for a confirmation request, comprising the canonical hash in which the high-entropy random identifier `do_id` participates, `schema_ref` (v0.2), and a version-locked reference to the evidence rules |
| do_id | Unique identifier of a Disclosure Object, generated by a ≥128-bit CSPRNG and participating in the `disclosure_hash` computation so that the original text cannot be reconstructed by enumeration (§3.1); it does not appear in the public custody log (§8) |
| `schema_ref` (v0.2) | Identifier (including version) of the Disclosure Baseline schema applicable to the disclosure, included in the hash; the verifier recomputes `required` completeness against it (§6.3) |
| Disclosure Baseline / MMDS (v0.2) | Mandatory Minimum Disclosure Set — the minimum content set required for conformance, rendering currently effective legal disclosure obligations as machine-readable schemas by transaction type (§6) |
| Schema Registry (v0.2) | Open-source type schema registry: versioned management of fields, `required` sets, N/A rules, and statutory mapping (§6.2) |
| `required` / `not_applicable` (v0.2) | Schema mandatory fields; an explicit exemption declaration (which **MUST** carry genuine legal grounds; silent omission, empty string, and null — the three states — **MUST NOT** be used) (§6.3) |
| Presentation Commitment | Mandatory format of the presentation binding information on a weak surface: cross-surface tail-code verification (which requires a second trusted surface) or the structured triple (§3.2) |
| structured triple | Presentation combination of amount (or rate limit) + counterparty entity name + the last 4 digits of `disclosure_hash` |
| DCC (Device Capability Class) | Device Capability Classes A/B/C (determined by the presence or absence of a security boundary); from v0.2, capability-bit declarations (e.g. `secure_clock`) may be attached (§5.1) |
| LoA (Level of Assurance) | Mechanism-graded assurance levels L0–L3; the highest target for Lite is L2 |
| Delivery Tier / Tier D (v0.2) | Disclosure delivery strength tiers D0 (render trace) / D1 (Delivery Receipt) / D2 (Transit Fidelity Receipt); an orthogonal axis independent of LoA (§5.6) |
| Delivery Receipt / D1 (v0.2) | Weak-surface delivery receipt: a device signature proving that "the render event occurred within the window" (§4.7.1); it carries no consent semantics |
| Transit Fidelity Receipt / D2 (v0.2) | Transit fidelity receipt: a three-segment chain proving that the transit rendering matches the source disclosure (§4.7.2); mismatch = VIOLATED |
| per-receipt nonce (v0.2) | Random salt of the delivery receipt content hash (≥128-bit device CSPRNG, included in the credential) — a low-entropy disclosure, after high-entropy injection, cannot be reconstructed by dictionary attack (§4.7.1) |
| `claimed` field (v0.2) | A renderer-self-reported field that participates in the hash for tamper resistance but is not part of the proof object (e.g. `render_duration_ms` on an endpoint without a secure clock) (§4.7.1) |
| Confirm Grant | One-time confirmation credential (single consumption) |
| Access Grant | Constrained standing authorization credential (quota and per-action LoA routing) |
| Receipt | Execution receipt; `executed_action_hash` is aligned with the disclosure (TOCTOU closure) |
| mandate_ref | One-way reference from a Grant to the `checkout_hash` of an AP2 Checkout Mandate |
| attested_by | Annotation of the subject signing the event: `device` (device self-attestation) / `app` (app self-attestation) |
| Accountability Chain | Four-segment hash chain of intent capture → disclosure → confirmation → execution (from v0.2, the disclosure segment includes D1/D2 delivery proof); it preserves evidence for responsibility attribution but does not itself determine legal liability |
| Custodian | Operator of a CT-style append-only hash log (stores no original text, adjudicates no liability) |
| N parameter | Number of days the disclosure original text is retained (a scenario-adjustable lower bound, locked by evidence_rules_version) |
| repeat-announcement-exempt confirmation | A confirmation mode that, within the same hash window, **skips repeat announcement but still requires physical confirmation**; it **MUST** be recorded in the audit layer (§10.1.3) |
| meta-confirmation | A confirmation about the confirmation system itself (e.g. the Access Grant issuance confirmation); it does not consume session quota (§10.1.2) |
| confirm-required | The confirmation request state defined by this specification (the equivalent consumer entry point for A2A auth-required) |

PAE, Controller App, and Control Plane are defined in §1.5; this appendix lists only terms newly introduced by this Profile.

## Appendix B: Reference Implementation Form (DCC-C)

**Form**: a commercially available generic Bluetooth camera remote shutter button (such devices have no security boundary and no firmware-level key, and report events over BLE HID/Notify) + a Controller app.

**Capability facts**: signing in fact occurs in the Controller app (`attested_by: "app"`); graded by mechanism per §5.2, such an implementation is **capped at L1.5**. This grading also yields a supply-chain upgrade ladder: DCC-B (firmware key + device-side signing) → L1.5 evidence may be marked `attested_by: "device"` and can carry a D1 Delivery Receipt (§4.7.1; without the `secure_clock` capability bit, the duration field is `claimed`); DCC-A (SE/TEE) → can reach L2.

**Release artifacts**: v0.1 ships an open-source minimal reference implementation (pseudocode level) — the companion document *IG-Lite Reference Implementation — Pseudocode* (《IG-Lite 参考实现-伪代码》), covering the minimal closed loop of Disclosure Object construction → announcement → challenge issuance → double-press event → Grant issuance → Receipt alignment; v0.2 extends it with the `required` fail branch, delivery receipt generation, and fidelity verification (§12, Implementation Status). All cryptographic primitives are delegated to platform libraries, and the honesty boundary is the same as in §12.

## Appendix C: Delta List Against Core v2.0

| Dimension | Core v2.0 (Enterprise) | IG-Lite v0.2.1 (Consumer) | Clause |
| --- | --- | --- | --- |
| Control plane | Enterprise administrator (report loss / force unbind / role handover) | Controller app + the user themself | §7.4 |
| Activation-layer pairing security | Not specified (implementation default) | LE Secure Connections + NC/OOB **MUST**; JW (Just Works) fallback capped at L1 | §3.5 |
| Activation-layer binding verification | Administrator verification | Manual device fingerprint verification before the first Grant | §3.5 |
| Event signing | Endpoint private-key signing (SE as design intent) | `attested_by` distinguishes device/app; L2+ **MUST** be device (app self-attestation capped at L1.5) | §3.5 |
| Disclosure semantics | Not defined | Disclosure Object + Presentation Commitment + render trace (D0) + Disclosure Baseline schema (§6) | §3.1–3.3 |
| Credential | Grant (operation-level, three risk-grading tiers) | Dual types Confirm/Access + loa_target + mandate_ref + confirm_modality + schema_ref | §4 |
| **Delivery proof (v0.2)** | Not defined | D0/D1/D2 delivery tiers and Delivery/Transit Fidelity Receipt | §4.7/§5.6 |
| **Disclosure Baseline (v0.2)** | Not defined | Schema Registry: machine-readable `required` + N/A rules + statutory mapping | §6 |
| Execution alignment | Audit-layer Result record | Receipt hash hard alignment; non-alignment = the credential chain provides no proof of authorization (recorded as unauthorized execution) | §4.5 |
| Assurance level | Implicit in risk grading | LoA × DCC mechanism-graded + prohibition of silent downgrade + LoA×D orthogonal matrix | §5 |
| Audit storage | Enterprise-side encrypted storage for 180 days | Neutral Custodian hash log; audit layer ≥6 months (mechanism data; the disclosure original text is subject to the N parameter) | §8, §9.3 |
| Legal track | Endpoint private-key signature (CA-certified track not defined, §9.1) | Contractual form (*Electronic Signature Law*, Article 13(2)) | §9.1 |
| Confirmation action | Double-press / long-press / triple-press (event semantics) | Consent signal specification: single press invalid + rate limiting + announcement pairing | §10.1 |
| Accessibility | Not specified | Alternative channel declaration included in the Grant + audit layer | §5.5 |

## Appendix C′: v0.1.1 → v0.2 Delta List (Field-Level)

> Baseline statement: this list uses **v0.1.1 (the 2026-09-19 open-source release)** as its baseline. v0.1.1 introduces no semantic change relative to v0.1 (errata and supplementary registration; see §13 Version History), so the v0.1→v0.1.1 delta is not repeated here; all v0.2 increments are listed item by item, each premised on capabilities already present in v0.1.1.

| # | Change | Type | Clause |
| --- | --- | --- | --- |
| 1 | Disclosure Object gains `schema_ref` (optional, included in the hash) | New field | §3.1 |
| 2 | Disclosure Object gains `delivery_tier` (optional, included in the hash) | New field | §3.1 |
| 3 | Grant gains `schema_ref`, `delivery_receipt_ref` | New field | §4.2 |
| 4 | Pre-issuance validation `required` fail branch | New flow | §4.2/§6.3 |
| 5 | State machine gains the `rejected_schema` pre-state | New state | §4.4 |
| 6 | New credentials Delivery Receipt (D1) / Transit Fidelity Receipt (D2) | New credential | §4.7 |
| 7 | DCC capability matrix gains the `secure_clock` capability bit | New capability | §5.1 |
| 8 | Prohibition of silent downgrade extended to delivery_tier | Rule extension | §5.3 |
| 9 | New Delivery tier definitions and LoA×D combination matrix | New section | §5.6 |
| 10 | New Disclosure Baseline section (Registry / enforcement chain / N/A rules / three schemas / version arbitration / statutory mapping) | New section | §6 |
| 11 | Render trace labeled D0 (no semantic change; renamed for alignment) | Wording | §3.3 |
| 12 | Sections §7–§13 renumbered (former §6–§12 shifted accordingly) | Structure | Whole document |
| 13 | Statutory mapping table gains v0.2 mechanism rows ×4 | Table addition | §9.2 |
| 14 | Threat model gains rows ×5 (receipt enumeration / duration misreporting / source hash re-minting / SDK bypass / statutory transition window) | Table addition | §11.1 |
| 15 | Two known limitations (disclosure sufficiency / deliverability) rewritten as "closed + residual limitation" | Rewrite | §11.2 |
| 16 | Implementation status table gains three rows: Schema Registry / D1D2 / `required` validation | Table addition | §12 |
| 17 | Glossary gains v0.2 terms ×9 | Table addition | Appendix A |
| 18 | Sources extended with S45–S56 (statutory verification batch + red-team forensics) | Sources | Sources |
| 19 | Capability Descriptor gains `profile_version` / `capability_flags` | New field | §7.2 |
| 20 | Custody log content floor extended: receipt hash enters custody (C.8 adopted) + nonce recording prohibited | Rule extension | §8 |

---

## Sources

> Citation discipline: statutory sources are governed by the currently effective text; official full texts can be verified uniformly through the National Database of Laws and Regulations (flk.npc.gov.cn). Entries marked with a domain name only are illustrative citations whose article texts were verified during the research phase (research/B_法律锚点.md). The statutory sources newly added in v0.2 (S45–S56) underwent multi-source cross-checking in the 2026-09-20 full-text verification batch (research/v02-factcheck-法源核验-2026-09-20.md); for verification grades and items pending flk, see §6.6.

- [S1] A2A task lifecycle and the auth-required state (AIXP-Labs/AIJP comparison document): https://github.com/AIXP-Labs/AIJP
- [S2] AP2 Agent Authorization Framework: https://ap2-protocol.org/ap2/agent_authorization/
- [S3] AP2 Checkout Mandate: https://ap2-protocol.org/ap2/checkout_mandate/
- [S4] FIDO Alliance establishes the Agentic Authentication Technical Working Group and accepts the AP2 donation (2026-04-28): https://fidoalliance.org/fido-alliance-to-develop-standards-for-trusted-ai-agent-interactions
- [S5] MCP Elicitation (2025-06-18 specification version): https://modelcontextprotocol.io/specification/2025-06-18/client/elicitation
- [S6] FIDO CTAP v2.3 (definition of User Presence as a single touch): https://fidoalliance.org/specs/fido-v2.3-ps-20260226/fido-client-to-authenticator-protocol-v2.3-ps-20260226.html
- [S7] Yubico: User Presence vs User Verification: https://developers.yubico.com/WebAuthn/WebAuthn_Developer_Guide/User_Presence_vs_User_Verification.html
- [S8] Apple Watch: pay by double-clicking the side button: https://support.apple.com/zh-sg/guide/watch/apdbe9c11bba
- [S9] Apple: adjusting the side button / Home button and accessible alternatives: https://support.apple.com/guide/iphone/adjust-the-side-or-home-button-iph75f461ff0/ios
- [S10] Ledger security model (secure screen wired directly to the SE; WYSIWYS): https://www.ledger.com/academy
- [S11] Trezor: item-by-item review before transaction signing: https://trezor.io
- [S12] Matter pairing flow: https://support.apple.com/zh-cn/102135 ; https://developers.home.google.com/matter/integration/pair
- [S13] RFC 3161 (Time-Stamp Protocol): https://www.rfc-editor.org/info/rfc3161
- [S14] RFC 9162 (Certificate Transparency): https://www.rfc-editor.org/info/rfc9162
- [S15] RFC 8785 (JSON Canonicalization Scheme): https://www.rfc-editor.org/info/rfc8785
- [S16] UniTrust Time Stamp Authority (联合信任时间戳服务中心): https://www.tsa.cn
- [S17] Nordic: Legacy Pairing vs LE Secure Connections (Just Works, TK=0): https://academy.nordicsemi.com/courses/bluetooth-low-energy-fundamentals/lessons/lesson-5-bluetooth-le-security-fundamentals/topic/legacy-pairing-vs-le-secure-connections
- [S18] TI: Numeric Comparison and MITM (Bluetooth 5.2 Table 2.8): https://dev.ti.com/tirex/explore/node?node=A__ANDzawDtfR7399ClR5bqCA__SIMPLELINK-ACADEMY-CC23XX__gsUPh6j__LATEST
- [S19] Wired: Tesla counters Bluetooth key relay attacks with UWB: https://www.wired.com/story/tesla-ultrawide-ble-keyless-entry-security-vulnerabilities
- [S20] Empirical evidence of voiceprint cloning (5-second sample): Hubei Daily (湖北日报) 2026-03-27; Xinhua News Agency (新华社) 2026-07-22 (app.xinhuanet.com)
- [S21] Timeline of the 2022 Uber compromise via MFA fatigue: https://phoenix.security/uber-hack-timeline
- [S22] The Akira ransomware group weaponizes MFA fatigue (2025-11): https://securityboulevard.com/2025/11/the-akira-playbook-how-ransomware-groups-are-weaponizing-mfa-fatigue
- [S23] Microsoft Security Blog (tiered recommendation for number matching): https://techcommunity.microsoft.com/blog/microsoft-security-blog/strengthening-identity-protection-in-the-face-of-highly-sophisticated-attacks/4006009
- [S24] Flowconsent (72% ignore cookie banners, 2026-04): https://www.flowconsent.com/de/blog/consent-fatigue-visitors-ignore-cookie-banner
- [S25] CookieYes (the behavior of 2/3 of users does not change with banner design): https://www.cookieyes.com/blog/cookie-consent-psychology
- [S26] UXMag (average banner dwell time of about 7 seconds, citing a 2022 study): https://uxmag.medium.com/consent-fatigue-are-we-designing-people-into-compliance-a04928cfba9f
- [S27] *Electronic Signature Law* (《电子签名法》), Articles 13, 14, and 16: http://www.nxlw.gov.cn/zwgk/zfxxgkml/flfg/201912/t20191209_1875963.html
- [S28] *E-Commerce Law* (《电子商务法》), Articles 48 and 31: full text on the Ministry of Commerce (商务部) website (mofcom.gov.cn)
- [S29] *Civil Code* (《民法典》), Article 491, article interpretation: faxin.cn
- [S30] *Provisions of the Supreme People's Court on Evidence in Civil Proceedings* (《最高人民法院民事证据规定》), Articles 93 and 94 (2019 amendment): ipc.court.gov.cn
- [S31] *Online Litigation Rules of the People's Courts* (《人民法院在线诉讼规则》), Articles 16–19 (2021, 法发〔2021〕12号 / Fa Fa [2021] No. 12): full text in the Supreme People's Court Gazette (最高人民法院公报) http://gongbao.court.gov.cn/Details/ac5f36e345967c22e0a2ac4fbeb0a6.html (the original wording of Article 16, "presumed authentic once on-chain" (上链后推定真实), has been verified)
- [S32] *Personal Information Protection Law* (《个人信息保护法》), Articles 4 and 47: full text on the official website of the Cyberspace Administration of China (中央网信办) http://www.cac.gov.cn/2021-08/20/c_1631050028355286.htm (the original wording of Article 4(1), "does not include information after anonymization processing" (不包括匿名化处理后的信息), has been verified)
- [S33] *Cybersecurity Law* (《网络安全法》), Article 21 (log retention of not less than six months): full text on the official website of the Cyberspace Administration of China (中央网信办) (paginated version) https://www.cac.gov.cn/2016-11/07/c_1119867116.htm (the original wording of Article 21, "not less than six months" (不少于六个月), has been verified)
- [S34] Supreme People's Court (最高人民法院), *Provisions on Civil Disputes over Bank Cards* (《银行卡民事纠纷规定》) (burden of proof): https://www.court.gov.cn/hudong/xiangqing/100362.html
- [S35] The end of "refund-only" ("仅退款") (Taobao / Douyin both abolished it on 2025-04-22): news.cctv.com/2025/04/28; https://www.yicai.com/news/102583799; paper.ce.cn 2025-04-28 (Economic Daily (经济日报))
- [S36] UnionPay dual-exemption limits and full compensation: small-amount no-password no-signature page on the official China UnionPay (中国银联) website https://cn.unionpay.com/upowhtml/cn/templates/smallSecretFree/smallSecretFree.html; announcements by Bank of China (中行) / Postal Savings Bank of China (邮储)
- [S37] Central bank Class III accounts (Ⅲ 类账户) (2016 classification management); e-CNY power-free payment (数字人民币无电支付): announcements and reports from the People's Bank of China (中国人民银行)
- [S38] Liability determination and appeals on food-delivery platforms: CBNData 2024; *Basic Requirements for the Service Management of Food-Delivery Platforms* (《外卖平台服务管理基本要求》) (2025-12) — cited by the companion document *The Economics of Liability Determination (One Page)* (《判责经济学（一页）》)
- [S39] *Civil Code* (《民法典》), Article 188 (three-year limitation period for ordinary civil litigation): full text on the official website of the Ministry of Justice (司法部) (moj.gov.cn)
- [S40] *Civil Code* (《民法典》), Article 496 (duty to highlight and explain standard-form clauses; if breached, the other party may assert that the clause does not become part of the contract): cross-checked across multiple sources (annotated edition of the Civil Code (国民法典注释本); original text quoted by government legal-popularization sites (政府普法站点))
- [S41] Mandatory national standard *Basic Security Requirements for Agent Applications* (《智能体应用安全基本要求》) approved for development (plan number 20263116-Q-252; the world's first mandatory agent security standard): Xinhua Net (新华网) 2026-07-28 https://www.news.cn/tech/20260728/8ebf5083cf0e487287f894fb31e123f5/c.html
- [S42] Two association standards launched — *Technical Framework for Agent Identity Authentication and Authorization* (《智能体身份鉴别与授权技术框架》) and *Technical Security Requirements for Agent Runtime* (《智能体运行时安全技术要求》) (proposed by the Cyberspace Security Association of China (中国网络空间安全协会), drafted with the participation of Ant Group (蚂蚁集团) and others; 2026 Bund Summit, 2026-09-09): Beijing Daily app (北京日报客户端) 2026-09-10 https://xinwen.bjd.com.cn/content/s6aa26173e4b039a8e2f0e33c.html
- [S43] *Implementation Opinions on Standardized Application and Innovative Development of Agents* (《智能体规范应用与创新发展实施意见》) (requires establishing an agent standards system): Cyberspace Administration of China (网信办) 2026-05-08 https://www.cac.gov.cn/2026-05/08/c_1779979789523320.htm
- [S44] Report on the approval for development of a national standard for agent identity management (a blockchain-based agent identity management model covering the full chain from overall architecture to interface specifications): Sina Finance (新浪财经) 2026-09-10 https://finance.sina.com.cn/stock/t/2026-09-10/doc-inirkiyt2381937.shtml

- [S45] *Price Law* (《价格法》), Article 13 (duty to mark prices clearly): official full text via the flk.npc.gov.cn verification channel (cited in the v0.2 verification batch)
- [S46] *Consumer Protection Regulation* (《消费者权益保护法实施条例》), Article 50(2) (**the penalty for the obligation under Article 22: a fine of 1–10 times the illegal gains; where there are no illegal gains, a fine of not more than CNY 500,000; in serious cases, suspension for rectification / revocation of the business license — the article number was confirmed word by word against the official full-text page**) + the Guangzhou Internet Court (广州互联网法院) full-refund case on auto-renewal (Tang v. a certain network services company; in the 2025-08 judgment, CNY 239.2 was refunded; settled on appeal): full text of the regulations mee.gov.cn/zcwj/gwywj/202403/t20240320_1068830.shtml; People's Daily (人民日报) 2025-08-07 ("Explaining the Law through Cases" (以案说法)) paper.people.com.cn; republished by the Ministry of Justice (司法部) smart legal-popularization platform legalinfo.moj.gov.cn; Guangming Daily Digest (光明日报文摘报) 2025-08-20
- [S47] Supreme People's Court (最高人民法院) 2026-03-15 release of typical consumer rights protection cases, Case 1, "Xie v. a certain video services company, network service contract dispute" (谢某诉某视讯公司网络服务合同纠纷案) (case source: Shanghai Pudong New Area People's Court (上海市浦东新区人民法院); auto-renewal without prominent reminder, **the operator was ordered to bear compensation liability for interest losses during the period of fund occupation** — confirmed word by word against the original text on the official court.gov.cn website): reported the same day by court.gov.cn / Xinhua News Agency (新华社) / People's Daily Online (人民网)
- [S48] *E-Commerce Law* (《电子商务法》), Article 17 (comprehensive, truthful, accurate, and timely disclosure): State Taxation Administration (国家税务总局) regulations database fgk.chinatax.gov.cn; republished by the Cyberspace Administration of China (网信办) cac.gov.cn
- [S49] *Consumer Protection Law* (《消费者权益保护法》), Article 20 (truthful and comprehensive information), Article 21 (marking of true names), and Article 25 (seven-day no-reason return and its four-item exclusion list): Shenzhen Municipal Market Supervision Administration (深圳市监局) amr.sz.gov.cn; Beijing legal-popularization alliance (北京市普法联盟) bj148.org, among multiple sources
- [S50] *Consumer Protection Regulation* (《消费者权益保护法实施条例》), Article 19 (the scope of goods not eligible for no-reason return must not be expanded without authorization): full-text republication page of the Ministry of Ecology and Environment (生态环境部) mee.gov.cn; National Courts Rule-of-Law Database (全国法院法治数据库) lawdb.cncourt.org
- [S51] *Measures for the Supervision and Administration of Rules of Online Trading Platforms* (《网络交易平台规则监督管理办法》) (released 2025-12; Article 21, obligation to provide a dispute resolution mechanism for on-platform transactions): republished by the Ministry of Commerce (商务部) global regulations site policy.mofcom.gov.cn; effective date and full text pending final verification via flk
- [S52] Civil Aviation Administration of China (民航局), *Notice on Improving Civil Aviation Ticketing Services* (《关于改进民航票务服务工作的通知》) (局发明电〔2018〕1952号; tiered fee rates / refund fees not exceeding the actual sale price): CAAC official website caac.gov.cn; cited by Xinhua Net (新华网) on 2025-04-14 as confirming it remains in force
- [S53] China Air Transport Association (中国航协) dedicated association standard on ticket refunds and changes (under development; planned for release within 2026; mandatory pop-up disclosure before payment): People's Daily (人民日报) 2026-08-26 (republished across multiple sources: Xinhua News app (新华社客户端) / China.org.cn (中国网) / 21st Century Business Herald (21世纪经济报道)); China Eastern Airlines (东航) new *Implementation Rules for Voluntary Refunds and Voluntary Changes of Domestic Tickets* (《国内客票自愿退票和自愿变更实施细则》) (applicable from 2026-08-06) is an early airline instance
- [S54] *Provisions on the Administration of Passenger Services in Public Air Transport* (《公共航空运输旅客服务管理规定》) (交通运输部令 2021 年第 3 号 / Ministry of Transport Order No. 3 of 2021), Article 23 (voluntary refunds and changes are handled in accordance with the general conditions of carriage): full text on the official website of the Ministry of Justice (司法部) moj.gov.cn
- [S55] *Consumer Protection Regulation* (《消费者权益保护法实施条例》), Article 10 (duty to prominently remind of automatic extension / automatic renewal; 国务院令第 778 号 / State Council Order No. 778, effective 2024-07-01): full-text republication page of the Ministry of Ecology and Environment (生态环境部) mee.gov.cn; Ministry of Justice (司法部) moj.gov.cn; National Courts Rule-of-Law Database (全国法院法治数据库) lawdb.cncourt.org — official multi-source verbatim agreement (2026-09-20 verification batch)
- [S56] *Measures for the Supervision and Administration of Online Transactions* (《网络交易监督管理办法》), Article 18 (**amended on 2025-03-18 by 国家市场监督管理总局令第 101 号 / SAMR Order No. 101; the amendments took effect on 2025-05-01**; the original measures were promulgated on 2021-03-15 by 第 37 号令 / Order No. 37): the revised Article 18 contains three elements — "a prominent reminder five days before the date of, e.g., automatic renewal + a prominent and simple option to cancel or change at any time + no unreasonable fees may be charged" (shdf.gov.cn / shanghaiinvest.com current-version snapshots agree across two sources; verbatim full text of the revised version pending final verification via samr.gov.cn); official website of the Ministry of Justice (司法部) moj.gov.cn (2021 original version)

---

*— End of Specification —*

This specification text and the reference implementation are released under the Apache License 2.0 (https://www.apache.org/licenses/LICENSE-2.0). Protocol designer: Su Yawei (苏亚伟). Comments and technical revision proposals are welcome as Issues / Pull Requests; when citing this specification, state the version number (v0.2.1, cross-reference errata dated 2026-09-25; mechanism semantics unchanged from v0.2).
