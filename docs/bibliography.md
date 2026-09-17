# Bibliography

PDFs live in [`docs/papers/`](papers/). Newer additions (2024–2026 and the Kondrak foundations) are marked ★.

## Core (implemented)

| File | Paper |
|------|--------|
| `Soft_Bigram_Distance.pdf` | Hadwan, Al-Hagery, Al-Sanabani, Al-Hagree. Soft Bigram distance for names matching. *PeerJ Comput. Sci.* 7:e465, 2021. |
| `Millan-2019-SoftBisim.pdf` / `SoftBiSim.pdf` | Millán-Hernández, García-Hernández, Ledeneva, Hernández-Castañeda. Soft Bigram Similarity to Identify Confusable Drug Names. MCPR 2019, LNCS 11524. |
| ★ `Kondrak_2005_N-Gram_Similarity_and_Distance.pdf` | Kondrak. N-Gram Similarity and Distance. SPIRE 2005, LNCS 3772. |
| ★ `Kondrak_Dorr_2004_Identification_of_Confusable_Drug_Names.pdf` | Kondrak & Dorr. Identification of Confusable Drug Names. COLING 2004. |
| `An_Improved_N-gram_Distance_for_Names_Matching.pdf` | Al-Hagree, Al-Sanabani, Hadwan, Al-Hagery. E-N-DIST. ICOICE 2019. |

## Name matching surveys and methods

| File | Paper |
|------|--------|
| `A_Comparison_of_Personal_Name_Matching_Techniques_and_Practical_Issues.pdf` | Christen. ICDM Workshops 2006. Still the practical baseline (Jaro, Winkler, editex, n-gram, phonetic). |
| `A_Comparison_and_Analysis_of_Name_Matching_Algorit.pdf` | Comparison/analysis survey (same family). |
| `An_assessment_of_name_matching_algorithms.pdf` | Assessment of name-matching algorithms. |
| `Fuzzy_Name_Matching_Techniques.pdf` | Fuzzy name-matching techniques overview. |
| `Name_Matching_Across_Datasets.pdf` | Cross-dataset name matching. |
| `Machine_Learning_Based_Name_Matching__A_Logistic_Regression_Perspective.pdf` | Logistic-regression combiner over string features. |

## Language-specific

| File | Paper |
|------|--------|
| `Cross_linguistic_name_matching_in_English_and_Arab.pdf` | English/Arabic cross-script matching. Directly relevant to Watchman OFAC. |
| `Designing_an_Accurate_and_Efficient_Algorithm_for_Matching_Arabic_Names.pdf` | Arabic name matching (Al-Hagree et al.). |
| `A_FRAMEWORK_FOR_NAME_MATCHING_IN_ARABIC_LANGUAGE.pdf` | Arabic framework (scan of a short note). |
| `An_Empirical_Study_of_Chinese_Name_Matching_and_Ap.pdf` | Chinese names. |
| `Improving_the_Effectiveness_of_Name_Matching_Algorithms_with_a_Portuguese_Wordnet.pdf` | Portuguese + WordNet. |
| `2008_Gotal_Pouliquen_edit-distance.pdf` | Gotal & Pouliquen edit-distance work (JRC / multilingual news). |

## Privacy, ethnicity, composites

| File | Paper |
|------|--------|
| `Private_and_Secure_Fuzzy_Name_Matching.pdf` | Earlier private matching. |
| ★ `Kasyap_2024_Privacy_Preserving_Fuzzy_Name_Matching.pdf` | Kasyap, Atmaca, Maple, Cormode, He. Privacy-preserving Fuzzy Name Matching for Sharing Financial Intelligence. arXiv:2407.19979, 2024. MinHash + FHE over character n-grams; HSBC/Turing. |
| `Name-Ethnicity_Classification_and_Ethnicity-Sensit.pdf` | Ethnicity-sensitive matching. Handle with care in a sanctions product. |
| `Name_Similarity_for_Composite_Element_Name_Matching.pdf` | Composite (multi-field) element names. |
| `Use_of_a_Matching_Preference_Index_to_Empirically_.pdf` | Matching preference index. |
| `A_global_analysis_of_matches_and_mismatches_betwee.pdf` | Global match/mismatch analysis. |

## Sanctions / entity resolution (newer)

| File | Paper |
|------|--------|
| ★ `OpenSanctions_Pairs_2026.pdf` | Smith, Sesodia, Lindenberg, Schroeder de Witt. OpenSanctions Pairs: A Large-Scale Dataset for Pairwise Entity Matching. arXiv:2603.11051, 2026. 755k expert-labeled pairs over 1M entities, 293 sources. Blocking uses character n-grams. This is the eval set Watchman did not have. |

## Not in this folder, worth knowing

- Kondrak & Dorr, *Automatic identification of confusable drug names*, Artificial Intelligence in Medicine 36(1), 2006. Journal version of the COLING paper; BI-SIM + ALINE + average.
- Lambert et al., look-alike/sound-alike medication errors, *Am J Health-Syst Pharm*, 2017. Trigram-2B, NED, Editex on LASA.
- Christen, *Data Matching*, Springer 2012. Book; Table 6 of the Soft-Bidist paper is taken from it.
- Ukkonen, Approximate string-matching with q-grams, TCS 1992. The *set* n-gram metric, not Kondrak's DP.
- OpenSanctions `nomenklatura` — production n-gram blocker for sanctions entity resolution.
