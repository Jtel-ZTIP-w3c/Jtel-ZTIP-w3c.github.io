# DEPLOYMENT.md — the deployment doctrine

The companion to the conformance vectors.

The [conformance family](INTEROP.md) proves a second implementation can **interoperate** — the
wire is open, a stranger can rebuild the spine from the vectors with no vendor in the loop. That is
necessary, not sufficient. A system can pass every vector and still be **captured at runtime**: by
an opaque model deciding silently, by a backend that hoards the data, by a per-message round-trip to
a hyperscaler, or by a single jurisdiction's trust root. Those are *deployment* properties, not
wire properties — and they are exactly where openness is usually lost.

This doctrine is the other half. The vectors answer *"can a stranger interoperate?"* The doctrine
answers *"is what is actually running still sovereign?"* Each rule below is a deployment-time
**check**, in the same spirit as the vectors: not a promise, an observable. Together they are what
"open all the way down" means — open beyond Krechmer, into the algorithmic age.

> These are deployment **defaults**, not protocol mandates. A deployment may diverge — but it must
> say where it diverges, and the checks make divergence visible. Silence is the only violation.

---

## The rules

**1. Identity is the key — never the name, the IP, or the transport.**
Admit on the resolved JIS Ed25519 actor key. The `.aint` label, the carrier address, and the
network path are all free and spoofable; trust binds only to the proven key.
*Check:* rotate the carrier IP or rename the `.aint` → access is unchanged. Strip the key proof →
access is gone.

**2. Identity ≠ authority. Audit is a precondition, not an afterthought.**
No action executes without a valid causal step (a TIBET-anchored token) proving it follows from the
intended one. The provenance gates execution; it does not merely record it after the fact.
*Check:* replay a step out of causal order → it is refused **before** it runs, not logged after.

**3. Trust is fresh — never stored.**
Authority comes from a fresh, per-use attestation (RVP), not a parked session, score, or cookie.
Biometrics and assurance are point-of-use and never persisted.
*Check:* a stale assurance is refused; no inherited score or supercookie grants the next action.

**4. Run it local — don't round-trip the planet.**
Inference runs locally (OomLlama); actors coordinate locally (I-Poll over plain HTTP); there is no
per-message egress to a hyperscaler. When a lighter path exists, the lighter path is the path.
*Check:* pull the cloud → core functions still run, and no data left the device to a third party.

**5. Keep no jurisdiction in the loop.**
Software Ed25519, no jurisdiction CA, no TEE-vendor lock, no centralized trust root. Every function
has a sovereignty-neutral path that depends on no single jurisdiction's certification or keys.
*Check:* name the one jurisdiction whose blessing is required for any function. If you can, you fail.

**6. Record properties — never content.**
Telemetry and provenance attest *classes, hashes, and aggregates* ("keyboard-generated",
cadence-match category, an event count) — never what was typed, the clipboard, or raw data.
Data-minimization by construction, not by policy.
*Check:* grep the logs for content. Finding any is the failure.

**7. Reproducible by any actor.**
Every actor — human, AI, or device — can produce a UPIP bundle that regenerates its own result.
"Test the bundle," never "trust the operator." Reproducibility is a habit, not a vendor promise.
*Check:* a stranger runs the bundle and gets the same result, having read none of your code.

**8. Run on a brick.**
It must work on legacy and low-power hardware — an eight-year-old phone, a MIPS router — not only a
flagship or a datacenter. The cost of the device is the floor of who is allowed in; keep it low.
*Check:* it runs on the cheapest device in the building.

**9. Fail closed; null-route the silent.**
When trust cannot be established, deny, quarantine, or null-route — never fail open. The absence of
a positive proof is a refusal, not a default-allow.
*Check:* cut the trust signal → the gate denies. If it passes, it is not a gate.

**10. You are never locked in.**
Revocation (tombstone), recovery (passkey / QR), and migration are first-class paths, not
afterthoughts. The exit is part of the design.
*Check:* revoke → you are out, locally and online, with no one's permission needed but your own.

**11. Leave no trace — the run is airlocked and self-cleaning.**
The execution itself is ephemeral. It runs in an airlock and cleans itself up completely — the way
a phantom session self-destructs after it seals. What survives is only the **outcome**, *reproducible
on demand*, never a stored process or its content; even the outcome can be served from a throwaway
sandbox endpoint, read once, and gone. Proof becomes something you *regenerate*, not something you
*hoard*.

This is the **default for an autonomous actor** — an unsupervised agent must not let intermediate
artifacts pile up on disk; ephemeral-by-default is hygiene and safety at once. For a human doing
ordinary, supervised work it is optional, sometimes overkill. And "leave no trace" is not "keep
nothing": what you may keep is a **thin, curated record** — the distilled outcome plus a journal of
the sources used and the key points — not the heap of intermediate files. The heap is reproducible
on demand; the journal is the part a human actually wanted.
*Check:* after a run, go looking for the process, its inputs, its intermediate state. Finding the
heap — beyond a result and a journal you deliberately chose to keep — is the failure.

---

## Why a doctrine, next to vectors

Krechmer's openness asks whether the *standard* is open. The four conformance families answer the
artifact half of that question — open documents, open interface, open access (runnable, not a paper
clause). But the 2026 questions — algorithmic transparency, data governance, environmental cost,
sovereignty — are decided at **deployment**, not on the wire. A standard can be fully open and still
be deployed through an opaque model, an extractive data layer, a datacenter externality, or a
jurisdictional backdoor.

So the pair is the point:

- **Conformance vectors** — *can an independent implementation interoperate without the vendor?* (the open wire)
- **Deployment doctrine** — *does the running system stay sovereign — local, fresh, content-blind, jurisdiction-neutral, reproducible, and exitable?* (the open runtime)

Pass the vectors and you have interoperated. Hold the doctrine and you have stayed free. Open is
both, all the way down.
