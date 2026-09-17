# PhonoGrams research

Papers, notes, and algorithm specs behind the [PhonoGrams](https://github.com/PhonoGrams) Go libraries:

- [`soft_bigram`](https://github.com/PhonoGrams/soft_bigram) — Soft-Bidist (distance)
- [`soft-bisim`](https://github.com/PhonoGrams/soft-bisim) — Soft-Bisim (similarity)

These are character n-gram scorers for **personal-name matching**, aimed at [Watchman](https://github.com/moov-io/watchman) (OFAC / AML watchlist search). They are *not* phonetic encodings (Soundex, Double Metaphone) and *not* token-set metrics (Jaccard on words). They score two short strings as sequences of overlapping character pairs.

## Start here

| Doc | What it is |
|-----|------------|
| [docs/algorithms.md](docs/algorithms.md) | Recurrences, cost scales, what the old Go code got wrong |
| [docs/survey.md](docs/survey.md) | Digest of the paper collection and what still matters for Watchman |
| [docs/watchman.md](docs/watchman.md) | How to plug these into Watchman without regressing Jaro-Winkler |
| [docs/bibliography.md](docs/bibliography.md) | Full list of PDFs in `docs/papers/` |

## The two algorithms in one paragraph

Kondrak (SPIRE 2005) defined **N-DIST** and **N-SIM**: edit distance / LCS run over n-grams instead of characters, with the first letter repeated so prefixes count. **BI-DIST** / **BI-SIM** are the n=2 cases. The FDA's POCA software used BI-SIM on look-alike drug names.

Soft-Bidist (Hadwan et al. 2021) keeps the BI-DIST DP and replaces the positional 0/0.5/1 substitution with a nine-case scale (exact, transpose, first-only, second-only, …). Soft-Bisim (Millán-Hernández et al. 2019) does the same to BI-SIM. Both papers tune the nine weights statistically; the Go packages ship the published best weights, with Soft-Bisim's exact-match weight raised to 1 so identical strings score 1.0.

## Name

**PhonoGrams** is a fine org name. A phonogram is a written sign for a sound; these libraries score n-grams of those signs. It is distinctive on GitHub, and it signals "phonetic-ish + n-gram" without claiming to be Soundex.

Caveats: it collides in search with *phonopy* (physics), *PhenoGram* (genomics), and reading-education "phonograms". The previous org description had a typo (`soft-nisim`). Repo names mix `soft_bigram` (underscore) and `soft-bisim` (hyphen); leaving them avoids breaking import paths.
