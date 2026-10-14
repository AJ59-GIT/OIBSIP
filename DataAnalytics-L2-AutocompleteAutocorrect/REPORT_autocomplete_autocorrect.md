# Report · Autocomplete and Autocorrect Data Analytics

## 1. Objective

Analyse the efficiency and accuracy of text-prediction (autocomplete) and spelling-correction (autocorrect) algorithms on a real text corpus, implementing and comparing multiple approaches for each.

## 2. Corpus

The supplied file `The_Project_Gutenberg.csv` is the plain text of *Alice's Adventures in Wonderland* (Project Gutenberg eBook #11) rather than a table. It was read as text; the licence header/footer and table of contents were removed.

| Statistic | Value |
|---|---|
| Paragraphs / chapters | 789 / 12 |
| Sentences | 1,630 (average 16.3 tokens) |
| Tokens (all) | 26,571 |
| Tokens after stop-word removal | 12,428 (53.2% removed) |
| Vocabulary | 2,610 distinct words, 1,131 (43%) seen only once |

## 3. Preprocessing

Curly quotes and apostrophes were normalised, Gutenberg italics markers (`_word_`) removed and dashes turned into spaces. Text was split into sentences, lower-cased and tokenised with a regular expression that keeps letters and inner apostrophes (so `don't` survives and punctuation and digits are dropped). Stop-words were removed using the NLTK English list. Two token views are kept deliberately: the **full stream** for autocomplete and autocorrect (a predictor must be able to suggest *the* and *of*, and a dictionary must contain them) and the **content-word stream** for frequency analysis.

## 4. Word-frequency analysis

*The* alone accounts for about 6% of tokens (1,635 occurrences); the top 20 is almost entirely function words, with only *said* and *alice* as content words. After stop-word removal, *said* (462) and *alice* (386) dominate, followed by *little*, *one*, *know*, *queen*, *king* and *turtle*, reflecting a dialogue-heavy story with a small cast. The heavy tail (43% of words occur once) limits what n-gram models can learn. Contractions such as *i'm* (59 occurrences) slip through the stop-word list.

## 5. Autocomplete

### 5.1 Method

Frequency-based models trained on a random 80% of sentences (1,304) and tested on the other 20% (326): **unigram** (global most frequent words), **bigram** (previous word) and **trigram with stupid back-off** to bigram then unigram. Sentence starts are padded with `<s>`. Each model can also complete a partly typed word. Demos use 12 prefixes (for example `the white` → *rabbit, kid, and*; `i don't` → *know, think, believe*) plus partial-word cases (`the whi` → *white, whiting, while*).

### 5.2 Metrics

Every position in the held-out sentences is a query (context, true next word) and the model returns three suggestions.

- **Recall@3 (= Hit@3):** queries where the true word is among the 3 suggestions ÷ all queries.
- **Precision@3:** correct suggestions shown ÷ all suggestions shown (capped at 33% because only one of three can be right).
- **Hit@1**, **coverage**, and **Hit@3 with 2 letters typed** (completion given the first two characters).
- **Efficiency:** training time, stored n-gram entries, time per query.

### 5.3 Results (5,316 queries; 5.4% of target words are unseen in training)

| Model | Hit@1 | Hit@3 (Recall@3) | Precision@3 | Hit@3, 2 letters typed | Fit time | Stored n-grams | µs / query |
|---|---|---|---|---|---|---|---|
| Unigram | 0.060 | 0.122 | 0.041 | 0.665 | 0.010 s | 2,347 | 1.2 |
| Bigram | 0.146 | 0.258 | 0.086 | 0.754 | 0.025 s | 13,883 | 1.7 |
| Trigram + back-off | 0.160 | 0.280 | 0.093 | 0.757 | 0.100 s | 31,174 | 2.1 |

- Context roughly doubles accuracy (Hit@3 12.2% → 25.8%).
- The trigram adds only 2.2 points over the bigram for about 2.2 times the memory; with about 21,000 training tokens most trigrams occur once. Larger corpora and smoothing (for example Kneser-Ney) are needed for higher orders to pay off.
- Typed letters matter more than context: with two letters typed even the unigram reaches 66.5%.
- Coverage is 100% for every model because all back off to the global word list. Speed is not a differentiator at this scale (microseconds per query).

## 6. Autocorrect

### 6.1 Method

A word not in the dictionary is treated as a typo and replaced by the dictionary word with the fewest edits (at most 2), ties broken by corpus frequency; known words are never changed. The dictionary is the 2,347 words of the training sentences.

| Approach | Operations | Candidate search |
|---|---|---|
| A. Levenshtein | insert, delete, substitute | scan words within ±2 letters of the length, early exit |
| B. Damerau-Levenshtein | also adjacent transposition | same scan |
| C. Norvig-style | same as B | generate all strings within 1 (then 2) edits, keep dictionary words |
| D. `pyspellchecker` (optional) | Levenshtein ≤ 2 | library, loaded with the same words |

Custom edit-distance code was verified with unit tests (for example kitten/sitting = 3, teh/the = 2 under Levenshtein and 1 under Damerau-Levenshtein).

### 6.2 Test sets

1. **Hand-written:** about 40 deliberately misspelled words (43 usable after filtering), covering dropped, extra, wrong and swapped letters and phonetic spellings.
2. **Synthetic:** 1,000 misspellings (250 each of deletion, insertion, QWERTY-neighbour substitution and transposition, generated from dictionary words of 4+ letters, non-words only) plus 1,000 correctly spelled words from the **held-out text**, of which 58 are not in the dictionary.

### 6.3 Metrics

TP = a misspelling corrected to the intended word; FP = any change to a wrong word (including changing a correct word); FN = a misspelling not corrected to the intended word; TN = a correct word left alone.

- **Precision** = TP ÷ (TP + FP): of the changes made, how many were right.
- **Recall** = TP ÷ (TP + FN): of the misspellings, how many were fixed correctly.
- **F1** and **accuracy** as usual. A before/after confusion matrix records whether the word was correct before and after.

### 6.4 Results

Hand-written test: Levenshtein **38/43 (88%)**, Damerau-Levenshtein **42/43 (98%)**, Norvig-style **42/43 (98%)**. Levenshtein fails on swapped letters (`teh`, `wiht`, `tabel`, `tierd`). All three fail on `wich` (*with* and *which* are both one edit away and the more frequent word wins).

Synthetic test:

| Approach | TP | FP | FN | TN | Precision | Recall | F1 | Accuracy | ms / misspelled word |
|---|---|---|---|---|---|---|---|---|---|
| A. Levenshtein | 771 | 272 | 229 | 957 | 0.739 | 0.771 | 0.755 | 0.864 | about 13 |
| B. Damerau-Levenshtein | 862 | 181 | 138 | 957 | 0.826 | 0.862 | 0.844 | 0.910 | about 14 |
| C. Norvig-style | 864 | 179 | 136 | 957 | 0.828 | 0.864 | 0.846 | 0.910 | about 2 |

Recall by error type:

| Approach | Deletion | Insertion | Substitution | Transposition |
|---|---|---|---|---|
| A. Levenshtein | 0.704 | 0.996 | 0.856 | 0.528 |
| B. Damerau-Levenshtein | 0.704 | 0.996 | 0.856 | 0.892 |
| C. Norvig-style | 0.708 | 0.996 | 0.852 | 0.900 |

- Transpositions explain the whole A vs B gap (52.8% → 89.2%).
- Inserted letters are easiest (99.6%); dropped letters are hardest (about 70%) because many words are one insertion away.
- **Precision is below recall** because of 43 false corrections: **43 of the 58 correct but unseen words (74%) were wrongly changed**, identically for all methods. This is a dictionary limitation, not an algorithm limitation (also shown by `weather` → *feather* and `internet` → *interest*).
- B and C are essentially equal in accuracy (they search the same edits; the 2-word difference comes from tie handling), but **C is about 7 times faster** because it looks up a few hundred generated strings in a hash table instead of running dynamic programming against every similar-length word.

## 7. Algorithm comparison

| | Best accuracy | Cost | Verdict |
|---|---|---|---|
| Autocomplete | Trigram + back-off (Hit@3 28.0%) | 31k n-grams, about 2 µs/query | Bigram is within about 2 points for less than half the memory; the trigram pays off as the corpus grows |
| Autocorrect | Damerau-Levenshtein and Norvig-style (F1 about 0.84) | A: 13 ms, B: 14 ms, C: 2 ms per word | Counting a swap as one edit is the most valuable improvement; candidate generation gives the same accuracy faster |

Scan-based correctors scale with dictionary size; candidate generation scales with word length and alphabet (about 560 strings one edit away for a 10-letter word, over 300,000 two edits away). Real systems use indexes such as SymSpell, BK-trees, tries or finite-state transducers. Times here are from unoptimised pure Python on a 2,347-word dictionary.

## 8. Limitations compared with production keyboards (for example Gboard)

- **Model:** count tables over one book versus neural language models trained on very large data (publicly described by Google as small on-device recurrent networks trained partly with federated learning), which generalise to unseen sequences.
- **Vocabulary:** closed here; production systems use large multilingual vocabularies, subword models and adaptive user dictionaries.
- **Error model:** uniform edit costs here; production systems model touch positions, neighbouring keys, phonetic errors and user habits.
- **Context:** autocorrect here ignores surrounding words and cannot detect real-word errors (*there/their*); production systems score candidates with the language model.
- **Data and evaluation:** one Victorian novel with a random sentence split and synthetic single-error typos, versus real keystroke logs and live metrics.
- **Engineering and behaviour:** latency and memory budgets, offensive-content filters, personalisation, privacy, deciding whether to autocorrect at all.

## 9. Statistical caveats

About 5,300 autocomplete queries from one book and one random seed: small differences (such as B vs C) should not be over-interpreted. Synthetic typos contain one error per word; real typing often has several.

## 10. Conclusion and next steps

Use **Damerau-Levenshtein ranking with candidate generation** for correction and a **trigram model with back-off** for prediction, but the dominant limitation is the small corpus rather than the algorithm choice. Next steps: a much larger and more varied corpus with Kneser-Ney smoothing, a small neural language model, a keyboard- and phonetics-weighted noisy-channel error model, context-aware correction (including real-word errors), a fast index (SymSpell or BK-tree), and an adaptive user dictionary.

## 11. Reproducibility and caveats

Seed 42. Requirements are in `req_nlp.txt`. NLTK and `pyspellchecker` were not installed where the notebook was tested: the built-in stop-word list was used and the optional `pyspellchecker` comparison is skipped automatically if the library is missing; those two paths are untested.
