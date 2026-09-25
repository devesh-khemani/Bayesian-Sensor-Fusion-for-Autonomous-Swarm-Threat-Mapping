# Bayesian Sensor Fusion for Autonomous Swarm Threat-Mapping

**[View Implementation Notebook](./swarm_project.ipynb)**

An algorithmic prototype and simulation investigating multi-modal Bayesian sensor fusion for distributed autonomous platforms operating under sensor degradation and range-dependent uncertainty.

The system models a spatial environment as a dynamic discrete probability grid, simulating the extraction of calibrated target presence beliefs from independent, noisy sensing modalities (electro-optical and thermal signatures).

---

## Technical Overview

* **Spatial Occupancy Grid:** Represents a $100 \times 100$ spatial domain as a continuous probability map, tracking cell-wise belief states for binary hypothesis testing (Target Present vs. Target Absent).
* **Multi-Modal Gaussian Noise Modeling:** Simulates independent observation channels—thermal infrared signatures and visual/optical classification metrics—using class-conditional Gaussian distributions.
* **Range-Dependent Uncertainty Scaling:** Explicitly inflates observational variance as a function of Euclidean separation distance ($d$) between sensor platform and target cell, modeling real-world signal attenuation and spatial resolution dropoff.
* **Two-Stage Bayesian Update:** 
  1. Computes single-channel posterior beliefs by conditioning prior grid cells on noisy observation streams.
  2. Synthesizes inter-channel agreement via recursive Bayesian fusion, yielding an integrated threat probability map.

---

## Mathematical Formulation

### 1. Class-Conditional Likelihood Modeling
Each sensing channel models measurement likelihoods using class-conditional Gaussian probability density functions:

$$f(z \mid \mathcal{H}_i) = \frac{1}{\sqrt{2\pi}\,\sigma_{\text{eff}}} \exp\left(-\frac{(z - \mu_i)^2}{2\sigma_{\text{eff}}^2}\right)$$

where:
* $\mathcal{H}_1$: Target Present
* $\mathcal{H}_0$: Target Absent (Null Hypothesis)
* $z$: Measured continuous sensor scalar

### 2. Distance-Scaled Variance Inflation
Rather than assuming static instrumentation noise, observational confidence degrades over range according to:

$$\sigma_{\text{eff}} = \sqrt{\sigma_{\text{baseline}}^2 + k \cdot d^2}$$

where $d = \sqrt{(x - x_{\text{agent}})^2 + (y - y_{\text{agent}})^2}$ and $k$ represents an empirical spatial dispersion penalty coefficient.

### 3. Recursive Bayesian Update
For each channel, incoming measurements update prior spatial beliefs via Bayes' Rule:

$$P(\mathcal{H}_1 \mid z) = \frac{P(z \mid \mathcal{H}_1) \cdot P(\mathcal{H}_1)}{P(z \mid \mathcal{H}_1) \cdot P(\mathcal{H}_1) + P(z \mid \mathcal{H}_0) \cdot P(\mathcal{H}_0)}$$

Independent posterior channels are subsequently consolidated into a joint belief state:

$$P(\mathcal{H}_1 \mid z_{\text{thermal}}, z_{\text{optical}}) \propto \frac{P(\mathcal{H}_1 \mid z_{\text{thermal}}) \cdot P(\mathcal{H}_1 \mid z_{\text{optical}})}{P(\mathcal{H}_1)}$$

---

## Tools & Technologies

* **Language:** Python
* **Computation & Linear Algebra:** NumPy
* **Visualization:** Matplotlib

---

## Repository Structure

| File | Description |
| :--- | :--- |
| `swarm_project.ipynb` | Grid formulation, sensor noise simulation, Bayesian fusion pipeline, and visualization |
| `README.md` | Theoretical formulation, mathematical derivations, and architecture details |

---

## Setup & Execution

1. **Clone the repository:**
   ```bash
   git clone [https://github.com/devesh-khemani/Bayesian-Sensor-Fusion-Swarm-Threat-Mapping.git](https://github.com/devesh-khemani/Bayesian-Sensor-Fusion-Swarm-Threat-Mapping.git)
   cd Bayesian-Sensor-Fusion-Swarm-Threat-Mapping
