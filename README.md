# ZTIP — Zero-Trust Identification Protocol

**Prove you're a trustworthy actor — without revealing who you are.**

Two Android phones. One **offers** an identity claim, the other **verifies** it. The verifier's
screen shows `✅ 18 or older`, `signature verified ✓`, and a line listing exactly what did **not**
cross the wire — name, document number, date of birth, photo. Every step prints live, with timing.

> **Where IAM remembers, ZTIP forgets.**
> IAM is a database that holds *who you are*; every check is a lookup it logs.
> ZTIP holds nothing — a short, signed attestation that a *fact* holds, verified point-of-use,
> then gone.

---

## What the verifier shows

```
── VERDICT · who/what: human · vandemeent.aint
   signature   VINK set signed & verified ✓ (Ed25519 against key-matched .aint)
   claim       ✅ 18 or older
   claim       ✅ valid passport
   claim       ✅ Dutch national
   attestation key MATCH · six-rule PASS (6/6) · fresh (54s TTL) · online AINS
   not received name · document number · date of birth · facial image
```

Find your name in that output. You can't — it never left the phone. That absence is the point.

---

## Verify it yourself (no app, just a terminal)

The name resolution is a real, public endpoint. Resolve any `.aint` and read the live record —
the Ed25519 public key the handshake key-matches against:

```sh
curl -s -H "User-Agent: ztip" https://api.ainternet.org/api/ains/resolve/root_idd | jq .record
# → { "public_key": "6aab3fb5…", "status": "active", "entity_type": "idd", … }
```

Claim your own `.aint` in the app, then resolve *that* from your terminal. Same key the phone shows.
Nothing here is staged: the resolve you watch in the app is the resolve you can run yourself.

---

## Tap it

```
Phone A  →  claim a .aint  →  (optional) bind a DEMO passport  →  role: User (offer)
Phone B  →  role: Terminal (verify)  →  tap A against B

No second phone?  →  type a .aint, hit "Verify online" — the same validator, in-band.
```

**APK:** latest [Release](../../releases/latest) · Android 12+ · `arm64-v8a` · NFC optional.
**Read along:** [docs/cookbook.md](./docs/cookbook.md) — every step, line by line.

---

## The seven steps

| # | step | primitive / crypto | transport | side |
|---|------|--------------------|-----------|------|
| 1 | offer + VINK set | Ed25519 sign over `VinkCanon` | NFC HCE | offerer |
| 2 | read | SELECT AID `F0:IDDROP` | NFC reader | terminal |
| 3 | resolve | AINS `/resolve` | HTTPS | name server |
| 4 | key-match | `pubkey ≟ resolved` | local | terminal |
| 5 | six-rule | `tibet-iddrop` native `.so` | local | terminal |
| 6 | VINK verify | Ed25519 verify | local | terminal |
| 7 | verdict | — | screen | terminal |

The VINK set is a list of signed booleans — `age_over_18=1 | valid_document=1 | nationality_nld=1`.
The offerer signs them; the verifier checks that signature against the key it just matched via AINS,
so the ticks can't be forged or stripped in transit.

---

## A rule, applied — not a person

The verifier asks a *rule*, never an identity:

```
verify   age ≥ 18 · context: online shopping · jurisdiction: NL
→ a 13-year-old's passport answers ❌ NOPE — the gate never sees the birth date, only "no".
```

Rules live with the verifier (the industry); identity lives with you. The attestation is the thin
bridge between them — verified on basic facts, sharing none of the characteristics that identify you
as a person.

---

## One resolver, many actors

The same seven steps carry any actor — only the proof-of-trust vocabulary differs:

```
human  → fresh biometric + state credential   (continuity)
AI     → mandate chain + causal step           (not persistence — stateless between calls,
                                                 it re-derives its right to act, carrying nothing)
IoT    → unbroken substrate / behaviour pattern
```

ZTIP isn't an age-gate. It's how *any* actor proves "I'm trustworthy for **this**" — in its own terms,
without a central registry of who-it-is. That's why it has to think about AI and devices, not only people.

---

## Honest notes

- Identity is `jis:`, **not** `did:` — no DID-method claim; the identifier is an Ed25519 key under a
  `jis:` scheme, VC-shape interop. The function DID aimed at, working today.
- **Signed predicates + selective disclosure** now; zero-knowledge (BBS+, unlinkable) is the direction,
  not yet claimed.
- **DEMO** credentials are labelled `(DEMO)` end-to-end — the technique is identical with or without;
  the flag only marks a test fixture, never fakes a real one.
- Full CSCA passive-authentication of the passport chip is Phase 2; the demo verifies the chip
  signature is present and reads the data groups.
- These are **Internet-Drafts (work in progress)** — `draft-vandemeent-iddrop`, `-ains-discovery`,
  `-jis-identity`, `-tibet-causal-time`, `-tibet-tat`, `-continuity-envelope`.

## Links

- SDK · [`tibet-iddrop`](https://crates.io/crates/tibet-iddrop) (the app runs this crate on-device)
- IETF · [`draft-vandemeent-iddrop`](https://datatracker.ietf.org/doc/draft-vandemeent-iddrop/)
- [humotica.com](https://humotica.com)

---

*computo et comprobo, ergo fui · attestor, ergo nunc sum*
*— I compute and verify, therefore I was; I attest, therefore now I am.*
