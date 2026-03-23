# Review — Dense Pseudo-Label Regression for Prospective Risk Forecasting in Large Language Models

## Verdict
- **Decision: REJECT**
- **Status: SCORECARD**
- **Total score: 65.0 / 100**
- **Decisive reasons:**
  1. **Limited novelty (B=2)**: The core contribution — converting sequence-level labels into dense regression targets with temporal priors — is a routine application of weak supervision principles (Sam & Kolter, 2023; Huang et al., 2022) to safety probing. The label families (Linear, QA, Onset) are simple parametric curves; the structural regularization (smoothness + monotonicity) is standard. The supervision-mismatch insight is valid but straightforward.
  2. **Narrow experimental scope (F=2)**: Experiments limited to two small models (Qwen2.5-3B, Gemma-2-2B) with 400-example test sets. Safety monitoring matters most for large deployed models; the absence of any evaluation on models ≥7B is a significant gap. No sensitivity analysis on key hyperparameters (T_inc, λ values).

## One-paragraph summary
The paper addresses a real supervision mismatch: runtime safety monitors need per-token risk scores, but available datasets only have sequence-level labels. The proposed Dense Pseudo-Label Regression (DPR) converts binary sequence labels into structured prefix-level regression targets — using linear, question-aware, or onset-based label families — that encode temporal inductive biases (smooth evolution, monotonic increase for unsafe continuations). Training lightweight probes on hidden states of Qwen2.5-3B-Instruct, DPR-QA variants consistently outperform binary prefix classification and response-only augmentation on both discrimination metrics (MCC improved from 0.148 to 0.509 on MLP_stan, Table 1) and forecasting-quality metrics (higher MLT, lower flip count, Figure 4). The work is technically competent and well-evaluated within its scope, but the novelty of the training signal design is limited and the experimental scope is too narrow for confidence in the approach's general utility.

## Adversarial brief resolution

1. **Routine weak supervision application (High)**
   - Status: **Accepted as major concern**
   - Evidence: The domain-removed contribution (structured pseudo-labels + temporal regularization from sequence-level labels) exists in adjacent fields (Sam & Kolter, 2023; Huang et al., 2022, both cited). The supervision-mismatch insight, while valid, is straightforward once stated — a competent practitioner would naturally consider smooth labels over binary replication. The forecasting-specific evaluation metrics (MLT, flip count) are a useful but modest methodological contribution.
   - Impact: Caps B at 2.

2. **Narrow scope: small models, small test sets (High)**
   - Status: **Accepted as major concern**
   - Evidence: Main experiments on Qwen2.5-3B only (p. 5, line 243); appendix adds Gemma-2-2B (Tables 2–3, pp. 12–13). Both are ~3B. Test sets are 200+200 (p. 4, line 195). No evaluation on models ≥7B where safety monitoring is most practically relevant.
   - Impact: Caps F at 2; contributes to D not reaching 4.

3. **No runtime evaluation (Medium)**
   - Status: **Accepted as minor concern**
   - Evidence: Paper frames contribution for "runtime monitoring" and "practical intervention" (abstract) but evaluates only offline metrics. No latency, no integration into generation pipeline, no intervention effectiveness measurement.
   - Rationale: A methods paper proposing a training signal is not expected to demonstrate full deployment, but the gap between framing and evaluation should be acknowledged more explicitly.

4. **Missing reproducibility details (Medium)**
   - Status: **Accepted as minor concern**
   - Evidence: Optimizer, learning rate, batch size, epochs, hardware, MC rollout parameters all unspecified.

5. **Calibration oracle approximation (Low)**
   - Status: **Accepted as minor concern**
   - Evidence: LlamaGuard-3 on complete rollouts is reasonable; this is one of several evaluation axes and the main discrimination results don't depend on it.

## Scorecard
| Dimension | Score | Weighted |
|-----------|-------|----------|
| A Problem & motivation | 3/4 | 11.25/15 |
| B Novelty & insight | 2/4 | 7.50/15 |
| C Technical quality | 3/4 | 11.25/15 |
| D Evaluation rigor | 3/4 | 18.75/25 |
| E Causal understanding | 2/4 | 5.00/10 |
| F Robustness/generalization | 2/4 | 5.00/10 |
| G Clarity/no fluff | 3/4 | 3.75/5 |
| H Reproducibility/artifacts | 2/4 | 2.50/5 |
| **Total** | | **65.00/100** |

## Major concerns (3)

1. **Core technique is a routine transfer of weak supervision**
   - Origin: adversarial brief + scorecard
   - Origin pointer: adversarial point #1, B-dimension scoring
   - Claim affected: DPR is a novel contribution to safety monitoring
   - Evidence: Sam & Kolter (2023) and Huang et al. (2022), both cited by the paper, establish the paradigm of structured losses/pseudo-targets from coarse labels. The specific application to safety probing is new, but the intellectual barrier is low.
   - Why it matters: Novelty below venue bar for ICML. "First application of X to Y" is not sufficient novelty.
   - Concrete fix: Demonstrate a non-trivial technical challenge in adapting weak supervision to this setting (e.g., show that naive label smoothing fails and requires the specific question-aware design), or provide deeper mechanistic insight into what the probes learn.
   - Classification: **(addressable)** — deeper ablation and mechanistic analysis could strengthen the novelty case.

2. **Evaluation limited to ~3B models with small test sets**
   - Origin: adversarial brief + scorecard
   - Origin pointer: adversarial point #2, F-dimension scoring
   - Claim affected: All claims — generality is uncertain
   - Evidence: Only Qwen2.5-3B (main) and Gemma-2-2B (appendix). Test sets: 200+200 examples (p. 4, line 195).
   - Why it matters: Safety monitoring matters most for large deployed models. Hidden state representations differ significantly across model scales; results at 3B may not transfer. Small test sets limit statistical power.
   - Concrete fix: Evaluate on at least one 7B+ model. Increase test set size or report confidence intervals.
   - Classification: **(unknown)** — no indirect evidence in the paper suggests results would transfer to larger models.

3. **Limited causal understanding of the method's mechanisms**
   - Origin: scorecard
   - Origin pointer: E-dimension scoring
   - Claim affected: Understanding of why DPR works
   - Evidence: The paper compares 6 DPR variants but doesn't ablate smoothness vs. monotonicity losses individually (only +Struct = both). No analysis of what the probes learn to detect in hidden states (just that middle layers are more informative, Table 4). No failure case analysis.
   - Why it matters: Without deeper understanding, it's unclear when DPR would fail or how to improve it beyond the tested configurations.
   - Concrete fix: (1) Ablate L_smooth and L_mono individually. (2) Analyze probe activations on specific examples to understand what signals the probe uses. (3) Present failure cases with analysis.
   - Classification: **(addressable)** — these experiments are straightforward to run.

## Minor concerns (5)

1. **No runtime latency measurements despite deployment framing** — Origin: adversarial brief. Origin pointer: adversarial point #3. The abstract and introduction frame DPR for "practical intervention" during generation but no latency overhead is reported.

2. **Missing optimizer/training details** — Origin: scorecard H-dimension. Optimizer, learning rate, batch size, epochs unspecified; reduces reproducibility.

3. **Fixed T_inc = 10 without sensitivity analysis** — Origin: scorecard. The mismatch between fixed T_inc and variable actual onset is acknowledged (p. 6, lines 317–320) but no sensitivity analysis explores the effect of different T_inc values.

4. **Title says "Large Language Models" but only tests ~3B models** — Origin: scorecard G7. Minor framing concern; both tested models are small by current standards.

5. **Absolute task performance remains low (best MCC ~0.52)** — Origin: adversarial brief. Origin pointer: adversarial point #5. Even with DPR, the forecasting task is far from solved. The paper does not discuss deployment-readiness thresholds.

6. **MC rollout calibration oracle may be unreliable on short continuations** — Origin: adversarial brief. Origin pointer: adversarial point #6. LlamaGuard-3 achieves MCC = 0.012 on prefix classification (Table 1), and MC rollouts from intermediate prefixes may produce short responses where LlamaGuard-3 is less reliable than on full responses.

7. **Typo "shos" → "shows" on p. 2, line 55** — Origin: QC pass. Minor.

## Required experiments (to flip verdict to ACCEPT)

1. **Evaluation on a 7B+ model** (highest information gain): Would directly address the narrow-scope concern and test whether hidden-state safety signals generalize across model scales. Expected effort: moderate.

2. **Individual ablation of smoothness and monotonicity losses**: Currently only tested jointly as +Struct. Separating them would clarify which inductive bias contributes and when. Expected effort: low.

3. **Comparison to a stronger probing baseline**: Test against a probe trained with simple label smoothing (no temporal structure) to isolate the contribution of the specific pseudo-label families vs. generic soft targets. Expected effort: low.

## Optional improvements

- Sensitivity analysis for T_inc and λ hyperparameters across a range of values.
- Analysis of probe predictions on specific examples (success and failure cases) to build mechanistic intuition.
- Runtime latency measurements for the probe during generation.
- Larger test sets for more statistically robust evaluation.
- Report optimizer, learning rate, batch size, and compute budget for reproducibility.

## "If I were the author, I would do this next"

1. **Scale to 7B+ models** — Test on Llama-3-8B, Qwen2.5-7B, or similar. This is the single most impactful experiment for demonstrating practical relevance.

2. **Ablate smoothness and monotonicity losses individually** — Show which prior matters when, and whether they interact synergistically.

3. **Compare against simple label smoothing** — Use a generic smoothed version of the binary label (e.g., Gaussian kernel smoothing) as a baseline to isolate the contribution of the specific QA/Onset designs vs. mere softening.

4. **Mechanistic analysis of probe representations** — Use probing or visualization techniques to understand what safety-relevant features the DPR-trained probes learn to read from hidden states.

5. **Integrate into a generation pipeline** — Demonstrate the monitor in a real intervention setting: show that DPR-triggered early stopping actually prevents harmful completions with acceptable false-positive rates.

6. **Adaptive T_inc** — Instead of fixing T_inc = 10 for all examples, explore methods to estimate the harmful content onset from the data or allow the model to learn it.

## Remaining uncertainty

The key uncertainty is whether DPR's improvements generalize beyond small (~3B) models. Hidden state representations change significantly with model scale, and safety alignment techniques (RLHF, DPO) reshape internal representations — the DPO analysis in Table 5 already shows that DPO fine-tuning doesn't improve (and slightly degrades) probe performance, suggesting the relationship between hidden states and safety signals is model-dependent. Without evaluation on larger, more widely deployed models, the practical impact of DPR remains uncertain.
