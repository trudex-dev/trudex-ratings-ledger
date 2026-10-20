# Trudex Ratings Ledger

The public, append-only ledger for [Trudex Ratings](https://trudex.dev). Each UTC day, one file `YYYY-MM-DD.json` holds the `payload_hash` of every forecast submitted that day, as the leaves of a Merkle tree, and the tree's root. Each root is stamped with [OpenTimestamps](https://opentimestamps.org).

A forecaster can prove that a forecast existed before its question resolved, and was never changed, by checking that its `payload_hash` is a leaf of that day's tree and that the root's OpenTimestamps proof is valid.

Files here are never edited or deleted. Disputes about a resolution are filed as issues on this repo within 72 hours of the resolution.

## Format (version 1)

**Payload hash.** Each submission's `payload_hash` is the SHA-256 of the UTF-8 bytes of five lines joined by `\n` (no trailing newline):

```
trudex-ratings/submission/v1
{forecaster_id}      lowercase hyphenated UUID
{question_id}        lowercase hyphenated UUID
{p}                  the stored probability, exactly 4 decimals, e.g. 0.7000
{submitted_at}       server receipt time, UTC ISO 8601 with milliseconds, e.g. 2026-10-21T14:03:07.123Z
```

The submission API returns `payload_hash`, `p` and `submitted_at` on receipt, so you can recompute the hash yourself. Example: forecaster `00000000-0000-0000-0000-0000000000f1`, question `00000000-0000-0000-0000-0000000000a1`, p `0.7000`, time `2026-10-21T14:03:07.123Z` hash to `ad5fa9174d8cd5e079d67874ad984e85bb28905731990899ced39c3b90bf9e89`.

**Daily file.** `YYYY-MM-DD.json` covers submissions received in that UTC day:

```json
{
  "version": 1,
  "day": "2026-10-21",
  "payload_encoding": "trudex-ratings/submission/v1",
  "tree": "rfc6962-sha256, leaves sorted by byte value",
  "leaf_count": 2,
  "root": "<hex>",
  "leaves": ["<payload_hash hex>", "<payload_hash hex>"]
}
```

**Tree.** The [RFC 6962](https://www.rfc-editor.org/rfc/rfc6962#section-2.1) Merkle tree (as used by Certificate Transparency) over the day's payload hashes sorted by byte value: a leaf is `SHA-256(0x00 ‖ payload_hash)`, a node is `SHA-256(0x01 ‖ left ‖ right)`, and the split is at the largest power of two below the leaf count. A day with no submissions is still published; its root is `SHA-256("")` = `e3b0c442…b855`. A missing day means the job did not run, never that submissions were left out.

**Timestamp.** `YYYY-MM-DD.json.ots` is an [OpenTimestamps](https://opentimestamps.org) proof for the day's file. It starts as a calendar commitment and is upgraded to a Bitcoin attestation within a day or so. Check it with `ots verify YYYY-MM-DD.json.ots`.

## Verify a forecast

1. Recompute your `payload_hash` from the fields above.
2. Find it in `leaves` of the day you submitted.
3. Rebuild the root from `leaves` and compare it with `root`, or ask the Trudex Ratings MCP tool `verify_submission` for an inclusion proof.
4. Run `ots verify` on that day's `.ots` file.
