# Technical Report: MUSS — Multilingual Unsupervised Sentence Simplification

## 1. What this paper is about

This paper propose MUSS, a system for text simplification that requires 'no labelled (complex, simple) training pairs'. Instead it works in two stages:

**Stage 1 — Paraphrase mining:** Mine semantically similar but differently expressed sentence pairs from Common Crawl (a massive web corpus) using multilingual sentence embeddings. Pairs where one sentence is meaningfully shorter than the other serve as a proxy for (complex, simple) pairs.

**Stage 2 — Controllable simplification:** Fine-tune a multilingual seq2seq model (mBART) on the mined pairs, with control tokens prepended to each input. The control tokens allow the user to specify at inference time how much to simplify:

| Token | Controls | Example values |
|-------|----------|---------------|
| `LEN_x` | Output length as fraction of input | 0.6 = 40% shorter |
| `LEX_x` | Lexical complexity | 0.6 = much simpler words |
| `RANK_x` | Maximum word rank (rarity) | 5000 = very common words only |

The paper evaluates on English (WikiLarge, Newsela), French (ALECTOR), and Spanish benchmarks using SARI, a metric that rewards the right combination of adding, keeping, and deleting words. Results closely match or outperform prior supervised systems despite using no labelled data.

## 2. What I reproduced

I focused on understanding the 'controllable generation mechanism', the core novel contribution of the paper and the **SARI evaluation pipeline**.

**Setup:**
- Model: `Helsinki-NLP/opus-mt-tc-big-en-de` (proxy model - note below)
- Control tokens: length ratio (LEN), lexical complexity (LEX), word rank (RANK)
- Test sentences: 5 sentences spanning general and scientific domains
- Evaluation: SARI scores across four control token settings

**What I did not reproduce:** The paper's full results require the authors' fine-tuned mBART checkpoint, trained specifically on their mined paraphrase data with control token conditioning. Obtaining this requires either downloading ~200GB of pre-mined data or running the full mining pipeline on Common Crawl, neither is feasible on a standard Colab session. I used Helsinki-NLP/opus-mt as a proxy to study the SARI evaluation pipeline and understand how control tokens are meant to work.


## 3. My results

### SARI scores by control setting (proxy model, 5 sentences)

| Control setting | SARI |
|----------------|------|
| Default (LEN=0.8, LEX=0.8, RANK=10000) | 32.3 |
| Short — length control (LEN=0.6) | 32.3 |
| Simple vocab — lex control (LEX=0.6) | 32.3 |
| Common words — rank control (RANK=5000) | 32.3 |

**Paper's reported SARI on WikiLarge (full MUSS system):** 42.53

### Key observation: identical scores across all settings

All four control token settings produced **identical outputs and identical SARI scores (32.3)**. This is not a bug, it reveals something fundamental about how controllable generation works.

The Helsinki-NLP opus-mt model was never trained with control token conditioning. It therefore ignores the LEN/LEX/RANK prefix entirely and produces the same translation regardless of what the tokens say. This directly demonstrates why the paper's fine-tuning step is essential: **control tokens only work if the model has been trained to attend to them**. A standard MT model treats the prefix as noise. The paper's MUSS system achieves controllability because it has seen millions of (prefix + complex sentence → simple sentence) training pairs where the prefix reliably predicts the output style.

The SARI gap (32.3 vs. 42.53) is explained by two factors: the proxy model was not trained for simplification, and the translations preserve technical vocabulary instead of replacing it with simpler equivalents.

## 4. Sample outputs

All four control settings produced identical translations for all five sentences. Two representative examples:

**Sentence 1:**
- Input: *"The administration of the pharmaceutical compound resulted in a significant reduction of inflammatory biomarkers."*
- All settings output: *"Die Verabreichung der pharmazeutischen Verbindung führte zu einer signifikanten Reduktion entzündlicher Biomarker."*
- Reference: *"The drug significantly reduced inflammation markers."*

**Sentence 2:**
- Input: *"Photosynthesis is the process by which plants convert light energy into chemical energy stored in glucose."*
- All settings output: *"Photosynthese ist der Prozess, durch den Pflanzen Lichtenergie in chemische Energie umwandeln, die in Glukose gespeichert ist."*
- Reference: *"Plants use sunlight to make food and store it as sugar."*

**Observation:** The proxy model translates accurately but does not simplify, it preserves the same technical vocabulary and sentence structure as the input. The reference simplifications use entirely different vocabulary ("the drug" instead of "the pharmaceutical compound," "sunlight" instead of "light energy"). This gap in vocabulary substitution is the primary reason SARI is lower than the paper's system.


## 5. What I learned

**On SARI:** SARI evaluates three separate operations, adding words that appear in the reference but not the source, keeping words that appear in both, and deleting words from the source that do not appear in the reference. A model that simply translates without simplifying scores well on "keep" but poorly on "delete." The proxy model here scores 32.3 rather than 42.53 primarily because it fails the delete component, it preserves technical vocabulary instead of replacing it.

**On control tokens:** The most important finding is that control tokens are not a prompt engineering trick, they are a training time commitment. The model must be trained on examples where different prefix values reliably predict different output styles before it learns to respond to them. This experiment made that concrete: changing the prefix from LEN=0.8 to LEN=0.6 had zero effect on an untrained model. This is why the paper's unsupervised paraphrase mining pipeline is the real contribution without that training data, the control tokens do nothing.

**On unsupervised learning:** The striking claim in MUSS is that it matches supervised systems without ever seeing a human annotated (complex, simple) pair. This experiment helped me understand why that is plausible: the model learns to associate the control prefix with a simplicity distribution from millions of mined paraphrase pairs. The control tokens then let the user dial into that learned distribution at inference time. The quality of the mined data not the model architecture is what determines whether this works.


## 7. References

- Martin et al. (2021). *MUSS: Multilingual Unsupervised Sentence Simplification by Mining Paraphrases.* arXiv:2005.00352
- Martin et al. (2020). *Controllable Sentence Simplification.* LREC 2020. (ACCESS — the predecessor system)
- Xu et al. (2016). *Optimizing Statistical Machine Translation for Text Simplification.* TACL. (SARI metric)
