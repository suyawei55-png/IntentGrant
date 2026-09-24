# Disclosure Boundary (DISCLOSURE-BOUNDARY)

> Repository-level governance document, companion to [IP-STATEMENT.md](IP-STATEMENT.md).
> Status: in force since 2026-09-24. Maintained by the protocol maintainers.

## Purpose

This repository is open by default: specifications, pseudocode-level reference
material, and governance documents are published under Apache-2.0. A small set of
assets is deliberately kept out of the repository — not as secret sauce, but
because they are either not part of the protocol, or are not ours to publish.
This document makes that boundary explicit and auditable, so that reviewers,
implementers, and partners know exactly where openness ends and why.

## 1. The three disclosure boundaries

### D1 — Protocol semantics are open; engineering implementations are not.

Everything needed to interoperate with and audit the protocol is published:
field definitions, timing windows, state machines, and liability-chain rules.
For example, the IG-Lite challenge semantics (§3.3) are fully open — fields,
the 60-second window, the quiet period, and failure branches.

Not published: entropy sources, hash-canonicalization engineering, deduplication
window parameters, deployment tuning. These carry no interoperability value and
are implementation details.

### D2 — Hardware trust root: graded semantics are open; hardware designs are not.

Device capability classes (DCC), capability flags, and what each class can
support are part of the specifications. Security-element product selection,
secure-boot/attestation engineering, and key-injection procedures belong to the
hardware team and are not published. The specifications themselves mark such
layers as *design intent*, never as implemented-capability claims.

### D3 — The reference implementation exists as an online black box, and that is deliberate.

The runnable reference implementation is provided as an independent verification
service (yishan.chat/demo): every credential it issues can be re-verified with
any Ed25519 library and RFC 8785 canonicalization. Its source code — including
sanitized versions — is not distributed. The verifiability of the protocol does
not depend on reading our code.

## 2. Permanently out of this repository (regardless of sanitization)

- Reference-implementation source code (sanitized or not)
- Hardware trust-root designs
- Litigation/evidence packages and business-development correspondence
- Internal business plans and briefing drafts
- Product roadmaps and customer information of the originating company
- Production deployment and operations parameters

## 3. Disclosure FAQ — what you can verify instead

- **"Can we get the code, or a sanitized version?"**
  The full specification and the pseudocode reference are open (Apache-2.0).
  The reference implementation runs as an online black box with independent
  signature verification — every artifact it produces can be re-verified with
  any Ed25519 library. The implementation layer is not distributed, sanitized
  copies included.
- **"How is the challenge implemented?"**
  Protocol semantics are fully open in IG-Lite §3.3: fields, 60-second window,
  quiet period, failure branches. Engineering parameters are implementation
  details and are not published.
- **"How is the secure element / trust root done?"**
  Graded device-capability classes (DCC) are published protocol content.
  Production hardware uses certified secure elements; design details belong to
  the hardware team.

## 4. Commit checklist (applied to every push)

1. Does the diff contain anything from the permanently-out list (§2)?
2. Engineering details of nonce entropy, hash canonicalization, or dedup windows? (D1)
3. SE/TEE/secure-boot/key-injection/attestation implementation details? (D2)
4. Real customer names, BD terms, or litigation-package text?
5. Staging discipline: files enter via an explicit whitelist, never by bulk directory sync.
6. Commit messages describe reality — no manufactured history.

## 5. Patent position (2026-09-24)

The maintainers have deliberately prioritized open publication of this
repository. Nothing in this repository constitutes a patent commitment, a
patent threat, or any patent grant beyond Apache-2.0 §3. See
[IP-STATEMENT.md](IP-STATEMENT.md) for the full licensing statement.

## Version history

- **2026-09-24 (v1.1, public)**: first in-repository version. Simplified from the
  internal working draft; internal execution notes are not part of the public record.
- **2026-09-23 (v1.0)**: initial boundary definition (internal working draft).
