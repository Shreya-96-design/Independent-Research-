# Independent-Research-
# Scientific MT Reproductions

## Overview

Reproducibility study of two peer-reviewed NLP papers on LLM-based machine translation and multilingual unsupervised text simplification. Both reproductions focus on understanding core methods and improved MT and text accessibility for scientific documents across European languages.

This repository contains technical reports documenting what was reproduced, the results obtained, and the insights gained. The reports are the primary output they demonstrate understanding of the methods, not just execution.

## Papers

| # | Paper | Venue | Focus | Report |
|---|-------|-------|-------|--------|
| 1 | Jiao et al. (2023) — *Is ChatGPT a Good Translator?* | arXiv:2301.08745 | LLM-based MT evaluation, prompting strategies | [Report →](paper1_llm_mt_eval/REPORT.md) |
| 2 | Martin et al. (2021) — *MUSS: Multilingual Unsupervised Sentence Simplification* | arXiv:2005.00352 | Paraphrase mining, controllable simplification | [Report →](paper2_muss/REPORT.md) |

---

## Why these two papers

- "Paper 1" directly addresses question: how well do LLMs translate complex text, and what inference-time strategies improve quality?
- "Paper 2" addresses simplification component: how can we simplify text across multiple languages without labelled data?

Together they gave me hands-on familiarity with the evaluation metrics (BLEU, COMET, SARI), the model architectures (Mistral-7B, mBART), and the core research questions I will work on during the PhD.

---

## Key findings

**Paper 1:** Pivot prompting consistently outperforms zero-shot across language pairs but all three strategies degrade on scientific vocabulary.

**Paper 2:** The word-rank control token is the most impactful dimension for simplification forcing substitution of rare vocabulary matters more than length or lexical complexity control, especially for technical text.
