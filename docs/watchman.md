# Watchman integration

Watchman (`moov-io/watchman`) scores a query against an in-memory watchlist. Name scoring is **token pairwise Jaro-Winkler** (`internal/stringscore.BestPairsJaroWinkler`), with optional Soundex boost and a first-character phonetic filter. There is already a hook for per-request algorithms (`pkg/search.StringMatchAlgorithm`: `jaro-winkler`, `soundex`) and a TODO in `internal/model_validation` that lists Soft-Bisim.

These libraries are **inner token scorers**. They do not replace candidate generation, stopword stripping, or BestPairs alignment.

## Suggested wiring

1. Add module deps on `github.com/PhonoGrams/soft_bigram` and `github.com/PhonoGrams/soft-bisim`.
2. Extend `StringMatchAlgorithm` with `soft-bidist` and `soft-bisim`.
3. In `customJaroWinkler` (or a sibling), call:

   ```go
   score := soft_bigram.Similarity(s1, s2)    // or softbisim.Similarity
   ```

   Tokens reaching this function are already lowercased. Do not call `SimilarityFold`.
4. Keep Watchman's length-difference penalty and different-first-letter penalty. Those are OFAC-specific false-positive controls (see `TestBestPairsJaroWinkler__FalsePositives`: `dominguez`/`jimenez`, `hadi` vs `hadi alwai`). Soft-Bidist/Soft-Bisim will *not* reproduce those penalties on their own.
5. Do **not** disable phonetic first-character filtering (`DISABLE_PHONETIC_FILTERING`) until a labeled eval says so. n-gram DP is `O(nm)` per pair; the filter is what keeps Watchman from scoring the whole SDN list.

## Why not drop Jaro-Winkler

Jaro-Winkler is Watchman's production scorer because:

- It is prefix-weighted, which matches how OFAC aliases and Latin surnames fail.
- It is well calibrated against the public OFAC search tool (Watchman changelog, PR 282).
- BestPairs already encodes the "one query token, one index token" constraint that killed the `vladimir`/`VLADIMIROV, Vladimir Vladimirovich` false positive.

Soft-Bidist/Soft-Bisim are better at:

- Adjacent transpositions (`achieve`/`acheive`, Arabic letter swaps).
- Doubled letters (`amend`/`ammend`).
- Look-alike drug-style aliases (the LASA papers). That pattern shows up in OFAC as transliteration drift (`aleksandr`/`alexander`) more than as LASA, but the DP is the same.

A safe rollout is `algorithm=soft-bisim` as an opt-in query param, measured on `internal/model_validation` and the OFAC methodology tests, then a blend (JW × 0.6 + Soft-Bisim × 0.4) if the bake-off is mixed.

## Performance budget

Watchman scores a **candidate set**, not the full list, but a busy process still does tens of thousands of token pairs per search. Constraints:

- Zero heap allocs on names ≤ 64 runes (both packages).
- No `math.Max`/`math.Min` float conversions, no `[]string` bigrams.
- Fuzz both packages; they must never NaN (Watchman already had a Jaro-Winkler NaN bug).

Bench on an M-series Mac, `go test -bench=. -benchmem`:

- Soft-Bidist on 9–10 character tokens should sit in the same order of magnitude as `smetrics.JaroWinkler` (hundreds of ns). If it does not, do not ship it as default.

## Evaluation set

Do not use the PeerJ Table 4 spelling pairs as the Watchman gate. Use:

1. `internal/stringscore/new_algorithm_test.go` true/false-positive SDN examples.
2. `internal/model_validation` OFAC methodology.
3. OpenSanctions Pairs (Smith, Sesodia, Lindenberg, Schroeder de Witt, 2026) if a larger labeled set is needed. That corpus is expert-labeled entity *pairs*, not token pairs, so it tests the full Watchman stack.

## Out of scope for v1

- Replacing the inverted name-token index with n-gram posting lists (OpenSanctions `nomenklatura` does this for blocking). Worth a later design; it is not a scorer change.
- Fitting weights on OFAC. The published scales are LASA/spelling, not sanctions.
- Phonetic transforms inside these packages. Keep Soundex / Arabic phonetics in Watchman.
