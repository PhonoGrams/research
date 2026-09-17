# Survey notes

Read against Watchman (OFAC / AML name search), not against LASA pharmacy software. Citations are to PDFs in `papers/`; see [bibliography.md](bibliography.md).

## What the literature actually agrees on

1. **Exact match is not a sanctions matcher.** Christen 2006 is still the right intro: names vary by spelling, transcription, missing tokens, and cultural order. Watchman already lives in the approximate-string world (Jaro-Winkler, not equality).

2. **Character n-grams beat naive Levenshtein on look-alikes, and lose to Jaro-Winkler on prefix-heavy Latin names.** Kondrak 2005 / COLING 2004 showed BI-SIM winning on USP LASA lists. Christen 2006 and Watchman's own OFAC bake-offs showed Jaro-Winkler winning on Western personal names. These are different error models. Shipping only one inner scorer is a product choice, not a theorem.

3. **The interesting n-gram work is DP over n-grams, not Jaccard on the n-gram set.** Ukkonen q-grams (set difference) go to zero on `Verelan`/`Virilon` (Millán-Hernández's example). Kondrak N-SIM still fires because `e/i` is a half-credit bigram and the rest aligns. Soft-Bisim exists to make that half-credit less dumb.

4. **Weights are task-specific.** Soft-Bidist and Soft-Bisim both admit that the nine-case scale should be fitted. They fitted it on spelling-error lists and LASA pairs. Those weights are a *starting point* for OFAC, not a result. OpenSanctions Pairs (2026) is the first public labeled set large enough to refit without leaking Watchman production data.

5. **Phonetics are a separate layer.** Soundex / Double Metaphone / ALINE / Arabic phonetization do a different job (sound-alike, cross-script). Watchman already has Soundex boost and Arabic phonetic mapping.

## Paper-by-paper, short

**Kondrak 2005.** The foundation. N-DIST / N-SIM, affixing, positional n-gram cost, proof that Levenshtein and LCS are n=1. Implement the n=2 cases as `KondrakDistance` / `Bisim` and treat Soft-* as a cost-scale swap.

**Kondrak & Dorr 2004.** BI-SIM on confusable drug names; recall-at-rank evaluation that Millán-Hernández copies. Combined measure (Prefix + NED + Bisim + Aline) beats any single measure. That is the strongest "how to integrate" result in the whole pile: **blend**, do not replace.

**Millán-Hernández 2019 (Soft-Bisim).** Softens BI-SIM's 0/0.5/1 scale with a GA on USP-858. Gains are real on LASA rank F-measure, modest in absolute terms (top-1 F from 48.9 to 51.1). The published exact-match weight of 0.8 must not be used as a Watchman similarity (identity would not be 1).

**Hadwan 2021 (Soft-Bidist).** Softens BI-DIST. Table 4 spelling pairs are a useful unit test and a poor Watchman test. The Go package follows the written recurrence and the authors' C# case order, with constant insert/delete costs.

**Al-Hagree 2019 (E-N-DIST).** Same group, earlier: more substitution/transposition states (`2n+1-1` vs `2n`). Soft-Bidist is the cleaned-up version. Do not implement both.

**Christen 2006.** Practical taxonomy. Jaro / Winkler / Editex / q-grams / phonetics. Watchman's current stack is "Christen-correct": Winkler at the token level, with extra OFAC false-positive controls.

**Cross-linguistic English/Arabic, Arabic name matchers.** Relevant because OFAC is full of Arabic, Cyrillic, and Chinese names in Latin transcription. Character-bigram DP on Latin transcriptions helps; it does not replace a real transliterator. Watchman's `internal/embeddings` cross-script path is the modern complement, not a competitor.

**Kasyap et al. 2024.** Privacy-preserving fuzzy name matching for financial intelligence (Turing / Warwick / HSBC). Character n-grams → MinHash → FHE. Confirms n-grams are what banks actually ship when they cannot send raw names. Not an accuracy paper; a systems paper. No change to the scorer, but it is why a clean `Similarity(a, b) float64` API matters (it is the inner kernel of a PSI/FHE scheme too).

**OpenSanctions Pairs 2026.** 755,540 expert-labeled entity pairs, sanctions/PEP/OSINT, n-gram blocking in `nomenklatura`. Two Watchman takeaways: (1) name-only matching fails on common names that share country and program (their 0.98 false positive); Watchman's identifier and country features already know this. (2) This is the eval set to use before calling Soft-Bisim "better than Jaro-Winkler" on sanctions.

**Ethnicity-sensitive matching.** Read it, do not copy it into a screening product. OFAC matching that changes thresholds by inferred ethnicity is a civil-liberties bug.

**ML / logistic regression combiners.** Same idea as Kondrak's average and Watchman's weighted BestPairs. A learned blend of JW + Soft-Bisim + Soft-Bidist on OpenSanctions Pairs is the obvious v2, and it should live in Watchman, not in these libraries.

## Newer work, 2022–2026, that did *not* replace these algorithms

LLM fuzzy matching (e.g. political-science ChatGPT matchers, 2024) wins on aliases (`DPRK`/`North Korea`) and loses on latency, cost, and determinism. Watchman cannot call an LLM per SDN token pair. Embeddings (Watchman already has an optional path) are the right place for that class of alias.

Named-entity normalisation papers for investigations (2025) still use gestalt / LCS as the cheap pass. Character n-grams remain the cheap pass.

The Go packages implement Kondrak n=2 plus the two soft scales: `Similarity` in `[0, 1]`, identity = 1, no phonetic transforms. Weight fitting for OFAC belongs on OpenSanctions Pairs, inside Watchman.
