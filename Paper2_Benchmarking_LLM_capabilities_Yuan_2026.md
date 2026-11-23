# PAPER-002

## Title: Benchmarking LLM’s Capability in Reasoning over Conflicting Web References

**Authors:** Yizhen Yuan, Rui Kong, Dongze Li, Yuanchun Li, Yunxin Liu  
**Year:** 2026  
**Venue:** Proceedings of the 64th Annual Meeting of the Association for Computational Linguistics (ACL 2026), Long Papers, pages 303–322.

---

## 1. PROBLEM

Large Language Models (LLMs) integrated with Retrieval-Augmented Generation (RAG) retrieve information from multiple web sources to answer user questions.

However, retrieved documents may contain contradictory claims, different opinions, or inconsistent information.

The main problem is that LLMs often generate a single answer without properly identifying, separating, and explaining the different viewpoints present in the retrieved documents.

The paper investigates whether LLMs can:
- Identify conflicting information across multiple sources.
- Group documents expressing similar viewpoints.
- Recover different possible answers.
- Explain the reasons supporting each viewpoint.

## 2. MOTIVATION

- Real-world web sources are diverse and may contain unreliable or contradictory information.
- RAG systems generally provide all retrieved documents to an LLM and expect it to synthesize an answer.
- A model may produce a plausible answer while ignoring important disagreements among sources.
- Existing benchmarks primarily evaluate reasoning with consistent information or short contradictory text pairs.
- There is a need to evaluate whether LLMs can reason over multiple, potentially conflicting viewpoints from real-world documents.

## 3. EXISTING WORK

The paper discusses several existing benchmarks:

**Natural Language Inference (NLI):**
- MultiNLI
- SNLI
- ANLI

These datasets evaluate relationships between premise and hypothesis, such as entailment, contradiction, and neutrality.

**Other contradiction-related benchmarks:**
- WikiContradict: Evaluates real-world knowledge conflicts using Wikipedia revisions, but contains only 253 examples with two viewpoints.
- BoardGameQA: Evaluates reasoning over contradictory information using synthetically generated examples.
- TruthfulQA: Evaluates whether models reproduce common human misconceptions.
- FalseQA: Studies answering questions with false premises.
- CREPE: Studies questions containing false presuppositions.

**Limitations of existing work:**
- Many datasets contain only short text or sentence-level contradictions.
- Most focus on two contradictory viewpoints.
- Some datasets are synthetically generated.
- They do not adequately evaluate reasoning over long, heterogeneous web documents containing multiple viewpoints.

## 4. GAP

There is a lack of a realistic benchmark that evaluates LLMs' ability to reason over multiple conflicting viewpoints obtained from real-world web documents.

Specifically, existing benchmarks do not adequately evaluate:

1. Whether an LLM can correctly group documents according to their viewpoints.
2. Whether it can recover all major possible answers.
3. Whether it can explain the supporting reasons for each answer.
4. Whether it can handle more than two distinct viewpoints.

The paper addresses this gap by introducing ConfRAG.

## 5. MAIN IDEA

The authors introduce **ConfRAG**, a benchmark for evaluating LLM reasoning over conflicting web references.

Instead of expecting one correct answer, ConfRAG allows multiple answer clusters, where each cluster represents a distinct viewpoint supported by a subset of retrieved documents.

The benchmark evaluates three capabilities:

1. Answer Clustering: Group documents that express similar viewpoints.
2. Answer Coverage: Recover all major answers or viewpoints.
3. Reason Coverage: Recover the supporting reasons associated with each viewpoint.

The central idea is to evaluate whether an LLM can organize and explain conflicting information rather than simply generate one answer.

## 6. METHOD

### A. Dataset Construction

The authors construct the dataset using a five-stage pipeline.

**Step 1: Question Collection**
- Generate controversial questions from six domains:
  - Health
  - Trivia
  - Lifestyle habits
  - Disease prevention
  - Science and technology
  - Nutrition and food
- Also incorporate questions from Natural Questions, ELI5, and Yahoo Answers.
- Apply filtering to select questions likely to generate different viewpoints.

**Step 2: Web Retrieval**
- Extract keywords from each question.
- Use Google Search through SerpAPI.
- Retrieve up to 40 websites per question.
- Use Jina's document reader pipeline to extract webpage content.
- Retain approximately 10 valid documents per question.

**Step 3: Answer and Reason Extraction**
- Use GPT-4o to extract an implied answer from each document.
- Extract supporting reasons for each answer.
- Evaluate the relevance and trustworthiness of the retrieved content.

**Step 4: Answer Clustering**
- Use LLM-based similarity judgments to group similar answers.
- Each cluster contains:
  - A representative answer.
  - Supporting document indices.
  - A list of supporting reasons.

**Step 5: Human-in-the-loop Annotation**
- Human annotators inspect the automatically generated clusters.
- Verify consistency within clusters.
- Check the correctness and completeness of answers and reasons.
- Identify contradictions between clusters.
- Resolve disagreements through an additional adjudicator.

### B. Evaluation Tasks

**Task 1: Answer Clustering**

Measures whether the model correctly groups documents into semantically coherent clusters.

Input: Retrieved documents.

Output: Document indices assigned to different clusters.

Metric: Normalized Mutual Information (NMI).

**Task 2: Answer Coverage**

Measures whether the model recovers all major gold-standard answer clusters.

Metric: Bipartite maximum matching using keyword-based substring similarity.

**Task 3: Reason Coverage**

Measures whether the model recovers the supporting reasons associated with each answer cluster.

Metric: Bipartite maximum matching using reason keywords and substring similarity.

All three metrics range from 0 to 1.

## 7. MATHEMATICS

### A. Normalized Mutual Information (NMI)

NMI measures the similarity between predicted document clusters and ground-truth clusters.

\[
NMI(U,V)=\frac{2I(U;V)}{H(U)+H(V)}
\]

Where:

- \(U\): Ground-truth clustering.
- \(V\): Predicted clustering.
- \(I(U;V)\): Mutual information between the two cluster assignments.
- \(H(U)\): Entropy of the ground-truth clustering.
- \(H(V)\): Entropy of the predicted clustering.

Interpretation:

- NMI = 1: Perfect agreement.
- NMI = 0: No meaningful agreement.

### B. Answer Coverage

Answer coverage measures how many gold-standard answer clusters are recovered by the model.

The paper uses bipartite maximum matching between predicted answers and gold answer keywords.

Each predicted answer can match at most one gold answer, and each gold answer can match at most one prediction.

### C. Reason Coverage

Reason coverage evaluates whether the model identifies the supporting reasons associated with each matched answer.

Reason matching is performed only after the corresponding answer has been matched.

Therefore, a missing answer also prevents its associated reasons from receiving credit.

## 8. DATA

**Dataset name:** ConfRAG

Dataset statistics:

- Total questions: 1,814.
- Average retrieved documents per question: 9.58.
- Average document length: Approximately 2,025 words.
- Questions with explicit contradictions: 1,037 (57.2%).
- Remaining questions: 777 (42.8%).

Data sources:
- Manually generated and curated questions.
- Natural Questions.
- ELI5.
- Yahoo Answers.

Document sources include:
- Wikipedia.
- Online forums.
- Blogs.
- News websites.

The dataset provides multiple answer clusters, supporting document indices, and structured reasons for each question.

## 9. EXPERIMENTS

### Experimental Setup

The authors evaluate seven LLMs:

1. GPT-4.1-mini
2. GPT-4.1
3. Qwen2.5-7B-Instruct
4. Qwen2.5-72B-Instruct
5. Claude-3-7-Sonnet
6. Llama-3.1-8B-Instruct
7. Llama-3.1-70B-Instruct

Experimental conditions:

- Zero-shot evaluation.
- No fine-tuning.
- Same standardized prompt for all models.
- Structured JSON output.
- Five randomly sampled documents from approximately ten retrieved documents per question.
- The ground-truth number of answer clusters (\(k\)) is provided to the model.

### Human Evaluation

- 150 randomly selected examples.
- Five human annotators.
- Each annotator groups documents into viewpoints and provides answers and supporting reasons.
- Human performance is used as a reference for model performance.

### Ablation and Error Analysis

The authors investigate:
- Contradictory versus non-contradictory questions.
- Effect of the number of answer clusters.
- Structural errors in document partitioning.

## 10. RESULTS

### Main Results

| Model | Answer Clustering (NMI) | Answer Coverage | Reason Coverage | Valid Partition Rate |
|---|---:|---:|---:|---:|
| GPT-4.1-mini | 0.370 | 0.452 | 0.151 | 0.648 |
| GPT-4.1 | 0.466 | 0.448 | 0.157 | 0.763 |
| Qwen2.5-7B | 0.147 | 0.312 | 0.083 | 0.308 |
| Qwen2.5-72B | 0.373 | 0.368 | 0.133 | 0.706 |
| Claude-3-7-Sonnet | 0.459 | 0.362 | 0.114 | 0.849 |
| Llama-3.1-8B | 0.161 | 0.276 | 0.071 | 0.428 |
| Llama-3.1-70B | 0.178 | 0.331 | 0.105 | 0.489 |
| Human Evaluation | 0.691 | 0.510 | 0.275 | 1.000 |

### Important Observations

1. GPT-4.1 achieves the highest LLM answer clustering score (0.466).
2. GPT-4.1 also achieves the highest answer coverage (0.448, approximately).
3. Claude-3-7-Sonnet achieves the highest valid partition rate (0.849).
4. All models perform poorly on reason coverage, with scores below 0.16.
5. Human annotators outperform all evaluated LLMs on the three reasoning metrics.
6. Larger Qwen models perform better than smaller Qwen models.

### Ablation Findings

- Models perform better on contradictory questions than on non-contradictory questions across the three metrics.
- Increasing the number of answer clusters generally reduces answer and reason coverage.
- GPT-4.1 answer coverage decreases from approximately 0.48 for two-cluster examples to 0.32 for four-cluster examples.
- Models frequently make document partitioning errors, including duplicated, missing, and extra document indices.

## 11. BASELINES

The paper compares seven LLMs and human annotators.

**LLM baselines:**
- GPT-4.1-mini
- GPT-4.1
- Qwen2.5-7B-Instruct
- Qwen2.5-72B-Instruct
- Claude-3-7-Sonnet
- Llama-3.1-8B-Instruct
- Llama-3.1-70B-Instruct

**Human baseline:**

Human annotations provide a reference for the performance achievable on the benchmark.

No fine-tuning or additional training method is introduced as a competing baseline.

## 12. CONTRIBUTION

The paper makes three main contributions:

1. **ConfRAG Benchmark:** Introduces a realistic benchmark containing 1,814 questions and retrieved web documents with multiple viewpoints and structured rationales.

2. **Three Evaluation Tasks:** Defines answer clustering, answer coverage, and reason coverage to evaluate different aspects of contradiction-aware reasoning.

3. **Generalizable Construction Pipeline:** Proposes a pipeline that can retrieve web content, identify conflicting viewpoints, and organize them into answer clusters and supporting rationales for future benchmark construction.

## 13. LIMITATIONS

The authors identify several limitations:

1. **Language and Domain:** The benchmark currently focuses on English-language, open-domain questions. It does not cover multilingual or specialized domains such as medicine and law.

2. **Text-only Data:** The benchmark does not include multimodal sources such as images and videos.

3. **Retriever Evaluation:** The study assumes that retrieval has already been performed. It focuses on the generator's reasoning ability rather than evaluating the retrieval component.

4. **Known Number of Clusters:** Models receive the ground-truth number of answer clusters (\(k\)). In real-world applications, the number of viewpoints is generally unknown.

5. **Evaluation Strategy:** The authors evaluate the complete reasoning task directly. They suggest a future classify-then-reason approach, where the model first detects contradictions and then selects an appropriate reasoning strategy.

Additional methodological considerations:
- Answer and reason coverage rely on keyword-based substring matching, which may fail to recognize valid paraphrases.
- Human evaluation is conducted on only 150 examples.
- The benchmark evaluates whether viewpoints are recovered and organized; it does not independently establish which conflicting viewpoint is factually correct.

## 14. MY QUESTIONS

1. How can an LLM distinguish genuine contradictions from differences in wording or context?
2. How can we determine whether two documents express the same viewpoint semantically?
3. Can embedding-based clustering improve answer clustering compared with direct LLM-based clustering?
4. How reliable is keyword-based substring matching for evaluating semantic answer coverage?
5. Can we develop an evaluation metric that measures the quality of reasoning behind each answer cluster?
6. How can a model identify the correct number of viewpoints without being given \(k\)?
7. Can source credibility and reliability be incorporated into contradiction reasoning?
8. How can we distinguish between conflicting evidence and genuinely different interpretations of the same evidence?
9. Can formal verification or external knowledge graphs help validate claims extracted from conflicting sources?

## 15. MY IDEAS

1. **Semantic Clustering:** Investigate embedding-based clustering methods to group documents according to semantic similarity rather than relying exclusively on LLM-based similarity judgments.

2. **Source Reliability:** Incorporate source credibility, publication date, author expertise, and evidence quality into the reasoning process.

3. **Automatic Cluster Discovery:** Develop methods that infer the number of distinct viewpoints automatically instead of providing the ground-truth \(k\).

4. **Claim-level Verification:** Extract individual claims from documents and identify whether they are supporting, contradicting, or unrelated to one another.

5. **Evidence-grounded Reasoning:** Require every generated answer and explanation to be linked to specific document passages.

6. **Uncertainty Estimation:** Allow the model to express uncertainty when the retrieved sources do not provide sufficient evidence to resolve a disagreement.

7. **Formal and Logical Reasoning:** Explore whether logical representations of claims and their relationships can improve contradiction detection and reasoning reliability.

## 16. IMPORTANT REFERENCES

1. Hou et al. (2024). *WikiContradict: A Benchmark for Evaluating LLMs on Real-World Knowledge Conflicts from Wikipedia.*

2. Kazemi et al. (2023). *BoardGameQA: A Dataset for Natural Language Reasoning with Contradictory Information.*

3. Williams et al. (2018). *A Broad-Coverage Challenge Corpus for Sentence Understanding through Inference (MultiNLI).*

4. Nie et al. (2020). *Adversarial NLI: A New Benchmark for Natural Language Understanding.*

5. Lin et al. (2022). *TruthfulQA: Measuring How Models Mimic Human Falsehoods.*

6. Hu et al. (2023). *Won't Get Fooled Again: Answering Questions with False Premises.*

7. Manakul et al. (2023). *SelfCheckGPT: Zero-Resource Black-Box Hallucination Detection for Generative Large Language Models.*

---

## PAPER LINKS

- **Code and Dataset:** https://github.com/XaiverYuan/ConfRAG
- **Conference:** ACL 2026
- **Paper:** Benchmarking LLM's Capability in Reasoning over Conflicting Web References
