# ZTIP cookbook — what happens, on both sides

**A read-it-yourself recipe for the Zero-Trust Identification Protocol demo (ID-Drop / "SpeakEasy").**
For W3C / VC-WG technical readers who want to verify *what the app actually does* at each step —
on both the offering and the verifying device — before any verbal walkthrough. No marketing;
honest about what is proven vs scaffolded.

> **ID-Drop** is the app you install; **SpeakEasy** is the handshake it demonstrates;
> **ZTIP** is the protocol underneath. Identity is `jis:`, not `did:` — no DID-method claim,
> an Ed25519 key under a `jis:` scheme, with VC-shape interop.

---

## 1. The two sides (the asymmetry)

| Mode | Who | NFC role | Does |
|---|---|---|---|
| **User** | the human (citizen's phone) | **HCE** (card emulation), reader OFF | *offers* an identity + its attested VINK set. The physical tap = consent. |
| **Terminal** | the verifier (vendor handheld, 18+ check, vending machine) | **reader** ON | *asks*: reads the offer and verifies it. |

Humans naturally **offer**; systems naturally **request**. Machine↔machine inverts this
(request-first). The demo shows the human/offer-first side.

---

## 2. The seven steps — primitive · crypto · transport · side · resolved · shared

Each row is a real, time-stamped line in the on-screen console. Nothing is simulated.

| # | step | primitive / crypto | transport | side | resolved | shared |
|---|------|--------------------|-----------|------|----------|--------|
| 1 | offer + VINK set | Ed25519 sign over `VinkCanon` | NFC HCE (ISO-DEP) | offerer | — | outer metadata + signed VINK booleans |
| 2 | read | SELECT AID `F0:IDDROP` | NFC reader | terminal | offer JSON | — |
| 3 | resolve | `GET /api/ains/resolve/{name}` | HTTPS/TLS | name server | `.aint` → pubkey + status | the `.aint` name only |
| 4 | key-match | `pubkey == sender_pubkey` | local | terminal | binds `.aint` ↔ key | — |
| 5 | six-rule | published `tibet-iddrop` native `.so` | local | terminal | offer-first verdict (n/6) | — |
| 6 | VINK verify | Ed25519 `verifyBase64` | local | terminal | set is authentic + unforged | — |
| 7 | verdict | — | screen | terminal | the human-readable result | the ticks only |

**The VINK set** is a list of anonymous booleans the `.aint` attests — e.g.
`age_over_18=1 | valid_document=1 | nationality_nld=1`. The offerer signs the canonical
form (sorted, `key=1/0`) with its Ed25519 key. The terminal verifies that signature against
the **same key it just matched via AINS** — so the ticks can't be forged or stripped in transit.

---

## 3. What crosses the tap — and what does NOT

Only **discovery metadata + the signed VINK booleans** cross:

```
offer_id · expires_at (60s TTL) · sender_pubkey (Ed25519)
claimed_aint · claim_class · semantic_type · entity_class (human|ai|iot) · display_name
vinks[] (key, label, granted, demo?) · vinks_sig (Ed25519 over VinkCanon)
```

What does **not** cross, ever: **name · document number · date of birth · facial image.**
The verifier's screen prints a `not received` line to make the absence visible. The eMRTD
chip read, the predicate derivation, and the binding all happen **on the holder's device**;
only the boolean answer is presented. Find your name in the output — you can't.

---

## 4. The verdict you read

```
── VERDICT · who/what: human · vandemeent.aint
   signature   VINK set signed & verified ✓ (Ed25519 against key-matched .aint)
   claim       ✅ 18 or older
   claim       ✅ valid passport
   claim       ✅ Dutch national
   attestation key MATCH · six-rule PASS (6/6) · fresh (54s TTL) · online AINS
   not received name · document number · date of birth · facial image
```

A failing key-match, an expired TTL, a minor age-gate (`snaft_gated`), or a bad signature
each downgrade the relevant line (`❌` / `🔒`) with the reason — never a silent pass.

---

## 5. The safety property you can verify

**An identity offer is never automatically binding.** It enters as a **T-1 candidate** and
becomes a bound **T0** truth *only* after explicit accept **and** successful validation. A real,
resolvable name that isn't yours stays a candidate. Resolve `storm.vandemeent.aint` (a supervised
minor): it's `snaft_gated` / offline-only until a parental grant is active — the age-gate is not
escapable by self-deletion.

**Validate is proportional (triage L0→L3).** An `age_over_18` check may auto-pass (L0); a
passport or eSIM-keypair binding escalates to an explicit human ceremony (L2/L3). Same gate,
proportional friction — that's the dial on "Heart-in-the-Loop".

---

## 6. One resolver, many actors

The same seven steps carry any actor; only the proof-of-trust vocabulary differs:

```
human  → fresh biometric + state credential   (continuity, proven point-of-use)
AI     → mandate chain + causal step           (NOT persistence — stateless between calls;
                                                 it re-derives its right to act, carrying nothing)
IoT    → unbroken substrate / behaviour pattern (no network-switch, no uptime gap, or duty-cycle)
```

This is why ZTIP is not an age-gate: it's how any actor proves "I'm trustworthy for *this*",
in its own terms, without a central registry of who-it-is.

---

## 7. Try it (requirements + quickstart)

- **Android 12+** (API 31), `arm64-v8a`. NFC + HCE for the tap (optional — the online path
  works without). Download the APK from the latest [Release](../../releases/latest).
- **Two phones:** A claims a `.aint` (optionally binds a **DEMO** passport) and sets role
  **User**; B sets role **Terminal**; **tap them**. B's console shows the full ceremony +
  verdict.
- **One phone:** type a `.aint`, hit **Verify online** — the same six-rule validator in-band.
- **DEMO credentials** are labelled `(DEMO)` end-to-end. The technique is identical with or
  without; the flag only marks a test fixture so you can try the flow without a real eMRTD.

---

## 8. Honest scope

- Signed predicates + selective disclosure **now**; zero-knowledge (BBS+, unlinkable) is the
  direction, not yet claimed.
- Full CSCA passive-authentication of the passport chip is **Phase 2**; the demo verifies the
  chip signature is present and reads the data groups.
- `jis:` is deliberately **not** a W3C DID method (per Will Abramson's advice — stay out of the
  DID-method maze). VC-shape interop, yes; DID-Core compliance, not claimed.
- These are **Internet-Drafts (work in progress)**: `draft-vandemeent-iddrop`,
  `-ains-discovery`, `-jis-identity`, `-tibet-causal-time`, `-tibet-tat`, `-continuity-envelope`.

*The app is the falsifiable artifact. Tap it.*
