# Quantum Paper Teardown

## Thesis
Richardson Zero-Noise Extrapolation stops being an extrapolation and becomes a fixed multiplication of a single noisy data point once noise amplification pushes the amplified expectation values to the observable's floor — and that degenerate arithmetic manufactures an "improvement" that looks like successful error mitigation but is not.

## Verdict
It succeeds, and it matters: the mechanism is proven in closed form, confirmed on real hardware with ordinary faithful folding, and comes with a zero-cost diagnostic that separates real mitigation from arithmetic — which means a large fraction of published ZNE benchmark numbers on non-trivial-depth circuits are now suspect until they report the check.

## Tags
`error-mitigation` `benchmarking` `verification` `near-term` `hardware-realism` `negative-control`

---

## Paper Info
- **arXiv link:** https://arxiv.org/pdf/2607.09360 (arXiv:2607.09360v1 [quant-ph])
- **Date:** 10 July 2026
- **Title:** Benchmarking Error Mitigation: Artefactual Improvements in Zero-Noise Extrapolation
- **Authors:** Dominik Köster, Wolfgang Mauerer (TH Regensburg; Siemens AG Foundational Technology)

---

## What the Paper Claims
Richardson ZNE fits a polynomial through expectation values measured at amplified noise levels λ and extrapolates back to λ=0. The authors claim that when the amplified points E(λ>1) collapse to the observable's floor — which happens fast on real hardware — the extrapolation degenerates into a deterministic rescaling of the single λ=1 measurement, with a coefficient fixed entirely by the choice of scale factors. For the standard {1,3,5} set this is Ê(0) = (15/8)E(λ₁) − (7/8)f, meaning the "mitigated" value is just the raw noisy value multiplied by 1.875, regardless of whether the noise amplification did anything at all. They confirm this on the 54-qubit IQM Euro-Q-Exa machine (21% overshoot of the ideal value at Trotter depth 3), and falsify the pipeline with a matched-cost "garbage folding" negative control — inserted folds that do *not* compose to the identity, matched in gate count and CZ count — which reports a *larger* apparent improvement (ρ = 2.77 ± 0.25) than genuine folding (ρ = 0.99 ± 0.13).

## Mechanism in Plain Language
Richardson extrapolation is a weighted sum of your measurements: Ê(0) = Σ cₖE(λₖ), with the weights cₖ determined solely by which noise scale factors you picked, and constrained to sum to 1. For {1,3,5} the weights are c₁=15/8, c₂=−5/4, c₃=3/8. Now suppose the amplified circuits are so noisy that E(λ₃) and E(λ₅) have both decayed to the floor value f (0 for a parity observable, 1/2ᴺ for a probability). Substitute f into the sum: everything past the first term collapses into (1−c₁)f, and you are left with Ê(0) = c₁E(λ₁) + (1−c₁)f — a straight-line function of one number. Because c₁ > 1 by construction, the output is always the raw value scaled *up*, so an "improvement" is generated mechanically no matter what the physics did. The extrapolation is no longer reading a decay curve; it is reading the same noisy point it started with and applying a fixed gain, and this holds for any linear extrapolator with c₁ > 1, not just Richardson or this particular scale-factor set.

## What Matters Practically
The failure onset is depth- and hardware-dependent, and on a current superconducting device it arrives at **Trotter depth 3 on a 4-qubit circuit** — i.e., almost immediately for anything of practical interest. Anyone benchmarking a hybrid workflow with ZNE in the loop is likely reporting a number that is partly a coefficient artefact, and the magnitude of the reported improvement is *anti*-informative: garbage folding produced a bigger improvement than real folding. The paper's second contribution is the fix, and it is free: apply the same Richardson coefficients to the per-basis-state probabilities rather than the scalar expectation, and count how many come out negative. The negative-probability weight W_neg = Σ|P̂(s)| over negative states separated valid folding (0.02–0.04) from garbage folding (0.74 ± 0.06) with **AUC = 1.0 across 405 hardware runs** — no ground truth required, unlike the recovery ratio ρ, which needs the ideal value and diverges when E_ideal ≈ E(λ₁).

## Likely Misinterpretation
The loudest wrong read will be **"ZNE is broken"** — it isn't. The authors run the control: where signal is retained, genuine folding lowers mean-squared error against the raw baseline by more than an order of magnitude, and an identity-fold control (extra gates composing to identity, count-matched) neither overshoots nor triggers the artefact (ρ_id = 0.74 ± 0.09). The failure is specific to the signal-destroyed regime, not to the technique. The second, subtler misread is **conflating this with the known ill-conditioning problem** of closely spaced scale factors. Those are orthogonal: ill-conditioning is driven by a large coefficient sum Σ|cᵢ| (the Kandala set {1, 1.1, 1.25, 1.5} gives Σ|cᵢ| = 681, a 194× larger variance bound than {1,3,5}'s 3.5) and can be fixed by choosing wider spacing — whereas *this* failure is driven by proximity to the floor and strikes the variance-optimal {1,3,5} set with full force. Picking good scale factors does not protect you. The third misread is treating the physical-range constraint as sufficient: the paper shows a d=5 case where the pure rescaling lands *below* the ideal value and inside the physical range while being entirely artefactual.

## Bottom Line
Stop treating the size of a ZNE improvement as evidence of its correctness — it is evidence of nothing, and can be evidence of the opposite. If you run ZNE, report whether E(λ>1) still retains signal above the floor, report W_neg alongside Ê(0), and run a matched-cost garbage-fold negative control at least once per circuit family; if your pipeline can't distinguish a fold that preserves the unitary from one that destroys it, your improvement number is arithmetic, not physics.

---

## Scores

| Dimension | Score (1–5) | Rationale |
|---|---|---|
| Technical significance | 4 | The closed form in Eq. (2) is almost trivial once written down — which is precisely the indictment. Not a deep theoretical result, but it identifies a real, live failure mode in a technique in wide implicit use, and pairs it with a discriminating diagnostic that costs nothing. Docked one point because the core algebra is elementary and the underlying "post-processing can manufacture improvement" insight was already flagged for PEC by Govia et al. (the horoscope effect); this is a rigorous port to ZNE, not a new idea. |
| Industrial relevance | 4 | Directly hits the credibility of NISQ-era benchmark claims that industrial users rely on to size quantum pilots. The regime boundary is not far away — it's at depth 3 on 4 qubits on current superconducting hardware. Anyone building a hybrid workflow with QEM in the loop must re-audit. Not a 5 only because it changes how you *report* and *validate*, not what you can *compute*. |
| Misinterpretation risk | 4 | High. The result is easy to over-generalize into "ZNE is fake," and the paper's own framing (garbage folding beating genuine folding) is quotable in a way that invites exactly that. It is also easy to conflate with the well-known ill-conditioning story and conclude, wrongly, that the standard {1,3,5} set is a defense. The authors explicitly pre-empt both, but the abstract travels further than Section V-C. |

---

## Reusable Grading Heuristics / Reviewer Checks

Three transferable checks extracted from this paper, applicable well beyond ZNE:

**H1 — The Degenerate-Estimator Check.** *Before believing any post-processing estimator, ask what it collapses to when its inputs go uninformative.* Any linear estimator Σcₖxₖ with Σcₖ = 1 and c₁ > 1 becomes a fixed gain on x₁ once x₂...x_K saturate to a constant. The check is mechanical: substitute the floor/saturation value into every input except the first and see whether a "result" still emerges. If it does, and if that result is systematically biased in the direction of the desired outcome, the estimator can manufacture its own conclusion. This applies to any weighted-correction scheme — QEM, Richardson/Romberg-style extrapolation in classical numerics, any bias-correction pipeline with signed weights.

**H2 — The Matched-Cost Negative Control.** *A benchmark claim is not credible without a control that is identical in cost and structurally identical in every respect except the one that is supposed to carry the mechanism.* Garbage folding is a model of how to build one: same gate count, same CZ count, same shot budget, same qubit chain, same transpilation flags — differing *only* in whether the inserted folds compose to the identity. If the control reports a larger improvement than the real thing, the pipeline is measuring its own arithmetic. **Corollary heuristic: the magnitude of a reported improvement is not evidence of its correctness, and a suspiciously large improvement should raise suspicion, not confidence.** Note the paper's careful *second* control (identity fold) that isolates gate count as a confounder — a single negative control is often not enough to attribute causation.

**H3 — Prefer Ground-Truth-Free Validity Flags Over Ground-Truth-Dependent Scores.** *When a quality metric requires the ideal value, it is unavailable exactly where you most need it — at scale, where classical simulation fails.* The recovery ratio ρ needs E_ideal and diverges pathologically as E_ideal → E(λ₁) (producing the paper's absurd ρ = 16, ρ = 125 values). W_neg needs nothing but data the benchmark already holds, stays bounded, and separated valid from invalid runs with perfect discrimination. **Reviewer check: does the paper's headline metric require a ground truth the deployed system won't have?** If yes, demand a companion self-consistency / internal-validity check — some quantity that must hold for a physically valid result and is computable from the raw output alone (negativity of a probability estimate, violation of a normalization or positivity constraint, a conservation law). Prefer bounded, regime-invariant flags: W_neg saturates against a ceiling set by the scale factors alone (CV < 8% across regimes), which is what makes it usable as a binary gate rather than a fuzzy score.

---

## Verification
- **Public post / GitHub URL:** [placeholder — not yet published]
- **Commit / post date:** 2026-07-13
