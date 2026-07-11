# Quantum Claim Grading Rubric — v1

*A reusable evaluation asset distilled from the first five teardowns in `july_2026_teardowns`.*

## Purpose

Convert the recurring reviewer heuristics from five recent rewarded quantum teardowns into a single, durable grading rubric — a portfolio asset that scores *any* future quantum computing claim, rather than another one-off paper review. Each criterion below is scored 1–5 and carries a paired heuristic stated as a test you can run against a paper's own figures, tables, and appendices.

---

## Source Teardowns

The heuristics are extracted from these five entries (first five in the repository):

| # | Slug | Focus area |
|---|---|---|
| 1 | `teardown-distributed-vqls-2026-07-02` | HPC-distributed variational linear solver (D-VQLS) |
| 2 | `teardown-gkp-spam-below-1e-3-2026-07-09` | Bosonic GKP QEC, SPAM below 10⁻³ via postselection |
| 3 | `teardown-hamqasbench-2026-07-08` | Hamiltonian-informed benchmark for quantum architecture search |
| 4 | `teardown-hisep-q2-rvv-quantum-control-2026-07-09` | RISC-V vector ISA for quantum gate dispatch |
| 5 | `teardown-logical-qml-de-solver-2026-07-02` | Error-detected logical qubits on a neutral-atom QML solver |

---

## The Rubric

### 1. Hardware Realism

**Question:** Does the result run on — or credibly transfer to — real QPUs, or is it simulation- or extrapolation-bound?

**Heuristic (D-VQLS):** Separate "scaling of the fix" from "scaling of the capability." Near-ideal scaling of a *classical simulation* of quantum circuits carries almost no information about hardware feasibility. Downgrade when FPGA LUT/FF or statevector-simulator extrapolation is presented as physical-qubit scaling.

### 2. Resource Transparency

**Question:** Are the true costs — circuit count, circuit depth, state preparation, postselection survival — fully disclosed, including the worst case rather than only the favorable one?

**Heuristic (D-VQLS + GKP):** Trace every headline number back to its assumption class and check the worst case (e.g., `L = 4ⁿ` for dense matrices still costing ~10²⁵ circuits). For any postselected fidelity, recover the survival probability behind it (7×10⁻⁴ hiding ~9% total survival) and classify the postselection as repeat-until-success (acceptable for prep) or destructive (not algorithm-compatible mid-circuit).

### 3. Validation Strength

**Question:** Is the method validated *against* a classical/ideal reference (a correctness check) or shown to *beat* it (an advantage claim)?

**Heuristic (D-VQLS):** "Validated against classical" is not "beats classical." Fidelity computed against direct classical inversion on 4–10 qubit problems demonstrates implementation correctness where the classical solver is ground truth, not advantage. Demand a separate runtime/resource comparison at a scale where the classical method is actually stressed.

### 4. Structural vs. Scalar Correctness

**Question:** Could two qualitatively different solutions earn the same headline score? Is there a state-level check, not just a scalar metric?

**Heuristic (HamQASBench):** Ask whether the metric distinguishes right-structure from right-number. Energy accuracy and structural correctness decouple — a circuit can reach lower energy precisely by *not* representing the required correlations. Require a structural check (per-qubit entropy consistency, pairwise state fidelity) before believing the problem was "solved."

### 5. Benchmark Quality

**Question:** Was the evaluation set curated to expose (or hide) specific behavior, and is each hard-case axis isolated rather than confounded?

**Heuristic (HamQASBench + Logical QML):** Audit benchmark curation — the more hand-selected the set, the more the result reads as "mechanism exists" rather than "advantage generalizes," so demand at least one out-of-family or adversarial case. Separately, confirm the hard axis is isolated (scaling failure with entanglement held constant is credible *because* the axis was controlled; qubit count usually confounds correlation, symmetry, and search-space size).

### 6. Baseline & Best-Case Honesty

**Question:** Is the speedup relative to state-of-the-art or the authors' own prior work, and does the peak rest on an embarrassingly-parallel best case?

**Heuristic (HiSEP-Q):** Interrogate the baseline and the per-circuit mix behind the peak. A 2.52× peak on independent-gate layers (Bell-8, TwoLocal-16) sitting alongside ~1× on dependent chains (QAOA, QFT) means the geometric mean (1.32×) is the honest headline, and the benefit is conditional on circuit structure — especially when the baseline is the authors' own earlier design, not a commercial controller.

### 7. Scope Discipline

**Question:** Do the claimed capabilities match what was actually demonstrated end-to-end?

**Heuristic (HiSEP-Q + GKP + Logical QML):** Grade the weakest link and refuse category-inflation. Separate dispatch-width ("128 qubits per instruction") from end-to-end control-scale (waveform generation, calibration, readout). Separate prepared-state fidelity from injectable-resource fidelity for magic states. Treat "logical" as scoped to the code's detectable-error guarantee — non-Clifford gates via partially fault-tolerant ancilla tricks (proxy-phasing) break the label.

### 8. Industrial Relevance & Misinterpretation Risk

**Question:** Is the transferable operational finding separated from the domain-specific one, and what will readers predictably conclude wrongly?

**Heuristic (D-VQLS + Logical QML):** Prefer the transferable operational finding — one with a *measured threshold and a mechanistic cause* (the 4-vs-8 MPS-client contention cliff with its 11× `cudaMemGetInfo` latency explosion) — over the application-specific speedup. Put error mitigation where the classical loop *cannot* reach (coherent shape errors survive weight/offset rescaling; amplitude errors don't), and state the predictable overread explicitly ("fault tolerance already pays off for QML") so a strong score can't be quoted out of context.

---

## Scoring Sheet Template

| Criterion | Score (1–5) | Evidence / Notes |
|---|---|---|
| 1. Hardware realism |  |  |
| 2. Resource transparency |  |  |
| 3. Validation strength |  |  |
| 4. Structural vs. scalar correctness |  |  |
| 5. Benchmark quality |  |  |
| 6. Baseline & best-case honesty |  |  |
| 7. Scope discipline |  |  |
| 8. Industrial relevance & misinterpretation risk |  |  |

---

## How to Reuse This Rubric

This rubric turns recurring reviewer instincts into a repeatable audit. For any new quantum paper or vendor claim, score the eight dimensions and attach the paired heuristic as the concrete test to run against the paper's own figures and appendices. Because the criteria are grounded in failure modes already observed across HPC simulation, bosonic QEC, architecture search, control hardware, and logical QML, the rubric generalizes across subfields rather than to a single paper type.

It is also a living asset. Each new teardown either confirms an existing criterion or contributes a heuristic that sharpens one, so the rubric compounds as the portfolio grows. A claim that scores poorly on resource transparency, benchmark curation, or scope discipline should be flagged before its headline number propagates — which is precisely the point of holding a standing rubric instead of re-deriving the same skepticism paper by paper.
