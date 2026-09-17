# Algorithm specs

This is the implementation spec. Package-level copies live in the Go repos; this file is the source of truth for how the two measures relate.

## Shared setup

Input is two Unicode strings, compared as rune slices. Watchman must pass **already-normalized tokens** (lowercase, punctuation stripped, ISO 4 / NFKD as Watchman already does). The libraries are case-sensitive on purpose: case folding is a prepare-pipeline job, not a scorer job.

Both algorithms run in `O(nm)` time and `O(min(n, m))` memory (two DP rows). Names up to 64 runes allocate nothing.

## Kondrak N-DIST / N-SIM (SPIRE 2005)

Affix by repeating the first character `n-1` times (bigrams: once). Then:

**Distance** (N-DIST):

```
D[i, 0] = i
D[0, j] = j
D[i, j] = min(
    D[i-1, j] + 1,
    D[i, j-1] + 1,
    D[i-1, j-1] + d_n(Γ_{i-1,j-1})
)
return D[n, m] / max(n, m)
```

`d_n` is the fraction of mismatched positions in the two n-grams. For bigrams that is `0`, `0.5`, or `1`.

**Similarity** (N-SIM):

```
S[i, 0] = S[0, j] = 0
S[i, j] = max(
    S[i-1, j],
    S[i, j-1],
    S[i-1, j-1] + s_n(Γ_{i-1,j-1})
)
return S[n, m] / max(n, m)
```

`s_n = 1 - d_n`. LCS is the n=1 case of N-SIM; Levenshtein is the n=1 case of N-DIST.

BI-SIM (Kondrak & Dorr, COLING 2004 / AIM 2006) is N-SIM with n=2. It was the best single orthographic measure on USP LASA lists and is what the FDA POCA stack used.

## Soft-Bidist (Hadwan et al. 2021)

Same DP as N-DIST. `d_n` is replaced by a nine-case scale on the four characters `(a1 a2)` vs `(b1 b2)`:

| wt | Case | Default |
|----|------|---------|
| 1 | exact | 0 |
| 2 | no shared letters | 1 |
| 3 | transposition | 0 |
| 4 | second chars match | 0.2 |
| 5 | first chars match | 0.2 |
| 6–7 | crossed single match | 1 |
| 8 | insert | 0.5 |
| 9 | delete | 0.5 |

Similarity `1 - D / max(n, m)`. Table 4 of the paper (spelling-error pairs) is the regression test in `soft_bigram`.

The authors' C# keeps a sticky `cost2` across cells; that is a bug. Using wt8/wt9 as constant insert/delete costs reproduces Table 4 and matches the recurrence they wrote.

## Soft-Bisim (Millán-Hernández et al. 2019)

Same DP as N-SIM. `s_n` is replaced by a nine-case *credit* scale. The published genetic-algorithm weights score an exact bigram at 0.8, so `Soft-Bisim(x, x) < 1`. That is correct for their LASA ranker and wrong for Watchman. The Go default raises exact / all-equal to 1.0; `PaperWeights` keeps the published numbers.

On USP-858 they report Soft-Bisim beating BI-SIM, Trigram-2B, and NED on the sum of top-4 macro F-measure. Replacing Bisim with Soft-Bisim in Kondrak's four-way average (`Prefix, NED, Aline, Bisim`) was a further small gain.

## What we are not implementing

- The papers' genetic-algorithm weight search. Watchman cannot fit weights on OFAC without a labeled pair set; OpenSanctions Pairs (Smith et al. 2026) is the first public one at scale.
- Language detection and ad-hoc phonetic rewrites. Those were in an earlier `soft-bisim` draft, did not compile, and are not in the papers. Watchman's Soundex boost and Arabic phonetics already cover that layer.
- Full-name scoring. Token alignment (`BestPairsJaroWinkler`) stays in Watchman.

## Duality

Soft-Bidist is the right inner scorer when the error model is edits (OCR, typos, doubled letters). Soft-Bisim is the right inner scorer when the error model is look-alike / subsequence (aliases, transliteration leftovers). Jaro-Winkler remains strong on prefix-heavy Latin names, which is why Watchman uses it today. The honest test is a three-way bake-off on OFAC true/false-positive pairs, not a paper table of spelling errors.
