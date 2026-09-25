# Bayesian Sensor Fusion for Autonomous Swarm Threat-Mapping

**[View Implementation Notebook](./swarm_project.ipynb)**

> **Project Status:** The core Bayesian sensor-fusion mathematics — likelihood modelling, distance-scaled uncertainty, and multi-sensor posterior fusion — is implemented and verified. Multi-drone coordination and dynamic pathfinding were intentionally left unimplemented as a defined scoping decision, not an oversight (see [Why this stopped here](#why-this-stopped-here) below).

This project was written in 2023–24 alongside independent research into swarm robotics and Lethal Autonomous Weapons Systems (LAWS) for my EPQ. It is an exploratory simulation testing how an autonomous agent should combine two independent, imperfect sensor channels into a single calibrated probability estimate — and how that estimate should degrade realistically with range and sensor noise. It is shared for transparency on the technical approach and mathematical mechanics behind that research, rather than as a production deployment.

---

## Motivation

My EPQ examined the technical limitations of AI-enabled swarm robotics and LAWS — specifically sensor reliability under environmental degradation, adversarial spoofing, and data drift, and how sensor fusion affects target-tracking reliability in contested environments. 

This notebook serves as the technical counterpart to that research: rather than reasoning about sensor fusion solely at an abstract policy or systems level, I constructed a simulation from first principles to test how an agent should fuse two noisy, independent observation channels (a thermal signature reading and an optical/visual detection score) into a joint belief state that properly scales with distance-dependent sensor attenuation.

---

## What This Project Does

* **Occupancy Grid Mapping:** Represents a $100 \times 100$ spatial environment as a discrete probability grid, where each cell tracks a localized belief regarding target presence.
* **Dual-Channel Noisy Sensing:** Simulates two independent sensor modalities per cell (thermal and optical), each generated with class-conditional Gaussian noise profiles ($H_1$: target present vs. $H_0$: target absent).
* **Distance-Dependent Uncertainty:** Models physical sensor degradation over range: the effective variance of each observation channel inflates proportionally to the Euclidean distance from the observing agent.
* **Hierarchical Bayesian Fusion:** Computes single-channel posteriors via Bayes' rule, then fuses the independent channel posteriors into a unified spatial belief state reflecting cross-modal agreement or disagreement.
* **Dynamic Belief Persistence:** Recursively updates the running grid map (`p_s_map` / `p_n_map`) as successive measurements arrive, maintaining strict probability consistency ($P(\text{no target}) = 1 - P(\text{target})$).

---

## How the Sensor Fusion Works

### 1. Likelihood Modelling
For each sensor channel, two Gaussian distributions represent the expected continuous observation under each hypothesis. A measured value $x$ is evaluated against both distributions using the standard Gaussian probability density function:

$$f(x \mid \mu, \sigma) = \frac{1}{\sqrt{2\pi}\,\sigma} \exp\left(-\frac{(x - \mu)^2}{2\sigma^2}\right)$$

### 2. Distance-Scaled Uncertainty
Rather than assuming static instrumentation precision across all ranges, the effective standard deviation $\sigma_{\text{eff}}$ inflates with Euclidean distance $d$ from the observing agent:

$$\sigma_{\text{eff}} = \sqrt{\sigma_{\text{class}}^2 + k \cdot d^2}$$

Readings captured from further away possess a broader variance profile, preventing the agent from forming overconfident posterior beliefs on distant observations.

### 3. Posterior Update (Bayes' Rule)
Each incoming observation is combined with the cell's prior occupancy probability:

$$P(\text{target} \mid x) = \frac{P(x \mid \text{target}) \cdot P(\text{target})}{P(x \mid \text{target}) \cdot P(\text{target}) + P(x \mid \text{no target}) \cdot P(\text{no target})}$$

### 4. Multi-Sensor Fusion
The thermal-channel posterior and optical-channel posterior are subsequently combined via a secondary Bayesian combination step, ensuring the integrated map responds to cross-sensor verification and downweights single-modality anomalies.

### 5. Belief Persistence
The fused probability is written back into the occupancy grid (`update_maps`), allowing spatial certainty to accumulate over successive measurement passes while enforcing:

$$P(\text{no target}) = 1 - P(\text{target})$$

---

## Why This Approach

Single-sensor detection is inherently fragile — thermal sensors trigger false positives on heat artifacts, while computer vision channels are susceptible to occlusions, poor illumination, or adversarial spoofing. 

Fusing two independent, differently-failing sensors via Bayesian updating is an established approach for robust target identification in contested environments. Explicitly modelling uncertainty as a function of range ensures that spatial confidence matches real-world physical limits. Implementing these mechanics from first principles using NumPy rather than high-level libraries verified the mathematical stability and edge-case behavior directly.

---

## Why This Stopped Here

Development was intentionally concluded once the sensor-fusion mathematics and single-agent spatial updates were verified:

1. **The Research Focus Was Fusion, Not Coordination:** The project aimed to evaluate whether multi-modal fusion yields calibrated target-tracking confidence under degraded observations. Multi-drone pathfinding and trajectory planning are separate search/coordination problems that fall outside that core inference question.
2. **Prioritizing Verifiable Foundations Over Surface Completeness:** The mathematical pipeline could be analytically verified against known manual calculations and controlled inputs. Bolting on an unverified swarm simulation layer would have obscured the underlying mathematical focus.
3. **A Bounded Scope:** The spatial grid, Gaussian sensor noise models, and Bayesian update pipeline form a complete, self-contained module. The subsequent multi-agent coordination steps are marked explicitly in the codebase (`#then implement the algorithm`).

---

## Relevance to Sensor & Defence Systems

Combining imperfect, independent sensor streams into a single calibrated confidence metric that degrades with range and noise is fundamental to real-world target tracking, integrated air defence, and distributed perimeter sensing. This prototype serves as a small-scale proof-of-concept for the statistical filtering techniques deployed in real-world degraded environments.

---

## Tools & Technologies

* **Language:** Python
* **Scientific Computing:** NumPy
* **Visualization:** Matplotlib

---

## Repository Structure

| File | Description |
| :--- | :--- |
| `swarm_project.ipynb` | Threat-map grid generation, Gaussian sensor models, distance-scaled uncertainty, and Bayesian fusion pipeline |
| `README.md` | Theoretical foundation, mathematical formulation, and architecture documentation |

---

## Setup & Usage

1. **Clone the repository:**
   ```bash
   git clone [https://github.com/devesh-khemani/Bayesian-Sensor-Fusion-Swarm-Threat-Mapping.git](https://github.com/devesh-khemani/Bayesian-Sensor-Fusion-Swarm-Threat-Mapping.git)
   cd Bayesian-Sensor-Fusion-Swarm-Threat-Mapping
