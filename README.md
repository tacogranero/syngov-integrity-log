# SynGov Integrity Log

This repository publishes periodic integrity roots for artifacts issued
by [SynGov](https://syntropygovernance.com) (Syntropy Governance,
S.L.). It is **not source code** — it is an external, append-only
publication log.

## What this is

Each file under `roots/` covers one calendar day (UTC) and lists the
`artifact_hash` of every T1-T4 committee-grade artifact issued by
SynGov that day (fully signed, `decision_signatures.ceo_signature`
present), together with:

- a Merkle root computed over those hashes (algorithm documented in
  [`VERIFICATION.md`](./VERIFICATION.md)),
- an RFC 3161 timestamp token anchoring that root to an external,
  independent time-stamping authority,
- the [Software Heritage](https://www.softwareheritage.org/) snapshot
  identifier (SWHID) returned when this repository was archived at
  publication time.

The intent: a third party who has one artifact's hash can confirm it
was part of a root published on a given day, and that the publication
itself was timestamped and archived externally — **without needing any
SynGov server to be reachable.** See `VERIFICATION.md` for the exact
steps.

## What this is NOT

- Not a claim of absolute, unconditional immutability. It is a
  **published integrity root, archived externally** — the specific,
  falsifiable guarantees are: (a) an independent time-stamping
  authority attests the root existed at a given instant, and (b)
  Software Heritage's own archive holds a copy of this repository's
  state at that time. Both of those are independently checkable by a
  third party; neither claim is stronger than that.
- Not a place to look up what an artifact's contents say. Only hashes
  are published here — never report content, client names, or any
  other evidence.
- Not maintained as a software project (no issues, no PRs expected in
  the ordinary course — publication happens via an automated,
  append-only process; see `.github/workflows/append-only-check.yml`
  for the mechanical rule enforced on every push).

## Repository rules

- **Append-only.** Every publication adds exactly one new file under
  `roots/`. No existing file is ever edited or deleted. Enforced by
  CI on every push (`.github/workflows/append-only-check.yml`) and by
  branch protection (force-push disabled on `main`).
- Filenames are `roots/YYYY-MM-DD.json`, one per UTC day with at least
  one artifact issued.

## Related

- [SynGov `/verify/sig/:hash`](https://syntropygovernance.com/verify/sig/) — per-artifact verification, including the RFC 3161 seal (Frente A1).
- [`VERIFICATION.md`](./VERIFICATION.md) — full independent verification procedure.
