# PAPER-001

## Title:
Safe: Enhancing Mathematical Reasoning in Large Language Models via Retrospective Step-aware Formal Verification

**Authors:** Chengwu Liu, Ye Yuan, Yichun Yin, Yan Xu, Xin Xu, Zaoyu Chen, Yasheng Wang, Lifeng Shang, Qun Liu, Ming Zhang

**Year:** 2025

**Venue:** ACL 2025 — Proceedings of the 63rd Annual Meeting of the Association for Computational Linguistics (Long Papers), pp. 12171–12186

**Paper:** https://arxiv.org/abs/2506.04592  
**ACL Anthology:** https://aclanthology.org/2025.acl-long.594/  
**Official code:** https://github.com/liuchengwucn/Safe  
**Benchmark:** FormalStep

---

## 1. PROBLEM

Large Language Models (LLMs) can generate multi-step mathematical reasoning containing subtle errors. A single incorrect intermediate step can invalidate the remainder of a solution.

Existing reasoning-verification approaches often assign scores to reasoning steps or trajectories, but these scores may not provide explicit, checkable evidence of mathematical correctness.

This paper addresses the problem of **verifying the correctness of individual natural-language reasoning steps and using this evidence to select better mathematical solutions**.

---

## 2. MOTIVATION

Chain-of-Thought (CoT) prompting improves mathematical reasoning by encouraging models to generate intermediate steps.

However, longer reasoning traces also create more opportunities for errors.

The authors argue that mathematical correctness should ideally be supported by a proof rather than an opaque confidence score (without showing why or how the score is derived).

Lean 4 provides:
- A formal language for expressing mathematical propositions.
- A proof assistant that checks whether a proposed proof establishes a proposition.
- Machine-checkable evidence that a formal theorem follows from its assumptions.

The motivation is to combine natural-language reasoning with formal verification to improve reliability and interpretability.

---

## 3. EXISTING WORK

Relevant approaches include:

### Chain-of-Thought prompting
Wei et al. (2022): encourages LLMs to generate intermediate reasoning steps.

### Self-consistency
Wang et al. (2022): samples multiple reasoning paths and aggregates their answers.

### Process Reward Models (PRMs)
Lightman et al. (2023): evaluate intermediate reasoning steps using learned reward signals.

### Best-of-N sampling
Generate multiple candidate solutions, then select the one with the highest score.

### Automated theorem proving
Systems such as Lean 4 provide formal verification of mathematical propositions and proofs.

### Formalization with LLMs
Prior work explores translating natural-language mathematics into formal statements and proofs.

The paper combines retrospective step verification with formal theorem proving and reasoning-trajectory selection.

---

## 4. GAP

The paper identifies limitations in existing verification approaches:

1. PRMs generally provide learned scores rather than formal proof certificates.
2. Self-consistency can select answers without identifying which reasoning steps are incorrect.
3. Opaque verification scores provide limited interpretability.
4. Mathematical hallucinations in intermediate steps can propagate into later reasoning.
5. Existing formal verification approaches do not fully integrate step-level formal evidence into the selection of natural-language reasoning trajectories.

The key gap is **the lack of interpretable, evidence-based verification of generated reasoning steps within a practical inference-time reasoning framework**.

---

## 5. MAIN IDEA

The authors propose **Safe**, a retrospective, step-aware formal verification framework.

Its central idea is:

> Generate candidate mathematical reasoning trajectories, retrospectively formalize individual reasoning steps in Lean 4, verify those formal statements, and use the resulting verification evidence to rank/select reasoning trajectories.

Safe combines:
- Natural-language Chain-of-Thought reasoning.
- Step decomposition.
- Automatic formalization into Lean 4.
- Automated theorem proving.
- Step-level verification states.
- Retrospective scoring.
- Existing prospective process-reward information.

The paper also introduces **FormalStep**, a benchmark containing 30,809 formal statements for step-correctness theorem proving.
In conclusion, it present the following three main contributions:

(1) We propose a novel autoformalization task, which aims to generate formal statements that validate the correctness of one specific step instead of simply translating the problem. Our dataset comprising 30, 809 formal statements, referred to as FormalStep, will be released to facilitate auto-formalization and automated theorem proving in low compute settings. Our empirical findings indicate that, despite these statements being out of distribution, they can still be effectively addressed by off-the-shelf automated theorem provers, provided there is an adequate computational budget.

(2) We propose a step-level formal verifier that outputs one of four distinct states, depending on whether a given step can be auto-formalized by an LLM and resolved by an automated theorem prover, rather than providing numerical scores for each step. To the best of our knowledge, this is the first study utilizing formal mathematics language Lean 4 to verify the correctness of mathematical reasoning trajectories generated by LLMs.

(3) We propose a formal verification framework Safe that aggregates a sequence of states generated by the formal step verifier. Furthermore, we demonstrate that the retrospective scores generated by our formal verifier can be effectively integrated with the prospective scores, resulting in state-of-the-art performance.

---

## 6. METHOD

### 6.1 Candidate generation

The reasoning model generates multiple candidate solutions for a mathematical problem.

Each candidate is represented as a sequence of natural-language reasoning steps:

T = (s₁, s₂, ..., sₙ)

### 6.2 Step decomposition

Each reasoning trajectory is divided into individual steps.

The goal is to assess the mathematical correctness of intermediate reasoning rather than only the final answer.

### 6.3 Step auto-formalization

For a natural-language step, the framework generates a Lean 4 formal statement and attempts to construct a formal proof.

This connects natural-language reasoning with symbolic mathematical verification.

### 6.4 Formal verification

Lean checks whether the generated proof establishes the generated formal proposition.

The verification outcome is used to assign a step-level verification state.

Important distinction:

Lean verifies the formal proposition and proof supplied to it. It does not independently establish that the generated proposition faithfully captures the original natural-language step.

### 6.5 Retrospective verification

Unlike prospective verification, which estimates whether a future reasoning step is likely to be correct, Safe examines already-generated reasoning steps and seeks formal evidence for their correctness.

The framework uses the resulting step-level information to evaluate reasoning trajectories.

### 6.6 Trajectory scoring

The verification information is aggregated into a retrospective score.

The framework combines retrospective formal verification with prospective process-reward information to select among candidate reasoning trajectories.

### 6.7 Final answer selection

The candidate trajectories are ranked, and the selected solution supplies the final mathematical answer.

The framework is evaluated using Best-of-N selection, including Best-of-5 experiments.

---

## 7. MATHEMATICS

### 7.1 Formal propositions

A natural-language claim is translated into a Lean proposition:

P : Prop

A proof term is constructed:

p : P

If Lean accepts the proof, then the proposition P is established within Lean's trusted foundations and the assumptions/imports used.

### 7.2 Proof checking

The formal verification objective can be expressed abstractly as:

LeanCheck(P, p) = 1

when p is a valid proof term of P, and 0 otherwise.

This is an abstraction of proof checking, not the exact scoring formula used by Safe.

### 7.3 Reasoning trajectory

Let a candidate reasoning trajectory be:

T = (s₁, s₂, ..., sₙ)

Each step is associated with a formal verification state:

vᵢ ∈ V

where V represents the step-verification categories defined by the framework.

The trajectory-level retrospective score aggregates the verification states of its constituent steps.

### 7.4 Combining retrospective and prospective information

Safe combines retrospective formal-verification evidence with prospective process-reward information.

Conceptually:

Score(T) = Combine(Retrospective(T), Prospective(T))

This expression is conceptual, not the paper's exact equation.


### 7.5 Best-of-N evaluation

Given N candidate trajectories:

T₁, T₂, ..., Tₙ

Safe selects:

T* = argmax Score(Tᵢ)

The final answer is obtained from T*.

### 7.6 Formal correctness versus semantic fidelity

A crucial distinction:

Lean proves:

Γ ⊢ P

where Γ denotes the assumptions and context.

But the desired natural-language claim is C.

A sound end-to-end verification process additionally requires that:

Formalization(P) faithfully represents C.

Thus:

Formal proof correctness ≠ guaranteed natural-language interpretation correctness.

This distinction is particularly important for research on mathematical reliability of LLMs.

---

## 8. DATA

### FormalStep

The authors introduce FormalStep, a benchmark for step-correctness theorem proving.

- 30,809 formal statements.
- Designed to evaluate formal verification of mathematical reasoning steps.
- Used to study the ability to formalize and prove step-level mathematical claims.

### Mathematical evaluation datasets

Safe is evaluated across multiple mathematical reasoning datasets, including:

- MATH
- GSM8K
- CollegeMath

The experiments include different LLMs and different mathematical reasoning settings.

---

## 9. EXPERIMENTS

The paper evaluates Safe through experiments involving:

1. Mathematical reasoning performance across multiple datasets.
2. Different underlying reasoning models.
3. Best-of-N candidate selection.
4. Comparison with existing trajectory-selection approaches.
5. Retrospective formal verification and prospective reward information.
6. FormalStep step-correctness theorem proving.
7. Ablation studies examining components of the framework.

The main evaluation considers MATH-500, GSM8K, and CollegeMath.

---

## 10. RESULTS

The paper reports improved mathematical reasoning performance when Safe is used to select among generated candidate solutions.

Representative Best-of-5 results:

| Model | MATH-500 | GSM8K | CollegeMath |
|---|---:|---:|---:|
| Llama 3.1 | 60.0 | 90.8 | 59.0 |
| Llama 3.0 | 36.3 | 87.4 | 43.4 |
| GPT-4o | 80.4 | 96.0 | 74.2 |
| DeepSeek-Math | 52.4 | 87.6 | 55.4 |

Values are reported as accuracy percentages for Safe's Best-of-5 setting.

Important interpretation:

- Safe improves candidate selection by incorporating formal step-verification evidence.
- Formal verification makes some judgments more interpretable and checkable.
- The results demonstrate empirical improvements, not a universal mathematical guarantee that an LLM's complete natural-language solution is correct.

---

## 11. BASELINES

The paper compares Safe against approaches including:

- Best-of-N sampling.
- Process Reward Models (PRMs).
- Self-consistency.
- Alternative reasoning-trajectory scoring/selection approaches.
- Ablations of Safe's components.

The relevant comparison is not merely whether Lean can prove a theorem, but whether incorporating formal step verification improves the selection of mathematical reasoning trajectories.

---

## 12. CONTRIBUTION

The main contributions are:

1. **Safe:** A retrospective, step-aware formal verification framework for LLM mathematical reasoning.
2. **Lean 4 integration:** Formal verification of individual natural-language reasoning steps.
3. **Step-level evidence:** Verification states that provide interpretable evidence beyond opaque reward scores.
4. **Reasoning selection:** Integration of retrospective formal verification with prospective process-reward information.
5. **FormalStep:** A benchmark containing 30,809 formal statements for step-correctness theorem proving.
6. **Empirical evaluation:** Demonstration that formal-verification-informed selection can improve mathematical reasoning performance across models and datasets.

The central conceptual contribution is the integration of **proof-based verification of intermediate reasoning steps into inference-time mathematical reasoning**.

---

## 13. LIMITATIONS

### 13.1 Formalization errors

An LLM may formalize a natural-language step incorrectly.

Lean can verify the formalized proposition without detecting that the proposition differs from the original intended claim.

### 13.2 Incomplete formalization

Not every step of mathematical reasoning is easy to translate into Lean.

Some steps rely on:
- Informal intuition.
- Hidden assumptions.
- Contextual reasoning.
- Definitions not explicitly stated.
- Mathematical facts not available in the formal environment.

### 13.3 Verification coverage

A successfully verified step does not necessarily establish the correctness of the entire natural-language trajectory.

### 13.4 Dependency between steps

A reasoning step may depend on earlier claims.

Verifying isolated propositions may fail to capture whether all assumptions and dependencies are justified.

### 13.5 Computational cost

Formalization and theorem proving can introduce substantial inference-time computation.

### 13.6 Domain restriction

The framework is demonstrated primarily on mathematical reasoning.

Extending it to scientific reasoning, programming, or general factual claims requires additional formal representations.

### 13.7 Reliability versus correctness

The paper improves empirical mathematical reasoning performance but does not establish a general probability that an arbitrary LLM answer is correct.

---

## 14. MY QUESTIONS

1. What exactly are the four step-verification states used in Safe?
2. How does Safe generate the Lean formalization of a natural-language reasoning step?
3. What happens when a step cannot be formalized?
4. Does Safe verify each step independently or preserve dependencies?
5. How are the step-level verification states aggregated into a retrospective score?
6. What is the exact equation for combining retrospective verification and PRM scores?
7. How does Safe handle incorrect formalizations that Lean successfully proves?
8. How is the FormalStep dataset constructed?
9. What is the difference between theorem proving accuracy and natural-language step correctness?
10. Can Safe provide a soundness guarantee for the entire reasoning trajectory?
11. How much improvement comes from formal verification versus additional sampling?
12. Can the framework be adapted to mathematical proofs generated directly in Lean?
13. Can the verification process be calibrated probabilistically?
14. Can a formal proof certificate be attached to every mathematical claim in a response?

---

## 15. MY IDEAS

### Idea 1: Reliability of the formalization process

Safe verifies formal propositions, but the natural-language-to-Lean translation itself can introduce errors.

Research question:

Can we mathematically quantify the reliability of LLM-generated formalizations?

Potential approach:

- Generate multiple independent formalizations.
- Check semantic consistency between them.
- Compare their assumptions and conclusions.
- Measure disagreement.
- Estimate formalization uncertainty.

### Idea 2: End-to-end mathematical reliability

Define the reliability of a mathematical answer in terms of:

- Natural-language interpretation.
- Formalization correctness.
- Proof correctness.
- Assumption validity.
- Dependency correctness.
- Final answer extraction.

Research question:

Can we derive an end-to-end reliability framework from these components?

### Idea 3: Proof certificates for LLM responses

For each mathematical claim, produce:

- Natural-language claim.
- Formal proposition.
- Lean proof.
- Explicit assumptions.
- Verification result.
- Formalization confidence or validation evidence.

### Idea 4: Mathematical reliability score

Develop a reliability score that distinguishes:

- Formally verified claims.
- Unverified claims.
- Failed proof attempts.
- Incorrect formalizations.
- Unsupported assumptions.
- Contradictory steps.

Investigate whether such a score can be calibrated against actual correctness.

### Idea 5: Dependency-aware verification

Represent reasoning as a directed graph:

G = (V, E)

where:
- V contains mathematical claims or reasoning steps.
- E represents logical dependencies.

Research question:

Can dependency-aware formal verification detect errors that isolated step verification misses?

### Idea 6: Soundness of the overall pipeline

Lean proof checking is sound relative to its trusted kernel and assumptions.

However, the overall LLM pipeline includes informal components.

Research question:

What additional conditions are sufficient to obtain an end-to-end soundness guarantee?

### Idea 7: Formal verification of LLM reliability

A possible research direction:

**From Step-wise Formal Verification to End-to-End Reliability Guarantees for LLM Mathematical Reasoning**

Potential objective:

Develop a framework in which mathematical claims are translated into formal propositions, verified using Lean, and accompanied by explicit assumptions and reliability information. Like a system which take Question and answer of LLM as input, system generates the formal representation that canbe used by Lean to verify, and then Lean verifies the correctness of the LLM answer. That's how we will have a sysytem which tell that the LLM generated answers are right or not matghematically.

---

## 16. IMPORTANT REFERENCES

### Formal verification and theorem proving

- Lean 4
- Lean theorem proving ecosystem
- Interactive theorem proving
- Automated theorem proving
- Formalization of mathematics

### LLM mathematical reasoning

- Chain-of-Thought Prompting
- Self-Consistency Improves Chain of Thought Reasoning
- Let's Verify Step by Step
- Mathematical reasoning with LLMs
- FormalStep

### Reliability and verification

- Process Reward Models
- Best-of-N reasoning
- Formal verification of neural networks
- Calibration and uncertainty estimation
- Probabilistic verification
- Proof-carrying systems

### Foundational reference

Avigad et al. (2021) — work on formal proof and the role of proof assistants in mathematics.

---

## RESEARCH CONNECTION

**My research interest:** Mathematical foundations for the reliability of LLM-generated reasoning.

**Potential research question:**

Can we construct a mathematically rigorous framework to establish the reliability of an LLM-generated answer using formal verification, semantic fidelity, and explicit assumptions?

Safe is a foundational reference because it demonstrates how Lean 4 can be integrated into the retrospective verification of intermediate reasoning steps.

A potential extension is to move from:

**Verified formal statements**

to:

**Verified natural-language claims**

to:

**End-to-end reliability guarantees for mathematical reasoning**.
