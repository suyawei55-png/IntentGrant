# IG-Lite v0.2.1 Minimal Reference Implementation (Pseudocode)

> **Language authority.** This document is a non-normative English translation of
> the Chinese `IG-LITE-PSEUDOCODE.md`. Both pseudocode documents are informative
> relative to the normative Chinese `IG-LITE.md` specification. If this
> translation conflicts with the Chinese pseudocode, the Chinese source controls
> the intended pseudocode content; the inconsistency is a translation defect,
> not an alternative interpretation or a divergence between co-equal language
> editions. Both language editions reflect the 2026-09-25 v0.2.1 errata, with no
> change to mechanism semantics.

> **Citation convention.** The **§N** notation refers only to sections of the
> IG-Lite specification.

> Nature: **pseudocode-level** — not an executable implementation. All cryptographic primitives (JCS canonicalization, SHA-256, signatures, CSPRNG) are delegated to the platform cryptography library; this document embeds no algorithm details. No BLE stack, no key management, no real persistence.
> Relationship to the specification: **the Chinese `IG-LITE.md` specification is normative; both language editions of this pseudocode are informative**. Field names and flow steps cite sections of the *IG-Lite v0.2.1 Specification* section by section; all capability claims are governed by the Implementation Status Statement in specification §12.
> Purpose: to show community reviewers the minimal closed-loop semantics and field flow of disclosure → completeness validation → confirmation → credential → receipt → proof of delivery.
> Version: extended from *IG-Lite-v0.1 Reference Implementation Pseudocode* (the v0.1 file is left untouched, as the archived v0.1.1 open-source release artifact). **v0.2 adds three sections: §1.5 `required` completeness validation (including the fail branch), §8 Delivery Receipt (D1, including nonce), and §9 Transit Fidelity validation (D2)**; §1 is incrementally revised (`schema_ref`/`delivery_tier` fields).
> v0.2.1 is a cross-reference and naming erratum with no mechanism-semantic change: Core architecture layers use Layer 0–Layer 3 terminology, and the audit helpers are renamed from `persist_L3` / `mark_L3` to `persist_audit` / `mark_audit` so that L0–L3 remain exclusive to LoA.

## Scope Covered (Appendix B Minimal Closed Loop + v0.2 Extensions)

| Step | Specification anchor | Version |
| --- | --- | --- |
| 1. Disclosure Object construction | §3.1 | v0.1 (incrementally revised in v0.2) |
| 1.5. `required` completeness validation (including the fail branch) | §6.3 / §4.2 / §4.4 | **New in v0.2** |
| 2. Announcement (Presentation Commitment, mode b structured triple) | §3.2 / §10.1.1 | v0.1 |
| 3. Challenge issuance | §3.3 | v0.1 |
| 4. Double-click event capture | §10.1.1 | v0.1 |
| 5. Confirm Grant issuance | §4.2 | v0.1 |
| 6. Receipt alignment | §4.5 | v0.1 |
| 8. Delivery Receipt generation (D1, including nonce) | §4.7.1 / §5.6 | **New in v0.2** |
| 9. Transit Fidelity validation (D2) | §4.7.2 | **New in v0.2** |

**Not covered (honest boundaries)**: the activation-layer pairing and binding flow (Core v2.0 Chapter 4 + the §3.5 association model and `association_model` records), §6.2 Registry governance and the schema publication process (this document assumes the schema has already been obtained from the Registry and cached locally), the full Custodian submission protocol in §8, the full rate-limiting state machine in §10.2, cross-device routing (§5.4), and the TEE secure-clock implementation for DCC-B and above devices (the §4.7.1 secure time source path only demonstrates the call site) — these are left to the scope of subsequent reference implementations.

## 0. Preliminaries: Constants and Dependencies

```
CONSTANTS:                                        # Defaults; industry Profiles may override (locked by evidence_rules_version)
  CHALLENGE_TTL_SECONDS = 60                      # §3.3 challenge validity window
  DOUBLE_CLICK_INTERVAL_MS = 600                  # §10.1.1 upper bound on the double-click detection interval
  SILENT_PERIOD_MS      = announcement duration   # §10.1.1 announcement silent period (keypresses during the announcement MUST be ignored)
  LONG_PRESS_MS         = 800                     # §10.1.1 long-press lower bound (aligned with Core v2.0 §5.5)
  GRANT_TTL_SECONDS     = 300                     # §4.2 Confirm Grant expiry (aligned with Core High tier)
  CONFIRM_PER_SESSION_MAX = 5                     # §10.1.2 rate-limiting threshold
  RECEIPT_NONCE_BITS    = 128                     # §4.7.1 per-receipt nonce entropy lower bound (privacy red line)
  RENDER_DURATION_DEFAULT_MS = 8000               # §4.7.1 default for the claimed path; the secure time source path is measured by dual timestamps

DEPENDS-ON (platform cryptography library; not implemented in this document):
  JCS(obj) -> bytes                               # RFC 8785 JSON canonicalization [S15]
  SHA256(bytes) -> b64url                         # §3.1
  CSPRNG(n_bits) -> bytes                         # entropy source for do_id ≥128bit / receipt nonce ≥128bit (§3.1/§4.7.1)
  sign(subject_key, bytes) -> sig                 # the signing subject is determined by attested_by (§4.2)
  verify(pubkey_or_fingerprint, bytes, sig) -> bool
  secure_clock_now() -> timestamp | UNSUPPORTED   # §5.1 secure_clock capability bit; source of the D1 dual timestamps (secure time source path)

SCHEMA-DEPS (from the Registry, §6.2):
  load_schema(schema_ref) -> schema               # {required: {field: spec}, na_rules, version}
  # local cache + version locking; the Registry retrieval protocol is out of scope for this document
```

## 1. Disclosure Object Construction (§3.1, Incrementally Revised in v0.2)

```
function build_disclosure(action_intent, context_template, sensitive: bool):
    d = {
        "do_id":        "do_" + base32(CSPRNG(128)),      # ≥128bit CSPRNG; Custodians MUST NOT record it (§8)
        "schema_ref":   pick_schema(action_intent.type),  # New in v0.2: ig:disclosure:<type>@<version> (§3.1/§6.4)
                                                          # unregistered type falls back to "ig:disclosure:T-0@…" or is left empty (in which case §1.5 is skipped)
        "context_template": context_template.text,         # context template (§3.2)
        "rendered":     fill(context_template, action_intent),
        "scope":        action_intent.scope,               # e.g. "commerce.order.create"
        "counterparty": action_intent.counterparty,
        "amount":       action_intent.amount,              # {value, currency}
        "attrs":        action_intent.attrs,
        "risks":        context_template.risks,
        "sensitive":    sensitive,                         # participates in the hash computation object (§3.1)
        "loa_target":   action_intent.loa_target,          # anchor of the §5.3 prohibition of silent downgrade
        "delivery_tier": action_intent.delivery_tier,      # New in v0.2: D0/D1/D2 declaration, hashed to prevent downgrade rewriting (§3.1/§5.6)
        "confirm_window": 60,
        "issued_at":    now_iso8601(),
        "evidence_rules_version": current_rules_version()
    }
    d["disclosure_hash"] = b64url(SHA256(JCS(d minus the disclosure_hash field)))
    persist_audit(d)                                       # Accountability Chain segment 1: intent capture trail (§3.4)
    state = "drafted"                                      # §4.4 state machine
    return d
```

## 1.5 `required` Completeness Validation (§6.3 Enforcement Chain Link 1; New in v0.2)

```
function validate_required(d):                             # pre-issuance validation: §4.2 fail branch / §4.4 rejected_schema state
    if d["schema_ref"] is None:
        return PASS                                        # no schema_ref → skip (unregistered types outside the T-0 fallback, §11.2 retained limitation ①)

    schema = load_schema(d["schema_ref"])                  # locally cached version, locked for the whole transaction (§6.5 version arbitration)
    for field, spec in schema.required:
        v = d.get(field)
        if v is None or v == "" or is_blank(v):
            # explicit N/A declaration rule (§6.3): missing ≠ not applicable; three-state prohibition — silent omission / empty string / null are all non-compliant
            na = d.get("applicability", {}).get(field)
            if na is None or na["applicability"] != "not_applicable" or is_blank(na["reason"]):
                return FAIL(reason="required_missing:" + field)

        if v is dict and v["applicability"] == "not_applicable":
            if is_blank(v["reason"]):
                return FAIL(reason="na_reason_blank:" + field)   # N/A MUST carry a genuine legal-grounds reason (§6.3)
            # spot checks for N/A abuse fall to the Custodian's admission audit (§6.3); the issuance side performs structural validation only

    return PASS

# —— issuance chain attachment points (§4.2 / §4.4) ——
function on_issue_challenge(d):
    verdict = validate_required(d)
    if verdict == FAIL(reason):
        state = "rejected_schema"                          # new pre-state in §4.4
        persist_audit({"event": "rejected_schema",
                    "schema_ref": d["schema_ref"],
                    "reason": reason})                     # rejection trail into the audit layer (no disclosure plaintext — isomorphic to the §8 Custodian content floor)
        return ABORT                                       # refuse to generate a challenge: the transaction never reaches confirmation (§6.3 link 1)
    return issue_challenge(d)
    # division of labor (§6.3 link 2): the hash guarantees integrity, this function guarantees completeness — the two mechanisms MUST NOT impersonate each other
    # ecosystem position (§6.3 honest boundary): this validation guards against "unintentionally missing fields"; deliberate circumvention (bypassing the SDK) falls to validator-side arbitration and the legal side
```

## 2. Announcement — Presentation Commitment Mode b (§3.2 / §10.1.1)

```
function present(d):
    triple = render_structured_triple(                   # action × counterparty × amount (weak-surface announcement form)
        d["rendered"], d["counterparty"], d["amount"])
    t_broadcast = now()
    speak_or_display(triple)                              # TTS or on-screen display; rendering trail into the audit layer (§3.2) — D0 as of v0.2 (§3.3/§5.6)
    hold_at_least(PRESENTATION_MIN_MS)                    # minimum presentation duration (§3.2)
    state = "presented"
    return t_broadcast                                    # silent period and start of the T window (§10.1.3)
```

## 3. Challenge Issuance (§3.3)

```
function issue_challenge(d):
    c = {
        "nonce":           hex(CSPRNG(128)),
        "disclosure_hash": d["disclosure_hash"],          # binds the keypress to the disclosure (anti-unbinding attack)
        "issued_at":       now_iso8601(),
        "expires_at":      now_iso8601() + CHALLENGE_TTL_SECONDS
    }
    persist_audit(c)                                      # do_id MUST NOT be recorded (§8 Custodian content floor)
    state = "challenge_issued"
    return c
```

## 4. Double-Click Event Capture (§10.1.1)

```
on PAE_button_event(ev):
    if state != "challenge_issued":
        discard(ev); return                               # any keypress outside the window MUST be ignored (anti-accidental-touch)

    if now() < t_broadcast + SILENT_PERIOD_MS:
        discard(ev); return                               # silent period: keypresses during the announcement MUST be ignored

    if ev == SINGLE_CLICK:
        discard(ev); await_second_click(DOUBLE_CLICK_INTERVAL_MS)
        return                                            # a single click MUST NOT constitute a consent signal (§10.1.1)

    if ev == DOUBLE_CLICK and interval(ev) <= DOUBLE_CLICK_INTERVAL_MS:
        if now() > challenge["expires_at"]:
            state = "expired"; restart_from(step 2)       # expired: the whole window is invalid; disclose again
            return
        route_challenge_to_PAE(challenge)                 # DCC-B and above: the device signs the challenge (§3.5)
        state = "confirmed" (pending Grant issuance)

    if ev == LONG_PRESS(duration >= LONG_PRESS_MS):
        route(high_sensitivity_flow)                      # high-sensitivity action channel (§10.1.1)
```

## 5. Confirm Grant Issuance (§4.2)

```
function issue_grant(d, c, dcc_class):
    g = {
        "grant_id":   "urn:uuid:" + uuid4(),
        "do_id":      d["do_id"],
        "dv":         "1.0",
        "scope":      d["scope"],                         # follows the Disclosure's action unit; MUST NOT be expanded
        "schema_ref": d["schema_ref"],                    # New in v0.2: the version is locked with the disclosure; the validator recomputes completeness from it (§6.5)
        "delivery_receipt_ref": null,                     # New in v0.2: D1/D2 receipt linkage (§4.7); nullable at issuance time
        "dcc_class":  dcc_class,
        "challenge":  c,                                  # embedded as a whole object (including disclosure_hash)
        "evidence_rules_version": d["evidence_rules_version"],
        "session_ref": current_session_id(),              # used for rate-limit counting (§10.1.2)
        "issue_time": now_iso8601(),
        "expire_time": now_iso8601() + GRANT_TTL_SECONDS
    }
    # attested_by determination (§3.5 / Appendix B upgrade ladder):
    #   DCC-C → "app" (application self-attestation, LoA capped at L1.5)
    #   DCC-B → "device" (device-side key signature; L1.5 may be labeled device-attested; this document only demonstrates the call site
    #            and does not claim the hardware already provides it — the DCC-B path is §12 "design intent")
    g["attested_by"] = (dcc_class == "B" ? "device" : "app")

    g["signature"] = sign(key_of(g["attested_by"]),
                          JCS(g minus the signature field))      # the signature covers all fields except itself (§4.2)
    g["verify_key_fingerprint"] = fingerprint_of(key_of(g["attested_by"]))
    # the verification public key ships with the credential: a disputing party can verify the signature without calling back to a registry (§4.2)

    persist_audit(g)                                       # Accountability Chain segment 3: confirmation trail
    return g
```

## 6. Receipt Alignment (§4.5)

```
function on_action_executed(execution_record):
    r = {
        "grant_id":           g["grant_id"],
        "executed_action_hash": b64url(SHA256(JCS(execution_record.action_core))),
        "disclosure_hash":    d["disclosure_hash"],
        "executed_at":        now_iso8601(),
        "result":             execution_record.result
    }
    # alignment decision: action, subject, and amount/limit agree within the tolerance of the scope alignment rules (§4.5, not bare hash equality)
    if aligns(r["executed_action_hash"], d, rules):
        r["alignment"] = "ALIGNED"; state = "receipted"    # TOCTOU closed (Accountability Chain segment 4)
    else:
        r["alignment"] = "VIOLATED"; mark_audit("unauthorized execution")   # misalignment = the credential chain provides no authorization proof for it (§4.5/§11.1)
    persist_audit(r)
    submit_to_custodian([ d["disclosure_hash"], g["grant_id"], r["executed_action_hash"] ])
    # §8 custody submission: hashes and timestamps only; do_id and any plaintext element MUST NOT be included
    return r
```

## 7. Rate-Limiting and State Machine Skeleton (§4.4 / §10.1.2, Illustrative)

```
on session_start():
    confirm_count = 0

function confirm_frequency_guard(action):
    if action.is_meta_confirmation:         # meta-confirmation (e.g. Access Grant issuance confirmation)
        return ALLOW                        # does not consume the session quota (§10.1.2)
    confirm_count += 1
    if confirm_count > CONFIRM_PER_SESSION_MAX:
        force_full_disclosure_replay()      # above threshold: full-segment disclosure replay; the short-announcement channel is cut off
        return THROTTLED
    return ALLOW

on same_disclosure_re-request(hash, dt):
    if dt < 30s:   mark_audit("Agent violation: repeated confirmation request within 30 seconds")   # §10.1.2
    if dt < T:     skip_rebroadcast(); require_physical_confirm()  # §10.1.3 exemption semantics: no re-announcement, but the keypress is still required
    else:          restart_from(step 2)                      # beyond the T window: full replay
```

## 8. Delivery Receipt Generation (D1; §4.7.1; New in v0.2)

```
function generate_delivery_receipt(d, render_session):
    # invocation point: when weak-surface rendering completes (DCC-B and above endpoints; DCC-C has no device signature channel, so it falls back to the D0 rendering trail — §4.7.3)

    nonce = CSPRNG(RECEIPT_NONCE_BITS)                     # per-receipt nonce ≥128bit (privacy red line, §4.7.1)
                                                           # a raw hash of a low-entropy disclosure lets the transaction content be reconstructed by dictionary attack (§11.1) —
                                                           # the nonce goes into the credential; the Custodian MUST NOT record the nonce↔content mapping (§8)

    content_hash = b64url(SHA256(JCS(render_session.rendered_disclosure) || nonce))
                                                           # the nonce is mixed into the hash computation (§4.7.1 structure)

    # —— duration probative-weight tiers (§4.7.1, red-team E-2/P-2 revision) ——
    t_start = secure_clock_now()                           # secure time source path (§5.1 secure_clock capability bit)
    ... rendering ...
    t_end   = secure_clock_now()
    if t_start != UNSUPPORTED and t_end != UNSUPPORTED:
        duration_proof = {"render_start": t_start, "render_end": t_end}   # the dual timestamps go into the proof object
    else:
        duration_proof = {"render_duration_ms": render_session.duration}  # claimed field: participates in the hash, not in the proof
                                                           # "rendered for 1 second, signed as 8 seconds" — self-reported duration cannot serve as a proof object

    r1 = {
        "receipt_type":  "delivery",
        "receipt_id":    "urn:uuid:" + uuid4(),            # high-entropy receipt_id: one of the entropy premises of the custody hash (§8/C.8)
        "device_id_cert": device_cert_chain(),             # certificate chain for DCC-B and above
        "grant_ref":     (g exists ? g["grant_id"] : null),# nullable (§4.7.1): announce-then-confirm / browse-without-confirm → null
                                                           # null semantics = "disclosure delivered but confirmation did not occur" (carries no consent semantics)
        "content_hash":  content_hash,
        "schema_ref":    d["schema_ref"],
        "nonce":         b64url(nonce),                    # stored with the credential (§4.7.1 privacy red line)
        "render_start":  duration_proof.render_start,      # on the claimed path only start + duration_ms
        "render_duration_ms": duration_proof["render_duration_ms"] or duration(t_start, t_end),
        "signature":     sign(device_key, JCS(r1 minus the signature field))
    }
    persist_audit(r1)
    if g exists: g["delivery_receipt_ref"] = r1["receipt_id"]   # backfill linkage (for re-signing scenarios see the §9 chain)
    submit_to_custodian([ r1["content_hash"], r1["receipt_id"] ])   # receipt hash into custody (§8/C.8 adopted)
    return r1
```

## 9. Transit Fidelity Validation (D2; §4.7.2; New in v0.2)

```
function render_with_fidelity(source_payload, source_anchor):
    # chain premise: the rendering controller ≠ the disclosure generator (B-side relay / phone main-Agent scenario, §4.7.2)
    # source_anchor: the source platform's custody-trail reference (full custody path → trusted tier; timestamp-only path → audit reference only, §4.7.2 arbitration baseline)

    rendered = render(source_payload.disclosure)           # relay-side rendering
    nonce = CSPRNG(RECEIPT_NONCE_BITS)                     # D2 upholds the same nonce privacy red line (§4.7.2)

    r2 = {
        "receipt_type":         "transit_fidelity",
        "receipt_id":           "urn:uuid:" + uuid4(),
        "device_id_cert":       device_cert_chain(),
        "source_disclosure_hash": source_payload.source_hash,     # generated by the source platform
        "source_anchor_ref":    source_anchor.ref,
        "rendered_content_hash": b64url(SHA256(JCS(rendered) || nonce)),
        "render_agent_id":      current_agent_id(),           # KYA integration point
        "render_timestamp":     secure_clock_now() or now(),
        "fidelity_verdict":     null,                          # pending comparison
        "prior_receipt_id":     current_chain.head,            # re-rendering chain (§4.7.2) — null at the head of the chain
        "signature":            sign(device_key, JCS(r2 minus the signature field))
    }

    # —— comparison baseline (§4.7.2 arbitration baseline, red-team L-5 revision) ——
    baseline = custodian_lookup_latest_anchor_before(
                   source_payload.source_hash, r2["render_timestamp"])
                   # always use "the latest custody anchor prior to the rendering instant" as the baseline —
                   # this prevents the source platform from re-minting a new version hash after the fact and disguising a swap as a normal match

    r2["fidelity_verdict"] = (r2["rendered_content_hash"] == baseline ? "match" : "mismatch")
    persist_audit(r2)

    if r2["fidelity_verdict"] == "mismatch":
        mark_chain(r2["prior_receipt_id"], "VIOLATED")     # that chain's status becomes VIOLATED (§4.7.2)
        mark_audit("transit fidelity mismatch: the party responsible (relay or source platform) is to be determined")   # attribution: whichever party made the change must rebut with the other party's evidence (§4.7.2)
        require_new_disclosure_chain()                     # subsequent disclosures MUST open a new chain; partial field updates go through incremental re-rendering + full re-signing
        return r2                                          # a VIOLATED chain MUST NOT enter confirmation (a hard gate isomorphic to rejected_schema)

    if source_anchor.mode == "timestamp_only":
        mark_audit("receipt limited to audit reference (timestamp-only path; MUST NOT be asserted as a trusted-tier transit fidelity claim)")   # §4.7.2 tiering
    return r2
```

## Honest Boundaries (Consistent with §12)

- This document is pseudocode: `persist_audit` / `broadcast` / `route_*` / `custodian_lookup_*` are all placeholder calls with no real implementation.
- `attested_by: "device"` (the DCC-B path) and `secure_clock_now()` (secure time source) only demonstrate call sites — device-side signing and the TEE-protected-region clock are §12 "design intent"; no hardware currently provides them.
- The two sections added in v0.2, §8/§9, are at the **semantic design** level (specification §12 Implementation Status Statement: the D1/D2 credential structures = specification definition, engineering implementation not started) — this document shows the field flow and decision logic and does not constitute an "implemented" claim.
- During the pilot, Custodian submission runs in a downgraded single-log mode, and the downgraded state MUST be publicly declared (§8 independence clause); the entropy premise for receipt hashes entering custody (C.8): the computation input contains a high-entropy `receipt_id` and the per-receipt nonce (§8).
- **Semantic** spot checks for N/A abuse (claiming an exemption where non-applicability is plainly evident) are borne by the Custodian's admission audit — the issuance side performs structural validation only (missing / empty string / blank reason); the two roles MUST NOT be conflated (§6.3).
- Any implementer SHOULD read the specification's §9 evidence rules and §11.1 residual risk table before writing code — this document does not replace the specification.
