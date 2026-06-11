# INTEROP.md — the primitive atlas

The map of the load-bearing **interop surface** — the small set of things a second
implementation must agree on to interoperate, with no vendor in the loop. Not the product
catalogue.

**What's in (the rule):** a thing belongs here only if it (1) defines a wire/crypto
**primitive** (its own IETF draft), (2) is a **sealed wire-format** (`.tza`), (3) is an
**enforcement primitive** (consumes trust signals, produces a verdict: allow/deny/quarantine/
null-route), or (4) is the **runtime substrate** the rest executes on (trust-kernel). One-line test: *does it define a
primitive, produce a trust verdict, or run the substrate? → in. Else → out.*

**What's out:** the apps (ID-Drop, KIT, cmail, Humotica) and the ~90 supporting runtime /
agentic / specialized packages — they *consume* this surface, they don't define it. Their
map is `stack-position-map.yml`. Keeping them out is what keeps this atlas readable.

Identifier scheme is **`jis:`**, never `did:` — Ed25519 keys under our own URI scheme, with
VC-shape interop but no DID-Core method claim. All ten drafts below are **active
Internet-Drafts** on the IETF datatracker (maintained by us; status checked 2026-06-11) —
several still at `-00` because the first revision cycle is waiting on external review, and
early changes came from pentest-driven fixes, not datatracker critique.

> **On IETF status.** The datatracker is openness of the tech, not a stamp of approval.
> Absence of critique there is **not** treated as validation. The drafts stay
> work-in-progress; revisions are driven by implementation, conformance vectors, pentest
> findings, and external review.

The shape on purpose: two roots, branches that grow from each, control/transfer primitives
above them, and composition protocols where the roots meet. ZTIP is **composition** — the
moment you say "attestation envelope", identity *and* provenance/order are both at the table.

```
                 L4  enforcement       SNAFT (semantic)  ·  NullRouteMux (routing)  ·  Cortex (consent)
                       │  → allow / deny / quarantine / null-route
                 L3  composition        ZTIP / VINK / offer / ceremony   ·   IDDrop (protocol)
                       │  composes JIS-sign + AINS-resolve + TIBET-proof/order
                 L2  control/transfer   RVP (continuous verify)   ·   TAT (transfer / re-attest)
                       │
  identity branch ───┐ │ ┌─── provenance / integrity branch
        AINS         │ │ │   continuity-envelope · causal-time · semantic-surface · tbz/.tza · UPIP
                 L0  JIS  ·  TIBET   ← root pair (substrate)
                 ▔▔  trust-kernel — Rust zero-trust runtime (the floor it all executes on)
```

---

## L0 · Root pair (substrate)

The two bedrock primitives. Everything composes on these.

| Primitive | Question it answers | Spec |
|---|---|---|
| **JIS** — identity | *who is allowed to speak?* — actor identity, Ed25519, FIR/A (fresh re-attestation), claim/bind | [draft-vandemeent-jis-identity](https://datatracker.ietf.org/doc/draft-vandemeent-jis-identity/) |
| **TIBET** — provenance | *what happened, signed, in order?* — intent/provenance token chain | [draft-vandemeent-tibet-provenance](https://datatracker.ietf.org/doc/draft-vandemeent-tibet-provenance/) |

*Runtime substrate:* the primitives execute on **trust-kernel** — the Rust zero-trust
runtime (snapshot-gate, airlock enforcement) that the higher layers run on and can
substitute. It is the floor, not a primitive of its own.

## L1 · Branch primitives

Each grows from one root.

| Primitive | Purpose | Depends on | Spec |
|---|---|---|---|
| **AINS** — discovery | name → key / actor / capability resolve, over JIS identities | JIS | [draft-vandemeent-ains-discovery](https://datatracker.ietf.org/doc/draft-vandemeent-ains-discovery/) |
| **Continuity envelope** | continuity object/envelope — the arrival/history carrier | TIBET | [draft-vandemeent-continuity-envelope](https://datatracker.ietf.org/doc/draft-vandemeent-continuity-envelope/) |
| **Causal Time** | happened-before / concurrency / causal ordering (CausalGuard) | TIBET (+ JIS lanes) | [draft-vandemeent-tibet-causal-time](https://datatracker.ietf.org/doc/draft-vandemeent-tibet-causal-time/) |
| **Semantic Surface Manifest** | sealed objects routable by semantic surface — without trusting name or content | TIBET | [draft-vandemeent-tibet-semantic-surface-manifest](https://datatracker.ietf.org/doc/draft-vandemeent-tibet-semantic-surface-manifest/) |
| **TBZ / .tza** — sealed carrier | sealed, content-addressed transfer container — what continuity envelopes, TAT transfers and file-drops ride in | TIBET (+ SSM routing) | vectors: [tibet-conformance-vectors](https://github.com/jaspertvdm/tibet-conformance-vectors) (tbz wire-format) |
| **UPIP** — process integrity | reproducible execution bundle (state · deps · process · result · verify) | TIBET evidence (may bind actors via JIS) | [draft-vandemeent-upip-process-integrity](https://datatracker.ietf.org/doc/draft-vandemeent-upip-process-integrity/) |

The integrity-branch backbone is provenance: UPIP's proof model is `state/deps/process/result/verify`. JIS can add actor-binding, but the spine is TIBET.

## L2 · Control & transfer primitives

Above the branches — they orchestrate rather than carry.

| Primitive | Role | Depends on | Spec |
|---|---|---|---|
| **RVP** — continuous verification | continuous trust/posture verification (untrusted-until-renewed) | control · JIS + TIBET evidence | [draft-vandemeent-rvp-continuous-verification](https://datatracker.ietf.org/doc/draft-vandemeent-rvp-continuous-verification/) |
| **TAT** — transfer / re-attestation | bridges an identity act + a provenance event; the transfer/envelope/proof of a re-attestation | composition · JIS + TIBET | [draft-vandemeent-tibet-tat](https://datatracker.ietf.org/doc/draft-vandemeent-tibet-tat/) |

RVP is neither an app nor an evidence object — it is a control primitive *above* the tree.
TAT is not pure JIS and not pure TIBET: re-attestation is an identity act, but the
transfer/envelope/proof is provenance-shaped.

## L3 · Composition protocols

Where the roots meet. **ZTIP is a composition primitive, not a JIS leaf.**

| Protocol | Purpose | Composes | Spec / vectors |
|---|---|---|---|
| **ZTIP** — VINK / offer / ceremony | anonymous attestation + exchange patterns | JIS-sign + AINS-resolve + TIBET-proof/order | vectors: [ztip-conformance](https://github.com/Jtel-ZTIP-w3c/ztip-conformance) |
| **IDDrop** (protocol) | proximity offer/accept composition | JIS + AINS + TIBET / TAT / continuity | [draft-vandemeent-iddrop](https://datatracker.ietf.org/doc/draft-vandemeent-iddrop/) |

`draft-vandemeent-iddrop` is the **protocol** anchor (interop). The **ID-Drop app** that
implements it is a product — see L6.

## L4 · Enforcement

The decision layer. SNAFT, NullRouteMux and Cortex are **not apps — they are enforcement
primitives**: they consume trust signals and produce verdicts. Their concrete packages live
in `stack-position-map.yml`; this atlas names the interop **role** and the **verdict shape**
(`verdict.v1`: allow / deny / quarantine / null-route — the same contract the airlock
runtime enforces).

| Enforcer | Role | Consumes | Produces |
|---|---|---|---|
| **SNAFT** | semantic policy enforcement (OWASP-style rules, intent firewall) | JIS · TIBET · AINS · SSM · RVP · TAT | allow / deny / quarantine |
| **NullRouteMux** | routing enforcement (silent null-route, the "0x00 of last resort") | JIS · SSM · RVP posture | route / null-route |
| **Cortex** | consent / permission gates (zero-trust knowledge & capability access) | JIS · RVP · policy | allow / deny |

Implementing packages: `snaft`/`tibet-snaft`, `tibet-mux`, `tibet-cortex` — what they *read*
is the primitives above.

## L5 · Conformance vectors (runnable proof)

Two repos, one per root branch. Run them; green = a stranger interoperates, no vendor needed.

| Repo | Proves |
|---|---|
| [ztip-conformance](https://github.com/Jtel-ZTIP-w3c/ztip-conformance) | ZTIP / VINK / offer composition — v1 VINK ✅ · v2 offer ✅ · v3 NFC 🔜 |
| [tibet-conformance-vectors](https://github.com/jaspertvdm/tibet-conformance-vectors) | TIBET / evidence primitives — continuity intake + tbz wire-format ✅ |

## L6 · Applications — explicitly out of scope

These **consume** the primitives above. They are products, not primitives, and are not
specified in this atlas:

- **ID-Drop** — the proximity attestation app (implements the IDDrop protocol, L3)
- **KIT** — the full identity wallet (passport-bind, Phoenix, liveness)
- **cmail** — signed-capsule messaging / approval surface
- the **Humotica** apps and the ~90 supporting packages (`stack-position-map.yml`)

---

## What this gives a developer

Everything above is built from **two roots: identity (JIS) and provenance (TIBET).** With
just those two you can already do real work — sign as an actor, prove what happened in
order. The layers above show *how far it reaches*: resolve names (AINS), order causality
(Causal Time), seal and route objects (SSM, `.tza`), prove a process ran honestly (UPIP),
re-verify continuously (RVP), transfer and re-attest (TAT), compose anonymous attestations
(ZTIP), and enforce a verdict (SNAFT/NullRouteMux/Cortex) — all on a Rust zero-trust runtime
(trust-kernel).

So a developer can program **shallow** (just sign and verify with JIS+TIBET) or **deep**
(steer enforcement, build a full trace/trail/evidence spine, re-attest per task) — and at
every depth it is the same spine. And the spine is **identity**: every layer is ultimately
answering *who may do this, is it still true, and can anyone prove it later — without a
vendor in the loop.*
