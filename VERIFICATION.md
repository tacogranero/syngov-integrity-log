# Independent verification procedure

This is the complete procedure to verify a SynGov artifact against this
repository's published integrity roots, **using only standard tools and
public services — no SynGov server involved at any step.**

You will need: `openssl` (or any RFC 3161-capable timestamp verifier),
a `sha256sum`/`shasum` tool, a web browser, and the artifact's
`artifact_hash` (visible at `https://syntropygovernance.com/verify/sig/<hash>`,
or supplied to you directly with the artifact).

## Step 1 — Confirm the artifact's hash is in a published root

1. Find the day the artifact was issued (visible at `/verify/sig/:hash`,
   field `platform.signed_at` or `ceo.signed_at`).
2. Open `roots/YYYY-MM-DD.json` for that day in this repository.
3. Confirm your `artifact_hash` appears, verbatim, in the
   `artifact_hashes` array.

If it is not there: either the artifact was issued on a different day
than expected, or something is wrong — stop here and do not proceed to
treat the root below as covering this artifact.

## Step 2 — Recompute the Merkle root yourself

Do not trust the `merkle_root` field printed in the file — recompute it
independently from `artifact_hashes`, using this exact algorithm
(`merkle_algorithm: "sha256-binary-tree-v1"`):

1. Take the `artifact_hashes` array. Sort the hex strings
   lexicographically (ascending), and remove exact duplicates.
2. If the list is empty, there is no root for that day (this file
   would not exist — every published file has at least 1 hash).
3. If there is exactly 1 hash after step 1, the Merkle root **is**
   that hash, unchanged.
4. Otherwise, decode each hex string to its 32 raw bytes. Build the
   tree bottom-up:
   - At each level, process the current list of node-hashes in pairs,
     left to right.
   - If the level has an odd number of nodes, duplicate the last node
     (pair it with itself) for that level only.
   - Each pair's parent = `SHA-256(left_bytes || right_bytes)` (raw
     byte concatenation, left node's 32 bytes followed by right node's
     32 bytes, then hashed — not a string concatenation of hex text).
   - Repeat until exactly one node remains: that is the Merkle root
     (hex-encode it for comparison).

A short Python reference (for cross-checking your own implementation,
not required to trust ours — write your own from the algorithm
description above if you prefer):

```python
import hashlib

def merkle_root(hex_hashes):
    hashes = sorted(set(hex_hashes))
    if len(hashes) == 0:
        return None
    nodes = [bytes.fromhex(h) for h in hashes]
    if len(nodes) == 1:
        return nodes[0].hex()
    while len(nodes) > 1:
        if len(nodes) % 2 == 1:
            nodes.append(nodes[-1])
        nodes = [hashlib.sha256(nodes[i] + nodes[i+1]).digest() for i in range(0, len(nodes), 2)]
    return nodes[0].hex()
```

Confirm your recomputed root equals the file's `merkle_root` field
exactly. If it does not, stop — do not treat the RFC 3161 seal below as
covering this set of hashes.

## Step 3 — Verify the RFC 3161 seal on the root

The file's `rfc3161.token_base64` field is a base64-encoded RFC 3161
TimeStampToken, sealing the Merkle root from Step 2 (the raw SHA-256
digest of `merkle_root`, i.e. the 32 bytes you get by hex-decoding
`merkle_root` — the token seals the digest bytes directly, not a
re-hash of the hex string).

```bash
# Decode the token to a file
echo "<token_base64 value>" | base64 -d > root.tsr

# Verify it independently (openssl never contacts any SynGov server)
openssl ts -reply -in root.tsr -token_in -text
```

Confirm:
- `Status: Granted` (or `Granted with modifications`).
- The `Message data` hex value shown by openssl matches the raw bytes
  of `merkle_root` from Step 2.
- The `TSA` field identifies the time-stamping authority (`tsa_url`
  in the file names which authority was used) — this is an
  independent third party, never SynGov.
- The `Time stamp` field is the instant an independent authority
  attests this root existed.

## Step 4 — Verify external archival (Software Heritage)

The file's `software_heritage.snapshot_swhid` field identifies a
snapshot of this repository, archived by Software Heritage
(a nonprofit digital preservation initiative, INRIA-backed) at
publication time.

1. Open `https://archive.softwareheritage.org/<snapshot_swhid>` in a
   browser (substitute the actual SWHID value).
2. Confirm the archived snapshot contains the same `roots/YYYY-MM-DD.json`
   file, with the same content, as what you read in Step 1.

This step confirms that a copy of this publication exists in an
archive independent of both SynGov and GitHub — if either disappeared,
the Software Heritage archive is a second, separate place this root
can still be found.

## What you have confirmed, end to end

If all 4 steps pass: your artifact's hash was included in a Merkle
root, that exact root was sealed by an independent RFC 3161 authority
at a specific instant, and the publication containing it was archived
by a nonprofit preservation service independent of SynGov — with zero
SynGov involvement in any of the 4 checks above.

## What this does NOT confirm

- It does not confirm the artifact's *content* is accurate — that is
  the job of the Ed25519 signature chain and (for T3-T4, once live)
  the qualified co-signature, both verified separately at
  `/verify/sig/:hash`.
- It does not make the underlying data legally "immutable" in an
  absolute sense — it gives you two independent, externally-checkable
  attestations (a timestamp and an archive copy) that did not depend
  on SynGov's own infrastructure being available or trustworthy at the
  moment you check.
