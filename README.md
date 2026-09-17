# PhonoGrams research

Papers, notes, and algorithm specs behind the [PhonoGrams](https://github.com/PhonoGrams) Go libraries for **personal-name matching**, aimed at [Watchman](https://github.com/moov-io/watchman) (OFAC / AML watchlist search):

- [`soft_bigram`](https://github.com/PhonoGrams/soft_bigram) — Soft-Bidist (distance)
- [`soft-bisim`](https://github.com/PhonoGrams/soft-bisim) — Soft-Bisim (similarity)
- [`editex`](https://github.com/PhonoGrams/editex) — Editex (Zobel & Dart 1996)
- [`ngram`](https://github.com/PhonoGrams/ngram) — Kondrak N-SIM / N-DIST (n=2, n=3)
- [`double_metaphone`](https://github.com/PhonoGrams/double_metaphone) — Double Metaphone (Philips 2000)
- [`beider_morse`](https://github.com/PhonoGrams/beider_morse) — generic Beider-Morse Phonetic Matching (v1)

The first four are character n-gram / edit-distance scorers (`Similarity` in `[0, 1]`). Double Metaphone and Beider-Morse are phonetic encodings used as Watchman boosts. They are not token-set metrics (Jaccard on words).

Watchman originally had no first-party Go impl of Editex, positional Kondrak N-SIM (including n=3), Double Metaphone, or Beider-Morse. Those four now live in this org at `v0.1.0` and are wired as `?algorithm=editex`, `nsim`, `nsim-3`, `double-metaphone`, and `beider-morse`.

## Start here

| Doc | What it is |
|-----|------------|
| [docs/algorithms.md](docs/algorithms.md) | Recurrences and cost scales |
| [docs/survey.md](docs/survey.md) | Digest of the paper collection and what still matters for Watchman |
| [docs/watchman.md](docs/watchman.md) | How to plug these into Watchman without regressing Jaro-Winkler |
| [docs/bibliography.md](docs/bibliography.md) | Full list of PDFs in `docs/papers/` |

## The two algorithms in one paragraph

Kondrak (SPIRE 2005) defined **N-DIST** and **N-SIM**: edit distance / LCS run over n-grams instead of characters, with the first letter repeated so prefixes count. **BI-DIST** / **BI-SIM** are the n=2 cases. The FDA's POCA software used BI-SIM on look-alike drug names.

Soft-Bidist (Hadwan et al. 2021) keeps the BI-DIST DP and replaces the positional 0/0.5/1 substitution with a nine-case scale (exact, transpose, first-only, second-only, …). Soft-Bisim (Millán-Hernández et al. 2019) does the same to BI-SIM. Both papers tune the nine weights statistically; the Go packages ship the published best weights, with Soft-Bisim's exact-match weight raised to 1 so identical strings score 1.0.
