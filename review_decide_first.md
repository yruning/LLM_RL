# Review: Dense Pseudo-Label Regression for Prospective Risk Forecasting in Large Language Models

## Verdict
- **Decision: REJECT**
- Status: SCORECARD
- Total score: 66.25 / 100
- Decisive reasons:
  - The paper's core technical contribution — a 2-parameter monotone label schedule (qend, Tinc) layered on top of an acknowledged weak-supervision principle (Sam & Kolter 2023) — does not constitute a non-trivial domain adaptation; the QA label construction is fully specified by two manually-tuned scalars with no principled derivation. (§3.2, p. 3; §4.4, p. 6)
  - The primary forecasting metrics (MLT, Flip Count) on which the headline claims rest are computed from 200 manually-annotated harmful-token onset positions with no inter-annotator agreement reported and no test-set confidence intervals, leaving the core result statistically unvalidated at the required scale for a venue acceptance. (§4.1.2, p. 4)

---

## One-paragraph summary

The paper addresses a real and underexplored problem: existing safety classifiers are trained on complete responses but must be deployed mid-generation on partial prefixes, creating a supervision mismatch. The proposed solution, Dense Pseudo-Label Regression (DPR), converts each sequence-level safety label into a dense prefix-level regression target using one of three monotone label families (Linear, Question-Aware, Onset), optionally augmented with smoothness and monotonicity regularization. Lightweight probes (linear, MLP, LSTM) trained on Qwen2.5-3B-Instruct hidden states with these targets achieve higher MCC (~0.5 vs ~0.15 for Binary, Table 1), better Mean Lead Time and lower Flip Count at fixed false-positive budgets (Figure 4), and lower calibration MSE vs Monte Carlo rollouts (Figure 5). The supervision-design insight (not architecture) is the bottleneck is well-supported across probe types. The work would matter more if the findings held across multiple model families and were validated on a larger annotated dataset; in its current form, the claimed generality exceeds what the evidence supports.

---

## Adversarial brief resolution

1. **Point**: The contribution is a routine application of weak supervision (Sam & Kolter 2023) to a new domain.
   - **Status**: Accepted as concern. The paper explicitly credits Sam & Kolter 2023 and Huang et al. 2022 as the foundational principle (p. 2). The QA label formulation is a specific instantiation with domain-appropriate priors, but does not require novel mathematical techniques. The label families are parameterized by 2 scalars (qend, Tinc) with no principled derivation and acknowledged OOD brittleness.

2. **Point**: The evaluation rests on 200 manually-annotated examples without IAA.
   - **Status**: Accepted as concern. §4.1.2 confirms 200+200 manual annotations. No IAA or test-set significance statistics are reported.

3. **Point** (counterverdict): DPR formalizes a useful evaluation framework (MLT, Flip Count, MC calibration) not present in prior work.
   - **Status**: Partially accepted. MLT/Flip Count are valuable new evaluation metrics for runtime safety monitoring. However, the metrics themselves are not validated (no IAA for th, no correlation to downstream intervention outcomes), and the paper's core claim is about DPR method superiority, not about the evaluation framework as a standalone contribution.

4. **Point** (counterverdict): Cross-architecture consistency (4 probe types) provides model-independent evidence.
   - **Status**: Refuted as "above-the-line" evidence. The cross-architecture validation is on the same model (Qwen2.5-3B-Instruct); it isolates probe architecture from supervision design but does not isolate the LLM family. The claim that supervision design is the bottleneck holds within Qwen2.5-3B, but whether this transfers to other model families is untested for the primary forecasting metrics.

---

## Fatal-flaw gates

No fatal gate failures. G4=CAUTION (bespoke metric construct validity concern for MLT/Flip Count).

---

## Scorecard

- A Problem & motivation: 3/4 → weighted 11.25/15
- B Novelty & insight: 2/4 → weighted 7.50/15
- C Technical quality: 3/4 → weighted 11.25/15
- D Evaluation rigor: 3/4 → weighted 18.75/25 *(G4=CAUTION cap)*
- E Causal understanding: 3/4 → weighted 7.50/10 *(G4=CAUTION cap)*
- F Robustness/generalization: 2/4 → weighted 5.00/10
- G Clarity/no fluff: 2/4 → weighted 2.50/5
- H Reproducibility/artifacts: 2/4 → weighted 2.50/5
- **Total: 66.25/100**

---

## Major concerns

**Concern 1 — Scale and statistical validity of the primary forecasting evaluation** **(unknown)**
- Origin: scorecard; Origin pointer: Dimension D, G4=CAUTION
- Claim affected: Claim 1 (MLT/Flip Count improvement)
- Evidence location: §4.1.2 (p. 4): "200 positive and 200 negative examples"; "we manually annotate the location of harmful content within the responses." No inter-annotator agreement reported. No confidence intervals on test-set comparisons in Table 1 or Figures 4/9.
- Why it matters: MLT and Flip Count are the paper's main differentiators from prior work. If the 200-example annotation is noisy or unreliable, the headline forecasting advantage is unverified. The gap on discrimination metrics (MCC: 0.509 vs 0.148, Table 1) is large enough to be robust at this sample size, but the trajectory metrics are not similarly validated.
- Concrete fix: Report inter-annotator agreement (IAA, e.g., Cohen's κ or Krippendorff's α) for the manual t_h annotations. Expand the trajectory evaluation set to ≥500 examples. Report bootstrap or permutation confidence intervals for the test-set MLT and Flip Count comparisons.

**Concern 2 — Single-model generalization gap** **(unknown)**
- Origin: scorecard; Origin pointer: Dimension F, Dimension B
- Claim affected: Core claim (headline generalizability of DPR as a training approach)
- Evidence location: §4.1.4 (p. 5): "We use Qwen2.5-3B-Instruct as the target model." Tables 2–3 (Appendix) show Gemma-2-2b results for structural loss only — not for MLT/Flip Count metrics.
- Why it matters: The claim that "supervision granularity is the bottleneck" (not architecture) is tested across 4 probe types on one model. Whether the label design benefit transfers to different base model geometries (e.g., Llama-3, Mistral) is unknown. Safety-relevant representations may be model-family-specific; a finding on Qwen2.5-3B is not necessarily transferable.
- Concrete fix: Replicate the primary MLT/Flip Count comparison (DPR-QA vs Binary vs Resp-only) on ≥1 additional base model. If the pattern holds, the generalizability claim is credible.

**Concern 3 — DPR label parameters are hand-tuned with acknowledged OOD brittleness** **(addressable if systematic)**
- Origin: decisive_factor_debate; Origin pointer: debate Judge ruling, bullet 1
- Claim affected: Claim 1 (DPR improves early-warning quality)
- Evidence location: §3.2 (p. 3): qend and Tinc defined as free parameters. §4.4 (p. 6): "the degraded performance of other DPR configurations is primarily due to a mismatch between the assumed tolerant reaction window during training and the actual onset of harmful content in the out-of-distribution evaluation dataset."
- Why it matters: If Tinc must be matched to the deployment distribution to avoid degradation, the method requires distributional assumptions about when harmful content appears — a practical constraint the paper does not address. The "mismatch" between training (Tinc=10) and BeaverTail (earlier onset) is a real brittleness. The fix for deployment would require knowing Tinc in advance.
- Concrete fix: Present a systematic Tinc sensitivity analysis showing how performance degrades as a function of Tinc mismatch. Propose a method for estimating Tinc from unlabeled deployment data, or acknowledge this as a deployment limitation.

**Concern 4 — Missing training details prevent full reproducibility** **(addressable)**
- Origin: scorecard; Origin pointer: Dimension H
- Claim affected: All claims (reproducibility)
- Evidence location: §4.1.4 (p. 5) omits optimizer, learning rate, batch size, and number of training epochs. Train/validation split sizes for PKU-Alignment + OpenAssistant are not reported.
- Why it matters: Without training details, replication is difficult. The variance over 5 seeds (Table 1) implies these are full training runs, but the training budget is unknown.
- Concrete fix: Add a reproducibility section with full training hyperparameters. Release code.

---

## Minor concerns

1. **Notation inconsistency in §4.1.3**: "(ii) DPR-QA(0.2): Question-Aware Label(qend = 0.4, Tinc = 10)" — qend=0.4 contradicts the naming convention where the parenthetical is qend. Appears to be a typo (should be qend=0.2). Origin: G clarity; pointer: §4.1.3 p. 4.

2. **Non-existent section reference**: §4.1.3 references "Sec 3.4" which does not exist in the paper. Origin: G clarity; pointer: §4.1.3 p. 4.

3. **Smoothness and monotonicity not ablated separately**: Table 1 and Figures 4/9 show "+Struct" as a combined regularization. The individual contribution of λ_smooth vs λ_mono is not isolate. Origin: scorecard E; pointer: Eq. 2 (p. 3).

4. **LlamaGuard-3-8B comparison paradigm mismatch**: The comparison to LlamaGuard-3-8B (text-based, 8B parameters) is presented alongside hidden-state probes but is not a fair head-to-head. The framing should clarify this is a motivating comparison, not a method comparison. Origin: scorecard D; pointer: §4.1.3 (p. 5).

5. **No performance-vs-prefix-length analysis**: The paper shows forecasting quality as a function of FP budget but not as a function of prefix length. For early-warning analysis, how MLT varies with response length would be informative. Origin: scorecard F; pointer: Figure 4.

6. **DPR-Onset not shown in Figure 4 (main body)**: Figure 4 shows DPR-QA(0.2) but not DPR-Onset(0.4) for the trajectory metrics. Figure 9 (appendix) includes all variants. Main body should show the full comparison. Origin: G clarity; pointer: Figure 4 p. 7.

---

## Required experiments (to flip verdict to ACCEPT)

1. **Multi-model replication** (Concern 2): Run the primary forecasting comparison (DPR-QA vs Binary vs Resp-only) on ≥1 additional base model (e.g., Llama-3-8B-Instruct or Mistral-7B-Instruct) including MLT/Flip Count metrics with annotated test sets. Expected information gain: HIGH. If the pattern holds on a second model, the generalizability of the supervision-design insight is established and the novelty claim is considerably stronger.

2. **Enlarged and IAA-validated trajectory evaluation** (Concern 1): Expand the manually-annotated MLT/Flip Count test set to ≥500 examples (ideally 1000) with at least 2 annotators and reported κ. Report bootstrap CIs for the MLT comparisons. Expected information gain: HIGH. If the MLT advantage survives at scale with high IAA, the core forecasting claim is validated.

---

## Optional improvements

1. A systematic Tinc sensitivity analysis would help practitioners choose label parameters for their deployment distribution.
2. Ablating smoothness and monotonicity regularization separately would give cleaner insight into their individual contributions.
3. Analyzing performance vs. prefix length / response position would provide richer understanding of when DPR is most valuable.
4. Code release would substantially improve reproducibility and community adoption.
5. The evaluation framework (MLT/Flip Count + Monte Carlo calibration) is independently valuable; a brief discussion of it as a standalone contribution to the runtime safety evaluation toolkit would strengthen the paper's framing.

---

## "If I were the author, I would do this next"

- Run the core DPR-QA vs Binary comparison on Llama-3-8B-Instruct. If the supervision-design bottleneck holds on a second major model family, that single replication substantially raises the contribution above the line.
- Expand the t_h annotation to 1000 examples with IAA, report bootstrap CIs for MLT comparisons. The current 200-example annotation is the weakest link in an otherwise well-designed study.
- Propose a method for estimating Tinc from unlabeled deployment data (e.g., using the trained probe itself to estimate onset distribution), removing the key brittleness identified in OOD evaluation.
- Release code. The evaluation framework (MLT, Flip Count, Monte Carlo calibration pipeline) is a more lasting contribution than the specific label designs; making it easy to use is the fastest path to community adoption.
- Consider separating the evaluation framework contribution from the DPR method contribution. The framework itself deserves more space; the two together dilute each other.

---

## Remaining uncertainty

The core empirical question is whether DPR-QA's advantage over binary prefix training is a model-specific artifact of Qwen2.5-3B's internal geometry, or a general property of supervision design for LLM safety probing. The paper provides no evidence on this, and the failure of DPR-Onset/QA(0.2) on BeaverTail suggests sensitivity to distributional assumptions. If the pattern holds on a second model with a larger annotated test set, the contribution would be above the line. If it does not hold, the paper's contribution reduces to the evaluation framework (MLT/Flip Count/MC calibration) plus a dataset-specific observation.
