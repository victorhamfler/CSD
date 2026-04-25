# SKILL.md — Constraint Sensitivity Diagnostic (CSD)

## Purpose

Use this skill to apply the **Constraint Sensitivity Diagnostic (CSD)** to scientific, engineering, machine-learning, simulation, or data-analysis workflows where observed changes may be caused by imposed constraints rather than by intrinsic structure.

CSD is a **constraint-aware diagnostic framework**. It measures how an observable changes under a controlled constraint and normalizes that change by the natural geometry of the constraint. Its purpose is to distinguish:

- **Inevitable constraint-induced distortion**
- **Residual intrinsic structure**
- **False thresholds, elbows, or phase-like artifacts caused by scaling**

CSD is not a correction method. It does not recover lost information. It is a methodological audit layer used before interpreting constrained data.

---

## Core question

Use CSD when the central question is:

> Is the observed effect intrinsic to the system, or is it mostly caused by the geometry, information loss, or stochasticity of the imposed constraint?

Examples of constraints:

- Rank truncation
- Projection
- Dimensional bottlenecks
- Reduced basis size
- Spectral cutoffs
- Dataset-size limits
- Label noise
- Context-window truncation
- Random projection
- Graph or manifold compression
- Topological discretization
- Low-rank model approximation
- Reduced-order simulation

---

## When to use CSD

Apply this skill when analyzing systems where an observable is measured across a family of constraints.

Good use cases:

- Quantum simulations under Hilbert-space truncation
- Tensor-network bond dimension studies
- Truncated QFT / TCSA calculations
- Open-system Liouvillian truncation
- Reduced-order numerical solvers
- PCA, autoencoder, or embedding bottlenecks
- Graph spectral embeddings
- Model compression
- Representation drift in ML systems
- Dataset scarcity or noisy-label experiments
- Robustness claims involving “critical dimension,” “collapse,” or “phase-like” behavior
- Agent-memory or AI-system diagnostics where memory geometry changes under compression or update rules

Avoid using CSD when:

- There is no tunable constraint parameter
- There is no meaningful reference state
- The observable is not comparable across constraint levels
- The goal is direct correction or reconstruction rather than diagnosis
- A topological invariant is being analyzed and normalization would destroy the signal

---

## Basic formulation

Let:

- `lambda` be the constraint strength or constraint parameter
- `X_lambda` be the observable under constraint `lambda`
- `X_ref` be the least-constrained or reference observable
- `Delta(lambda)` be the raw deviation
- `G(lambda)` be the constraint-geometry normalization

The raw deviation is:

```text
Delta(lambda) = || X_lambda - X_ref ||
```

The CSD signal is:

```text
CSD(lambda) = Delta(lambda) / G(lambda)
```

The key step is choosing `G(lambda)` according to the **constraint type**, not according to what produces the prettiest plot.

---

## Minimal algorithm

1. **Define the reference**
   - Choose the least-constrained, highest-resolution, largest-rank, cleanest, or best-validated state as `X_ref`.
   - Record exactly how the reference was produced.

2. **Define the constraint family**
   - Examples: rank `k`, bottleneck dimension `d`, sample count `n`, noise level `eta`, projection seed, graph eigenmode count, context length.
   - The constraint must be varied systematically.

3. **Compute the constrained observable**
   - For each constraint value, compute `X_lambda`.
   - Use the same observable definition at every constraint level.

4. **Measure raw deviation**
   - Choose a norm or discrepancy appropriate for the observable.
   - Common choices: Frobenius norm, operator norm, vector norm, reconstruction error, similarity-matrix distortion, correlation distance, KL/JS divergence, Wasserstein distance.

5. **Select the constraint-geometry normalization**
   - Use the taxonomy below.
   - State why the chosen normalization is appropriate.

6. **Compute CSD**
   - Calculate `Delta(lambda) / G(lambda)` for every constraint level.

7. **Compare raw and normalized behavior**
   - Plot raw deviation and CSD side by side.
   - Look for collapse, residual structure, discontinuity, or failed normalization.

8. **Interpret conservatively**
   - Do not claim discovery from raw deviation alone.
   - Treat CSD residuals as candidates for intrinsic structure, not proof by themselves.

---

## Constraint-normalization taxonomy

### 1. Independent geometric constraints

Use when removed or compressed degrees of freedom behave approximately independently.

Typical cases:

- Rank truncation
- Random projection
- PCA dimension under weak correlation
- Independent feature bottlenecks
- Euclidean/Frobenius accumulation

Recommended normalization:

```text
G(k) = sqrt(k)
```

or, if measuring removed dimension rather than retained dimension:

```text
G(k_removed) = sqrt(k_removed)
```

Interpretation:

- If raw deviation grows but CSD is smooth and order-one, the effect is likely constraint-geometric.
- If CSD retains parameter dependence, genuine structure may remain.

---

### 2. Correlated geometric constraints

Use when dimensions or modes are strongly correlated, anisotropic, graph-structured, or spectrally concentrated.

Typical cases:

- Graph Laplacian embeddings
- Anisotropic representation spaces
- Correlated latent dimensions
- Vision or NLP embeddings with concentrated spectra
- Manifold-like feature spaces

Recommended normalization:

```text
G(lambda) = sqrt(d_eff)
```

Effective dimension can be estimated with the participation-ratio form:

```text
d_eff = (sum_i sigma_i)^2 / sum_i sigma_i^2
```

where `sigma_i` may be singular values, eigenvalues, variances, or mode weights.

Interpretation:

- If raw dimension normalization fails but effective-dimension normalization collapses the data, the constraint is correlated rather than independent.
- Residual trends after `sqrt(d_eff)` normalization may indicate anisotropy, clustering, or intrinsic structure.

---

### 3. Information-limiting constraints

Use when the constraint removes information rather than geometric degrees of freedom.

Typical cases:

- Dataset-size reduction
- Label noise
- Context-window truncation
- Measurement noise
- Missing observations
- Compression of semantic memory
- Reduced sensor availability

Recommended normalization:

```text
G(lambda) = Delta S(lambda)
```

or another information-loss estimate, such as:

- Entropy loss
- Mutual-information reduction
- KL/JS divergence from reference distribution
- Information bottleneck estimate
- Effective sample information

Interpretation:

- Dimension-based normalization is usually wrong here.
- Smooth entropy-normalized CSD suggests degradation is expected from information loss.
- Persistent residuals suggest intrinsic sensitivity to missing information.

---

### 4. Stochastic constraints

Use when constraints are random or ensemble-based.

Typical cases:

- Random projection seeds
- Dropout-like constraints
- Stochastic pruning
- Random masking
- Monte Carlo subspace sampling
- Randomized numerical solvers

Recommended approach:

```text
CSD(lambda) = E[Delta(lambda)] / G(E[constraint geometry])
```

Also report uncertainty:

```text
mean CSD ± standard deviation
```

or confidence intervals across stochastic realizations.

Interpretation:

- Individual raw deviations may be noisy.
- A stable mean CSD with shrinking variance suggests constraint-driven stochasticity.
- Persistent seed-dependent anomalies may reveal unstable or fragile structure.

---

### 5. Topological observables

Use special caution when the observable is topological or non-additive.

Typical cases:

- Winding number
- Zak phase
- Chern number
- Homology or Betti numbers
- Discrete phase indices
- Nonlocal invariants

Recommended normalization:

```text
G(lambda) = 1
```

In other words: **do not normalize away the discontinuity**.

Interpretation:

- Topological discontinuities are often the signal.
- Applying smooth geometric normalization can destroy the meaning of the observable.
- For topological quantities, CSD functions mainly as a resolution/stability check rather than a scaling-collapse tool.

---

## Interpretation rules

### Large raw deviation + small or smooth CSD

Likely interpretation:

```text
The effect is mostly inevitable constraint-induced distortion.
```

Do not claim a critical threshold or intrinsic collapse from raw deviation alone.

---

### Large raw deviation + large CSD

Likely interpretation:

```text
Intrinsic sensitivity remains after accounting for constraint geometry.
```

This is a candidate signal for genuine structure, instability, phase behavior, or important model dependence.

---

### Smooth CSD curve

Likely interpretation:

```text
No strong evidence for an intrinsic threshold.
```

A raw “elbow” may be a geometric scaling artifact.

---

### CSD anomaly, discontinuity, or persistent residual

Likely interpretation:

```text
There may be real structure not explained by the constraint model.
```

Follow up with ablations, alternative references, alternative norms, and independent validation.

---

### No plausible normalization collapses the data

Likely interpretation:

```text
The system may contain genuinely new structure, the observable may be poorly chosen, or the constraint taxonomy is incomplete.
```

This is a discovery condition, not an automatic proof.

---

## Standard workflow template

Use this template when applying CSD in a report or experiment log.

```markdown
# Constraint Sensitivity Diagnostic Report

## 1. Objective
What scientific or engineering claim is being tested?

## 2. System
Describe the model, dataset, simulation, operator, or memory system.

## 3. Observable
Define X precisely.

## 4. Reference
Define X_ref and why it is the least-constrained reference.

## 5. Constraint family
Define lambda and the tested values.

## 6. Raw deviation
Define Delta(lambda) and the norm/discrepancy used.

## 7. Constraint geometry
Classify the constraint as independent, correlated, informational, stochastic, topological, or mixed.

## 8. Normalization
Define G(lambda) and justify it.

## 9. Results
Compare raw deviation and CSD.

## 10. Interpretation
State whether the observed effect is constraint-driven, residual, anomalous, or unresolved.

## 11. Limitations
List reference dependence, norm dependence, missing ablations, and possible alternative normalizations.

## 12. Next experiments
Define the next ablation or validation test.
```

---

## Pseudocode

```python
import numpy as np


def frobenius_delta(x_lambda, x_ref):
    return np.linalg.norm(x_lambda - x_ref, ord="fro")


def effective_dimension(spectrum, eps=1e-12):
    spectrum = np.asarray(spectrum, dtype=float)
    spectrum = np.maximum(spectrum, 0.0)
    numerator = spectrum.sum() ** 2
    denominator = (spectrum ** 2).sum() + eps
    return numerator / denominator


def csd_score(delta, geometry_scale, eps=1e-12):
    return delta / (geometry_scale + eps)


def csd_independent(x_values, x_ref, k_values):
    rows = []
    for k, x_lambda in zip(k_values, x_values):
        delta = frobenius_delta(x_lambda, x_ref)
        g = np.sqrt(k)
        rows.append({"k": k, "delta": delta, "G": g, "CSD": csd_score(delta, g)})
    return rows


def csd_correlated(x_values, x_ref, spectra):
    rows = []
    for x_lambda, spectrum in zip(x_values, spectra):
        delta = frobenius_delta(x_lambda, x_ref)
        d_eff = effective_dimension(spectrum)
        g = np.sqrt(d_eff)
        rows.append({"d_eff": d_eff, "delta": delta, "G": g, "CSD": csd_score(delta, g)})
    return rows
```

---

## Application areas

### Scientific computing

CSD can audit reduced-order models, spectral solvers, discretization effects, low-rank approximations, mesh-resolution studies, and surrogate models.

Best use:

```text
Separate solver-resolution artifacts from genuine physical or numerical structure.
```

---

### Quantum simulation and physics

CSD can audit Hilbert-space truncation, tensor-network bond dimension, TCSA truncation, Liouvillian cutoffs, projected Hamiltonians, and open-system approximations.

Best use:

```text
Prevent false claims of phase transitions or emergent effects caused by truncation geometry.
```

---

### Machine learning representation analysis

CSD can audit PCA, autoencoders, bottleneck models, embedding compression, representation similarity, model pruning, and latent-dimension claims.

Best use:

```text
Test whether apparent critical latent dimensions are real or caused by dimensional scaling.
```

---

### AI agent memory systems

CSD can analyze memory databases under compression, summarization, embedding updates, retrieval pruning, or long-term memory growth.

Best use:

```text
Distinguish useful memory reorganization from distortion caused by compression or retrieval constraints.
```

Possible observables:

- Embedding covariance
- Cluster centroids
- Topic distributions
- Memory retrieval graph
- Semantic similarity matrix
- Effective rank of the memory store

---

### Continual learning and model adaptation

CSD can measure representation drift across tasks, adapter updates, or memory injections. It can be paired with active methods such as G-CL, but it remains diagnostic rather than a learning controller.

Best use:

```text
Measure whether task adaptation causes intrinsic representation change or only expected constraint-driven drift.
```

---

### Engineering design and robustness

CSD can be used as a robustness audit when designing systems that operate under constraints.

Examples:

- Sensor-limited robotics
- Compressed control policies
- Edge AI models
- Constrained optimization systems
- Noisy measurement systems
- Resource-limited inference pipelines

Best use:

```text
Identify which constraints produce tolerable distortion and which produce residual structural failure.
```

---

## Common mistakes

### Mistake 1: Treating raw deviation as structure

Large raw deviation often means only that the constraint is strong.

Always compare raw deviation against CSD.

---

### Mistake 2: Using `sqrt(k)` everywhere

`sqrt(k)` is valid mainly for independent geometric constraints. It is not universal.

Use effective dimension, entropy, stochastic expectation, or no normalization when appropriate.

---

### Mistake 3: Normalizing topological signals away

For topological invariants, discontinuity may be the true structure.

Do not smooth or divide away topological transitions.

---

### Mistake 4: Choosing `G(lambda)` post hoc to force collapse

CSD is only scientifically meaningful if the normalization is predicted from the constraint geometry.

State the normalization before interpreting the result.

---

### Mistake 5: Calling CSD a correction method

CSD does not reconstruct lost information. It diagnoses whether residual structure remains after expected distortion is accounted for.

---

## Validation checklist

Before trusting a CSD result, confirm:

- [ ] The reference observable is clearly defined.
- [ ] The constraint family is systematic and reproducible.
- [ ] The norm or discrepancy is appropriate for the observable.
- [ ] The normalization is justified by constraint geometry.
- [ ] Raw and normalized curves are both reported.
- [ ] At least one wrong normalization or baseline is tested when possible.
- [ ] Topological observables are not incorrectly normalized.
- [ ] Residual anomalies are validated with ablations.
- [ ] Claims are phrased as diagnostic evidence, not automatic proof.

---

## Recommended outputs

A complete CSD analysis should produce:

1. Raw deviation table
2. CSD-normalized table
3. Raw deviation plot
4. CSD plot
5. Constraint-geometry justification
6. Interpretation table
7. Failure-mode or wrong-normalization check
8. Reproducibility notes

Suggested table columns:

```text
constraint_value | raw_delta | geometry_scale | csd | constraint_type | notes
```

---

## Report language

Use conservative language.

Good:

```text
After normalization by the predicted constraint geometry, the apparent threshold disappears, suggesting that the raw elbow was primarily constraint-induced.
```

Good:

```text
The residual CSD anomaly persists across reference choices and norms, making it a candidate intrinsic feature.
```

Avoid:

```text
CSD proves the system has a phase transition.
```

Avoid:

```text
CSD removes all artifacts.
```

Avoid:

```text
The correct normalization is whatever makes the curve collapse.
```

---

## Relationship to G-CL / Continuous Learning Geometry

CSD and G-CL are related but distinct.

- **CSD** is diagnostic. It measures and interprets constraint-induced distortion.
- **G-CL** is active. It uses geometry signals to regulate learning or memory updates.

A useful combined workflow is:

1. Use CSD to diagnose how a model, memory store, or representation changes under constraints.
2. Use the result to design a G-CL controller or regularizer if active stabilization is needed.
3. Re-run CSD after stabilization to verify whether residual distortion has decreased.

---

## Future development directions

### Predictive constraint analysis

Estimate the expected normalization before running the experiment.

### Constraint taxonomy expansion

Add new constraint classes beyond independent, correlated, informational, stochastic, and topological.

### Automated CSD pipelines

Build tools that automatically:

- detect constraint type,
- compute candidate normalizations,
- compare raw and CSD curves,
- flag possible false thresholds.

### Integration with AI memory systems

Use CSD to audit how memory compression, summarization, retrieval pruning, or embedding updates distort an agent’s memory geometry.

### Discovery protocol

Treat failure of all plausible constraint normalizations as a signal that either:

- the observable contains new structure,
- the constraint class is unknown,
- or the experimental design is incomplete.

### Links to established theory

Develop formal connections with:

- finite-size scaling,
- random matrix theory,
- concentration of measure,
- information geometry,
- numerical analysis,
- representation learning,
- robustness testing.

---

## One-sentence summary

The Constraint Sensitivity Diagnostic is a geometry-informed method for deciding whether observed changes under constraints reflect genuine structure or inevitable distortion caused by the constraint itself.

---

# Advanced Additions: CSD Window, Constraint Incompatibility, and Control Use

## CSD window

CSD can be used not only to normalize distortion, but also to identify a **constraint-critical operating window**.

A system is in a useful CSD regime when:

- the observable responds to constraint deformation;
- the response is nonzero but bounded;
- higher-order changes do not diverge;
- the sensitivity concentrates in structured subspaces, modes, or trajectories;
- the system is neither fully rigid nor unstable.

Interpretation:

| Regime | Diagnostic meaning | Typical action |
|---|---|---|
| Very low sensitivity | over-constrained, redundant, frozen, or trivial | reduce constraint stiffness or test weaker constraints |
| Bounded intermediate sensitivity | constructive structure, adaptation, useful response | preserve or study this regime |
| Very high unstable sensitivity | collapse, noise amplification, uncontrolled drift | increase stabilizing constraint or change architecture |

This “CSD window” is especially useful in adaptive systems, quantum measurement competition, memory-bearing dynamics, and AI continual learning.

---

## Constraint incompatibility principle

CSD is most informative when multiple constraints cannot be simultaneously satisfied.

Examples:

- non-commuting measurements in quantum systems;
- competing loss functions in machine learning;
- conflicting objectives in control systems;
- incompatible memories in agent systems;
- competing regularizers during model adaptation;
- multiple projectors or truncations that do not share a stable basis.

Useful structure often appears when incompatibility is **balanced**:

```text
not zero incompatibility
not dominant instability
but persistent bounded sensitivity
```

In this regime, CSD helps distinguish real collective structure from inevitable distortion.

---

## CSD as a control signal

CSD is primarily diagnostic, but the same measurements can be used as a feedback signal in adaptive systems.

Possible controlled variables:

- learning rate;
- regularization strength;
- geometry-preservation strength;
- projection rank;
- memory update size;
- replay pressure;
- measurement strength;
- adapter routing;
- architecture expansion triggers.

Generic control rule:

```text
if the system is rigid:
    reduce constraint stiffness
elif the system is unstable:
    increase stabilizing constraint
elif the system is in the CSD window:
    preserve current regime
```

When using CSD for control, keep the diagnostic record separate from the control action. Always report both the measured CSD signal and the intervention chosen from it.

---

## Constructive emergence criteria

Do not interpret every nonzero signal as emergence. A CSD signal is a strong candidate for constructive emergence only when:

1. the raw response is nonzero;
2. the response survives the predicted normalization;
3. the signal is not random noise;
4. the signal remains bounded;
5. the signal is localized in a structured subspace, mode, representation, or trajectory family;
6. the result persists under at least one ablation or alternative reference.

Rule:

```text
large raw deviation is not evidence of emergence by itself
```

---

## Operator-sensitivity formulation

For operator-defined systems, CSD can also be written as a sensitivity analysis.

Let a constrained system be represented by a state, operator, model, or process `S(lambda)`, and let `O(lambda)` be an observable extracted from it. Then CSD studies:

```text
C(lambda) = dO / dlambda
```

or, in finite experiments:

```text
Delta(lambda) = || O(lambda) - O(reference) ||
CSD(lambda) = Delta(lambda) / G(lambda)
```

Useful operator-level constraint parameters include:

- commutator strength;
- memory time;
- measurement rate;
- projector rank;
- regularization strength;
- learning-rate pressure;
- adapter capacity;
- memory compression ratio.

This formulation is useful when CSD is applied to quantum dynamics, emergent-geometry pipelines, adaptive AI systems, or agent memory stores.

---

## CSD-driven AI and memory diagnostics

When used with AI systems, CSD should usually be applied to **representation geometry**, not only to final accuracy or loss.

Useful AI observables:

- hidden-state mean orientation;
- hidden-state covariance;
- effective rank;
- adapter activation geometry;
- embedding-memory centroid drift;
- cluster separation;
- retrieval graph distortion;
- task-interference matrix.

Useful AI constraints:

- LoRA rank;
- adapter capacity;
- replay size;
- memory compression;
- context length;
- learning-rate schedule;
- geometry-preservation strength;
- number of incompatible tasks.

Interpretation:

```text
stable low drift may mean preservation
bounded drift with rank growth may mean useful adaptation
unbounded drift or rank collapse may mean forgetting or instability
```

---

## Additional future development directions

### Adaptive control and CSD window tracking

Develop controllers that automatically keep systems inside the useful CSD window. Examples:

- adaptive regularization in continual learning;
- adaptive measurement strength in quantum simulations;
- adaptive replay pressure in ML;
- adaptive memory insertion thresholds in agent systems;
- architecture expansion when no stable CSD window exists.

### Constraint incompatibility benchmarks

Create benchmark tasks where the degree of incompatibility is known or controlled. This is useful for testing whether CSD detects genuine incompatible constraints rather than ordinary noise or scale effects.

### CSD + G-CL combined workflows

Use CSD to identify the geometry and stability regime, then use G-CL or another controller to regulate adaptation. Re-run CSD afterward to verify whether stabilization genuinely reduced residual distortion.

