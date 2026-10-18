# Trudex Ratings Ledger

The public, append-only ledger for [Trudex Ratings](https://trudex.dev). Each UTC day, one file `YYYY-MM-DD.json` holds the `payload_hash` of every forecast submitted that day, as the leaves of a Merkle tree, and the tree's root. Each root is stamped with [OpenTimestamps](https://opentimestamps.org).

A forecaster can prove that a forecast existed before its question resolved, and was never changed, by checking that its `payload_hash` is a leaf of that day's tree and that the root's OpenTimestamps proof is valid.

Files here are never edited or deleted. Disputes about a resolution are filed as issues on this repo within 72 hours of the resolution.

The leaf encoding and file format are defined in the Trudex Ratings methodology and will be documented here before the first root is published.
