# Quantum Paper Teardown

## Thesis
A single-mode GKP (grid-state) qubit in a superconducting cavity can reach combined state-preparation-and-measurement (SPAM) error below 10⁻³ by replacing gate-based state prep with postselected stabilization and replacing single-shot readout with repeated finite-energy measurement — without degrading autonomous QEC.

## Verdict
It succeeds and it matters: a ~100× SPAM reduction that puts a bosonic logical qubit on par with transmon SPAM is real progress, but the headline number is bought with heavy postselection (≈9% total survival), which the paper is honest about but which limits how the result should be read.

## Tags
`error-correction` `bosonic` `GKP` `superconducting` `SPAM` `postselection` `near-term`

---

## Paper Info
- **arXiv link:** https://arxiv.org/pdf/2607.06718
- **Date:** 7 July 2026 (v1)
- **Title:** Quantum error correction of a grid-state qubit with state preparation and measurement errors below 10⁻³
- **Authors:** Sara Turcotte, Lucas St-Jean, Amélie L. Pessonneaux, Ross Shillito, Bohdan Kulchytskyy, Eliott Ouellet, Jean Olivier Simoneau, Florian Hopfmueller, Matthew Hamer, Pascal Lemieux, Dany Lachance-Quirion, Baptiste Royer, Nicholas E. Frattini (Nord Quantique; Université de Sherbrooke)

---

## What the Paper Claims
The authors report combined SPAM error below 10⁻³ (7(7)×10⁻⁴ averaged over the six cardinal states) for a single-mode grid-state qubit — roughly two orders of magnitude better than prior bosonic-platform state of the art, and comparable to transmon SPAM. They achieve this with two composable protocols: repeat-until-success state preparation via postselected sBs stabilization (starting from vacuum or a 9 dB squeezed state), and a measurement protocol that repeats finite-energy Pauli measurements within a single shot under an all-agree postselection policy. They further show H-type magic states prepared from vacuum at 8(5)×10⁻³ SPAM, and that layering these protocols in front of autonomous QEC leaves the logical error per round essentially unchanged (0.0081(2) vs 0.0085(2)).

## Mechanism in Plain Language
A grid state stores one qubit inside an oscillator's large Hilbert space as a lattice of peaks in phase space; errors like photon loss nudge those peaks, and "stabilization" (the sBs protocol) is a repeated push that snaps them back onto the lattice. Rather than building the target state with a bespoke sequence of conditional-displacement gates (ECD) — which are vulnerable to auxiliary-qubit errors mid-gate — the authors just run stabilization from vacuum (or a squeezed state) and, after each round, measure the auxiliary qubit and **throw away any run where it wasn't found in ground state**. Keeping only "clean" histories is repeat-until-success: it costs attempts, not fidelity. On the readout side, a single finite-energy measurement is limited by the auxiliary qubit's ~95% readout visibility, so they repeat the measurement several times in one shot and keep only shots where every repetition agrees, which suppresses both readout errors and photon-loss-induced errors. Pre-squeezing helps because a squeezed state is already an eigenstate of one code stabilizer, so fewer rounds — and thus fewer postselection rejections — are needed to converge, raising survival probability for a given fidelity.

## What Matters Practically
SPAM has been the quiet tax on bosonic-qubit demonstrations: you can beat break-even on logical lifetime and still have your reported logical fidelities dominated by how badly you initialize and read out. This work shows SPAM is not a fundamental floor of the platform — it's largely an auxiliary-qubit-readout artifact that repetition + postselection can strip away, and it does so without touching the autonomous QEC loop, so it's a drop-in front/back end rather than a redesign. For system architects, the update is that bosonic logical-qubit benchmarks should now separate "SPAM-limited" from "coherence-limited" performance, and that vacuum is a surprisingly good resource for both cardinal and magic states. The magic-state result (8×10⁻³ from vacuum, no feedback) is the more strategically interesting number because non-Clifford resource states are the usual bottleneck for universality.

## Likely Misinterpretation
The dangerous misread is treating 7×10⁻⁴ as a deterministic SPAM figure comparable to a transmon's — it is a **postselected** number with ~9% total survival (24% prep × 39% measurement), so it describes the quality of the states you keep, not the rate at which you get them. In a real algorithm, measurement-stage postselection cannot be repeat-until-success (you can't rerun a mid-circuit measurement whose outcome you needed), so the measurement half of this SPAM budget does not straightforwardly survive into fault-tolerant operation — the authors say this explicitly, and Appendix D is entirely about the survival/fidelity tradeoff. The second overread is the large error bar: 7(7)×10⁻⁴ has a fractional uncertainty near 100%, so "below 10⁻³" is the defensible claim, not "7×10⁻⁴" as a precise point. Finally, don't generalize the magic-state fidelity to logical magic states usable in a distillation pipeline — it's a prepared-state SPAM fidelity, not a certified injectable resource.

## Bottom Line
Treat this as proof that bosonic-qubit SPAM is an engineering problem, not a physics floor — but when comparing to transmons or feeding these numbers into resource estimates, always divide the fidelity by its survival probability and ask whether the postselection is repeat-until-success (prep, fine) or destructive (measurement, not fine).

---

## Scores

| Dimension | Score (1–5) | Rationale |
|---|---|---|
| Technical significance | 4 | A genuine ~100× SPAM improvement on a hard, previously-limiting axis, cleanly decomposed into two reusable protocols and shown compatible with autonomous QEC. Held below 5 because the gains lean on postselection rather than a new deterministic capability. |
| Industrial relevance | 4 | SPAM was a real blocker for bosonic-platform credibility; removing it (even conditionally) plus vacuum-sourced magic states matters for anyone building GKP-based FTQC. Capped because measurement-side survival (~39%) is not algorithm-compatible as-is. |
| Misinterpretation risk | 4 | High: the postselected nature, ~9% survival, and ~100% error bar on the headline number are all easy to drop when quoting "SPAM below 10⁻³," and the measurement-side result is especially prone to being cited out of context. |

---

## Reusable QEC Evaluation Heuristics

Transferable reviewer checks distilled from this paper, intended to be reapplied to future QEC / bosonic-qubit papers:

1. **Always recover the survival probability behind any postselected fidelity, and classify the postselection.** A SPAM or logical-fidelity figure is meaningless without its acceptance rate. Divide the quoted fidelity's implied error by the survival fraction to get a rough "attempts-weighted" cost, then ask whether the postselection is *repeat-until-success* (acceptable for state prep — you just retry) or *destructive/mid-algorithm* (measurement selection you cannot rerun). Numbers from the second category should never be quoted alongside deterministic-device benchmarks. Here: 7×10⁻⁴ hides ~9% total survival, and the measurement half (39%) is destructive.

2. **Attribute the error to the limiting subsystem before crediting the encoding.** When a bosonic/logical result improves, check whether the gain came from the code or from an auxiliary component (readout visibility, reset, ancilla coherence). This paper's single-shot SPAM was *auxiliary-qubit-readout-limited* at ~95% visibility — the improvement is a readout-repetition trick, not a code advance. Reviewer check: if the paper's "limit" line (e.g., a dashed auxiliary-readout bound) tracks the data, the result is characterizing peripheral hardware, not the logical qubit.

3. **Check that SPAM-improvement claims are demonstrated *not to degrade* the QEC loop, using a matched-conditions comparison.** A prep/readout upgrade is only useful if it's compatible with continuous error correction. Demand a side-by-side logical-error-per-round measurement with and without the new protocol under identical (ideally restless) conditions. Here the 0.0081(2) vs 0.0085(2) comparison is exactly the right control and is what makes the compatibility claim credible rather than asserted.

4. **Separate "prepared-state fidelity" from "injectable resource fidelity" for magic states.** A high-fidelity magic-state *preparation* number (8×10⁻³ from vacuum here) is a tomographic SPAM quantity, not evidence the state is usable in a distillation or gate-teleportation pipeline. Reviewer check: was the state consumed in a non-Clifford operation and the *output* verified, or only the input reconstructed? If only the latter, treat it as a resource-availability result, not a universality result.

5. **Interrogate error bars on headline sub-10⁻³ numbers.** Rare-event error rates estimated from finite shots after aggressive postselection carry large relative uncertainty. When the reported value is X(X)×10⁻ⁿ with ~100% fractional error (as with 7(7)×10⁻⁴), the load-bearing claim is the *order of magnitude / threshold crossed*, not the central value — cite it that way.

---

## Verification
- **Public post / GitHub URL:** [to be published]
- **Commit / post date:** 2026-07-09
