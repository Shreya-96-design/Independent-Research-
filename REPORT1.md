# Technical Report: Is ChatGPT a Good Translator?

## 1. What this paper is about

In this paper it evaluate ChatGPT (GPT-3.5 and GPT-4) as a machine translation system by comparing three prompting strategies across multiple language pairs. The core question is: without any MT specific training, can a general purpose LLM match dedicated MT systems like DeepL or Google Translate?

The three strategies tested are:

1) Zero-shot — simply ask the model to translate with no additional framing
2) Expert-role — prompt the model to act as an expert translator before translating
3) Pivot prompting — for distant language pairs, route through English as an intermediate language

The paper finds ChatGPT is competitive with commercial MT on high resource European pairs (EN↔DE, EN↔FR) but weaker on low resource or linguistically distant pairs. It also finds that pivot prompting helps for distant pairs where neither the source nor target is English.

## 2. What I reproduced

I reproduced the 'prompt strategy comparison' from Table 2 of the paper, the core experiment comparing how much the choice of prompt affects translation quality.

**Setup:**
- Model: `mistralai/Mistral-7B-Instruct-v0.2` (open-source; freely available proxy for GPT-3.5)
- Language pair: EN→DE (English to German)
- Test sentences: 10 WMT-style news sentences with human reference translations
- Metrics: BLEU (sacrebleu) and COMET-22 (Unbabel/wmt22-comet-da)

**Why Mistral-7B instead of GPT-3.5:**
GPT-3.5 and GPT-4 require paid API access. Mistral-7B-Instruct is a freely available open-source model that follows instructions in a similar way. Lower absolute scores are expected, but the relative ordering of strategies should be consistent which is the key thing being tested.

## 3. My results

### Prompt strategy comparison (EN→DE, 10 sentences)

| Strategy | My BLEU | My COMET | Paper BLEU (GPT-3.5) |
|----------|---------|----------|----------------------|
| Zero-shot | 20.88 | 0.9337 | 26.8 |
| Expert-role | 23.10 | 0.9291 | 27.4 |
| Pivot | 20.71 | 0.9255 | 28.1 |

### Observation 1: BLEU ranking partially matches the paper

On BLEU, expert-role (23.10) outperforms zero-shot (20.88), which matches the paper's finding that role framing improves translation quality. However, pivot (20.71) is the weakest strategy in my experiment, the reverse of the paper's finding where pivot is the best.

This reversal makes sense on reflection. The paper recommends pivot prompting specifically for 'distant language pairs' (e.g. Chinese→German, Japanese→French) where neither the source nor target is English. For EN→DE, which is a high resource pair where English is already the source, pivot prompting adds an unnecessary intermediate step that introduces redundancy rather than helping. Running EN→DE with pivot is essentially asking the model to confirm its English understanding before translating, an extra step that slightly degrades output rather than improving it.

### Observation 2: BLEU and COMET rankings disagree

On COMET, zero-shot (0.9337) outperforms both expert-role (0.9291) and pivot (0.9255), the exact opposite of the BLEU ranking where expert-role wins.

This disagreement between metrics is itself a meaningful finding. BLEU measures n-gram overlap with the reference translation, it rewards exact wording. COMET uses a neural model trained on human judgement scores, it rewards semantic similarity and fluency even when phrasing differs from the reference. A translation can score higher on COMET but lower on BLEU if it uses different but equally correct phrasing.

In this case, zero-shot translations may be more semantically natural (higher COMET) while expert-role translations more closely match the specific wording of the reference (higher BLEU). Neither metric tells the complete story alone, which is precisely the argument made by HELM (Liang et al. 2022) and reinforced here.


## 4. Sample translations

**Source sentence:**
*"The European Central Bank raised interest rates for the third consecutive time."*

| Strategy | Translation |
|----------|------------|
| Zero-shot | Die Europäische Zentralbank hat die Zinssätze zum dritten Mal in Folge erhöht. |
| Expert-role | Die Europäische Zentralbank erhöhte die Zinssätze zum dritten Mal hintereinander. |
| Pivot | Die Europäische Zentralbank hat die Zinsen zum dritten Mal in Folge erhöht. |
| Reference | Die Europäische Zentralbank erhöhte die Zinssätze zum dritten Mal in Folge. |

All three translations are accurate. The differences are subtle, verb tense (raised vs. has raised), synonym choice (hintereinander vs. in Folge). These small differences explain why BLEU scores vary while COMET scores remain relatively close.


## 5. What I learned

**On metrics:** The disagreement between BLEU and COMET rankings in this experiment is one of the most concrete demonstrations of why using a single metric is unreliable. BLEU penalises expert-role's translation for using "hintereinander" instead of the reference's "in Folge" even though both mean the same thing. COMET recognises semantic equivalence. For scientific MT, this distinction matters even more, since technical terms often have multiple valid translations and surface level n-gram overlap is a poor proxy for terminological accuracy.

**On pivot prompting:** The paper presents pivot prompting as a strategy for distant language pairs. My results confirm this for EN→DE (a close, high-resource pair) pivot is the weakest strategy, not the strongest. This conditional effectiveness is important for the right inference-time strategy depends on the language pair and domain, not just the task type. There is no single best prompt.

**On absolute vs. relative scores:** My BLEU scores (20–23) are notably lower than the paper's (26–28). The gap is partly explained by the model (Mistral-7B vs. GPT-3.5) but also by test set size - 10 sentences is a small sample, and BLEU is sensitive to corpus size. The COMET scores (0.92–0.93) are high in absolute terms, reflecting that the translations are genuinely good even if they do not match the reference wording exactly.

## 7. References

- Jiao et al. (2023). *Is ChatGPT a Good Translator? Yes with GPT-4 as the Engine.* arXiv:2301.08745
- Rei et al. (2020). *COMET: A Neural Framework for MT Evaluation.* EMNLP 2020.
- Papineni et al. (2002). *BLEU: a Method for Automatic Evaluation of Machine Translation.* ACL 2002.
- Liang et al. (2022). *Holistic Evaluation of Language Models.* arXiv:2211.09110.
