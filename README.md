# Bayesian Sensor Fusion for Autonomous Swarm Threat-Mapping

**[View Implementation Notebook](./swarm_project.ipynb)**

An algorithmic prototype and simulation framework investigating multi-modal Bayesian sensor fusion for distributed autonomous platforms operating under contested conditions and range-dependent sensor uncertainty.

The project models a continuous 2D spatial environment as a dynamic discrete probability grid, evaluating how an autonomous agent can synthesize calibrated target presence estimates from multiple independent, imperfect sensing modalities (electro-optical and thermal signatures).

---

## Technical Overview

* **Spatial Occupancy Grid:** Models a $100 \times 100$ spatial domain as a continuous probability map, maintaining localized, cell-wise belief states for binary hypothesis testing (Target Present $\mathcal{H}_1$ vs. Target Absent $\mathcal{H}_0$).
* **Multi-Modal Gaussian Noise Modeling:** Simulates independent observation channels using class-conditional Gaussian probability density functions with distinct noise profiles for target-present and target-absent states.
* **Range-Dependent Uncertainty Scaling:** Explicitly models physical sensor attenuation over distance by scaling noise variance proportionally to Euclidean range, preventing overconfident posterior estimates on distant targets.
* **Hierarchical Bayesian Fusion Pipeline:** 
  1. Computes single-channel posterior belief states by evaluating raw continuous measurements against prior grid occupancy states.
  2. Synthesizes cross-channel consensus via recursive Bayesian updating, resolving single-sensor false positives and cross-modality ambiguity.

---

## Mathematical Formulation

### 1. Class-Conditional Likelihood Modeling
Each sensing channel models observation likelihoods using class-conditional Gaussian distributions:

$$f(z \mid \mathcal{H}_i) = \frac{1}{\sqrt{2\pi}\,\sigma_{\text{eff}}} \exp\left(-\frac{(z - \mu_i)^2}{2\sigma_{\text{eff}}^2}\right)$$

where:
* $\mathcal{H}_1$: Target Present
* $\mathcal{H}_0$: Target Absent (Null Hypothesis)
* $z$: Scalar sensor observation (thermal signature or optical confidence score)

### 2. Range-Dependent Variance Inflation
Rather than assuming a static instrumentation noise floor, observational uncertainty degrades over distance ($d$):

$$\sigma_{\text{eff}} = \sqrt{\sigma_{\text{class}}^2 + k \cdot d^2}$$

where $d = \sqrt{(x - x_{\text{agent}})^2 + (y - y_{\text{agent}})^2}$ and $k$ represents an empirical dispersion penalty coefficient, formalizing decreased sensor resolution over range.

### 3. Recursive Bayesian Update
For each channel, incoming measurements update prior spatial occupancy beliefs via Bayes' Rule:

$$P(\mathcal{H}_1 \mid z) = \frac{P(z \mid \mathcal{H}_1) \cdot P(\mathcal{H}_1)}{P(z \mid \mathcal{H}_1) \cdot P(\mathcal{H}_1) + P(z \mid \mathcal{H}_0) \cdot P(\mathcal{H}_0)}$$

Independent posterior channels are subsequently synthesized into an integrated joint threat map:

$$P(\mathcal{H}_1 \mid z_{\text{thermal}}, z_{\text{optical}}) \propto \frac{P(\mathcal{H}_1 \mid z_{\text{thermal}}) \cdot P(\mathcal{H}_1 \mid z_{\text{optical}})}{P(\mathcal{H}_1)}$$

Grid consistency is maintained after each update epoch via $P(\mathcal{H}_0) = 1 - P(\mathcal{H}_1)$.

---

## Performance & System Relevance

* **Multi-Modal Robustness:** Fusing non-homogenous sensor modalities prevents single-point spoofing and environmental false positives (e.g., optical occlusion vs. thermal noise).
* **First-Principles Inference:** Implemented direct mathematical updating rather than generic black-box tooling, verifying numerical stability and convergence bounds under sensor disagreement.
* **Calibrated Confidence:** Range-scaling guarantees that distant or ambiguous signals do not distort localized target tracks, enabling robust tracking under signal degradation.

---

## Tools & Technologies

* **Language:** Python
* **Scientific Computing:** NumPy
* **Visualization:** Matplotlib

---

## Repository Structure

| File | Description |
| :--- | :--- |
| `swarm_project.ipynb` | Grid formulation, sensor noise simulation, Bayesian fusion pipeline, and visualization |
| `README.md` | Theoretical derivations, mathematical formulations, and architecture documentation |

---

## Setup & Execution

1. **Clone the repository:**
   ```bash
   git clone [https://github.com/devesh-khemani/Bayesian-Sensor-Fusion-Swarm-Threat-Mapping.git](https://github.com/devesh-khemani/Bayesian-Sensor-Fusion-Swarm-Threat-Mapping.git)
   cd Bayesian-Sensor-Fusion-Swarm-Threat-Mapping
