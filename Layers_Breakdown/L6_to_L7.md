# L6 to L7 — Digital Twin RUL Estimation & Scheduling (Expanded)

## **Overview**
Layers **L6 and L7** form the predictive core of the pipeline. The goal is to estimate **Remaining Useful Life (RUL)** for each transceiver or cluster, using **physics-informed models**, and translate that into actionable maintenance scheduling.  

The layers integrate **anomaly signals**, **risk scores**, and **temporal forecasts** into a digital twin framework, providing both **probabilistic RUL distributions** and **deterministic scheduling recommendations**.

---

## **L6 — Digital Twin & RUL Inference**

**Role:** Construct a per-device Digital Twin that mirrors the internal physics of the hardware, and simulate its degradation trajectory.

### **1. Inputs**

- **L5 Risk Scores:** Discrete/continuous risk predictions per transceiver or rack cluster.  
- **L4 Forecasts:** Temporal multi-step predictions for critical metrics (e.g., preFECBER, temperature, biasCurrent).  
- **L2 Latent State Vectors:** Physics-state variables from the telemetry-conditioned metrics.

---

### **2. Physics-based Digital Twin**

Digital Twin models combine **thermal, electrical, and optical physics** to propagate the device state:

1. **Thermal Model:**  

\[
T_{t+\Delta t} = T_t + \Delta t \cdot \left( k_{heat} \cdot P_{loss} + k_{ambient} \cdot (T_{ambient} - T_t) \right)
\]

- \(T_t\): Current module/laser temperature  
- \(P_{loss}\): Power dissipation (from voltage, biasCurrent, txPower)  
- \(k_{heat}\), \(k_{ambient}\): Empirical thermal coefficients

2. **Electrical Aging Model:**

\[
V_{drift}(t) = V(t) - V_0 + \int_0^t f(I_{bias}(\tau), T(\tau)) d\tau
\]

- Integrates current and temperature history to model capacitor aging, solder fatigue, or threshold drift.

3. **Optical Degradation Model:**

\[
BER_{pred} = BER_{base} \cdot \left(1 + \alpha \cdot CD + \beta \cdot DGD + \gamma \cdot PDL \right)
\]

- Incorporates chromatic dispersion (CD), differential group delay (DGD), and polarization dependent loss (PDL) as cross-metric correlates for signal degradation.

---

### **3. Particle Filter RUL Estimation**

L6 uses a **particle filter approach** to generate probabilistic RUL distributions:

**Step 1 — Particle Initialization:**

- Generate \(N\) particles representing possible latent device states:  

\[
x_0^{(i)} = [T_0^{(i)}, V_0^{(i)}, BER_0^{(i)}, \dots]
\]

- Assign uniform or prior-weighted probabilities \(w_0^{(i)} = 1/N\).

**Step 2 — Physics Projection:**

- Propagate each particle using physics models over future time steps \(\Delta t\):  

\[
x_{t+\Delta t}^{(i)} = f_{physics}(x_t^{(i)}) + \epsilon
\]

- \(\epsilon\) captures process noise (environmental uncertainty, voltage fluctuation).

**Step 3 — Posterior Update (Bayesian Weighting):**

\[
w_{t+\Delta t}^{(i)} = w_t^{(i)} \cdot p(y_{t+\Delta t} | x_{t+\Delta t}^{(i)})
\]

- Likelihood \(p(y | x)\) derived from **L5 risk scores**, e.g., high BER deviation increases particle weight for imminent failure.

**Step 4 — RUL Estimation:**

- For each particle, determine failure threshold crossing \(t_f\) (e.g., BER > max allowable, temperature > max rating).  
- Aggregate across particles:  

\[
RUL = \mathbb{E}[t_f - t_{current}] \quad \pm \text{Std Dev}
\]

**Correlation Handling:**

- Particle state vector includes **all critical telemetry metrics** (temperature, biasCurrent, preFECBER, cdMax, dgdMax).  
- Cross-metric correlations are preserved in the physics propagation function \(f_{physics}\).  
- Example: High biasCurrent + rising laserTemperature → accelerates BER degradation.

**Example Calculation:**

- Latent particle state: `[T=42°C, V=3.3V, preFECBER=15]`  
- Physics model projects temperature to 45°C in 2 weeks, BER to 25 → threshold breached.  
- Multiple particles simulated → Expected RUL = **12 ± 2 weeks**.

---

## **L7 — Scheduler / Decision Orchestrator**

**Role:** Translate RUL distributions into actionable maintenance schedules and integrate with dashboards.

### **1. Priority Scoring**

\[
Priority_i = \frac{Criticality_i}{RUL_i + \epsilon}
\]

- Criticality: device-level importance (core vs edge link)  
- RUL: predicted remaining weeks  
- \(\epsilon\): smoothing term to avoid divide-by-zero

### **2. Maintenance Window Selection**

- Constraint-aware scheduling: avoids peak traffic windows, ensures N+1 redundancy.
- Uses **integer programming / heuristic optimization**:  

\[
\text{Minimize } \sum_i Priority_i \cdot \text{DowntimePenalty}_i
\]

- Selects optimal timing for preventive interventions.

### **3. Dashboard Integration**

- **NetOps Dashboard:** Uses API to push scheduled maintenance, RUL metrics, and risk scores.  
- **Cisco Nexus Dashboard API:** Primary interface.  
- **Telemetry Fallback:** If in-house collector unavailable, fallback gRPC-based telemetry ensures data continuity.

**Example:**  

- Device RUL < 2 weeks, priority 0.9 → maintenance auto-scheduled next 48h, alert pushed via NetOps → Nexus API updates device health panel.

---

### **Layer Output Summary Table**

| Layer | Input | Model / Method | Output |
|-------|-------|----------------|--------|
| L6 | L5 risk scores, L4 forecasts | Digital Twin Physics + Particle Filter | RUL distributions, failure probabilities |
| L7 | L6 RUL distributions | Scheduler + Optimization + API integration | Maintenance actions, prioritized alerts, dashboard integration |

---

### **Key Points**

- Digital Twin **correlates multiple metrics** for accurate RUL.  
- Particle filter enables **probabilistic, uncertainty-aware prediction**.  
- Scheduler translates predictions into **practical operational actions**.  
- Full **integration with NetOps and Nexus Dashboards** closes the loop between predictive analytics and operational response.
