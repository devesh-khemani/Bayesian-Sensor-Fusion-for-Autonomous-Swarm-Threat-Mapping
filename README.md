# Bayesian Sensor Fusion — Simulation

**[View Implementation Notebook](./swarm_project.ipynb)**

> **Project Status:** The core Bayesian sensor-fusion mathematics — likelihood modelling, distance-scaled uncertainty, and multi-sensor likelihood fusion — is implemented and verified. Multi-drone coordination and dynamic pathfinding were intentionally left unimplemented as a defined scoping decision, not an oversight (see [Scope and Limitations](#scope-and-limitations-honest-status) below).

This project was written in 2023–24 alongside independent research into swarm robotics and Lethal Autonomous Weapons Systems (LAWS) for my EPQ. It is an exploratory simulation testing how an agent should combine two independent, imperfect sensor channels into a single calibrated probability estimate, and how that estimate should degrade realistically with range and sensor noise. It is shared for transparency on the technical approach and mathematical mechanics behind that research, rather than as a production or deployment-ready system.

---

### Motivation

My EPQ examined the technical limitations of AI-enabled swarm robotics and LAWS — specifically sensor reliability under environmental degradation, adversarial spoofing, and data drift, and how sensor fusion affects target-tracking reliability in contested environments.

This notebook is a small technical counterpart to that research: rather than reasoning about sensor fusion only at a policy/systems level, I built a simulation from first principles to test how an agent should fuse two noisy, independent observation channels (a thermal signature reading and an optical/visual detection score) into a single belief state that scales sensibly with sensor noise and distance-dependent attenuation.

---

### What This Project Does

- **Occupancy Grid Mapping** — Represents a 100 × 100 spatial environment as a discrete probability grid, where each cell holds a belief about target presence.
- **Dual-Channel Noisy Sensing** — Simulates two independent sensor modalities per cell (thermal and optical), each generated with class-conditional Gaussian noise profiles ($H_1$: target present vs. $H_0$: target absent).
- **Combined Sensor + Distance Uncertainty** — Each channel's uncertainty comes from two independent sources: the sensor's intrinsic precision, and additional noise introduced by distance from the observing agent. These are combined correctly in quadrature (root-sum-of-squares), rather than simply added.
- **Bayesian Likelihood Fusion** — Combines the two channels' *likelihoods* (not their posteriors) under a conditional-independence assumption, then applies Bayes' rule once to produce a single fused posterior per cell — avoiding the common error of double-counting the prior by naively combining two independently-computed posteriors.
- **Grid Write-Back** — Writes the fused probability back into the occupancy grid (`update_maps`), maintaining $P(\text{no target}) = 1 - P(\text{target})$ for the updated cell.

---

### How the Sensor Fusion Works

#### 1. Likelihood Modelling

For each sensor channel, two Gaussian distributions represent the expected observation under each hypothesis. A measured value $x$ is evaluated against both using the standard Gaussian PDF:

$$f(x \mid \mu, \sigma) = \frac{1}{\sqrt{2\pi}\,\sigma} \exp\left(-\frac{(x-\mu)^2}{2\sigma^2}\right)$$

#### 2. Combined Sensor + Distance Uncertainty

Rather than treating sensor precision as either fixed or purely range-dependent, both sources of uncertainty are modelled explicitly and combined in quadrature, since they are independent noise sources:

$$\sigma_{\text{eff}} = \sqrt{\sigma_{\text{sensor}}^{2} + k \cdot d^{2}}$$

- $\sigma_{\text{sensor}}$ — the sensor's own intrinsic noise, present even at zero range.
- $k \cdot d^{2}$ — additional noise that grows with Euclidean distance $d$ from the observing agent.

This prevents the agent from forming overconfident beliefs from either a noisy sensor or a distant reading.

#### 3. Likelihood Fusion (Bayes' Rule, Applied Once)

The two channels' likelihoods are multiplied together (valid under conditional independence given the true state), and Bayes' rule is applied a single time against the cell's prior:

$$P(\text{target} \mid x_{\text{th}}, x_{\text{cam}}) = \frac{P(x_{\text{th}}\mid\text{target})\,P(x_{\text{cam}}\mid\text{target})\,P(\text{target})}{P(x_{\text{th}}\mid\text{target})\,P(x_{\text{cam}}\mid\text{target})\,P(\text{target}) + P(x_{\text{th}}\mid\text{no target})\,P(x_{\text{cam}}\mid\text{no target})\,P(\text{no target})}$$

This is the statistically correct way to combine two independent channels against a shared prior — an earlier version of this code instead combined two already-complete single-channel posteriors, which double-counts the prior and biases the result. That was identified and corrected.

#### 4. Grid Update

The fused probability is written back into the occupancy grid for the observed cell, enforcing $P(\text{no target}) = 1 - P(\text{target})$.

---

### Why This Approach

Single-sensor detection is inherently fragile — thermal sensors trigger false positives on heat artifacts, while computer-vision channels are susceptible to occlusion, poor illumination, or adversarial spoofing. Fusing two independent, differently-failing sensors via Bayesian updating is a well-established approach to more robust target identification. 

Explicitly modelling both intrinsic sensor noise and range-dependent attenuation keeps spatial confidence consistent with physical limits. The mechanics were implemented from first principles (plain Python / NumPy, no high-level fusion libraries) to directly verify the underlying mathematics rather than treat it as a black box.

---

### Scope and Limitations (Honest Status)

This is a deliberately bounded proof-of-concept, not a finished tracking system:

- **Single measurement pass, not recursive tracking.** The fusion pipeline (likelihood modelling → distance-scaled uncertainty → Bayesian fusion → grid write-back) is demonstrated on one observation at a time. The grid is structured to support accumulating belief over repeated passes, but the notebook does not currently loop this over multiple sequential measurements.
- **No multi-agent coordination or path planning.** The project tests whether fusing two channels produces a sensible calibrated confidence estimate — it does not address swarm coordination, search strategy, or trajectory planning, which are separate problems outside this project's scope. This boundary is marked explicitly in the code (`#then implement the algorithm`).
- **Synthetic sensor data.** Thermal and camera readings are generated from a simple synthetic grid generator, not from real sensor hardware or logs.

---

### Relevance to Sensor & Defence Systems

Combining imperfect, independent sensor streams into a single calibrated confidence metric that degrades with both sensor noise and range is a foundational idea in real-world target tracking, integrated air defence, and distributed perimeter sensing — and is conceptually related to recursive estimators like the Kalman filter, though this project implements a simpler, non-recursive Bayesian update rather than a full state-space filter. This prototype is a small-scale exploration of that statistical fusion principle, built to accompany and test ideas from my EPQ on sensor reliability in contested environments.

---

### Tools & Technologies

- **Language:** Python
- **Scientific Computing:** NumPy
- **Visualisation:** Matplotlib
