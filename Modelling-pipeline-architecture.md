# Modelling Pipeline Architecture: L0–L7

## Overview

The pipeline is designed to transform raw telemetry into actionable RUL and risk scoring. Each layer builds on the previous, using physics-informed models, temporal attention mechanisms, and digital twins to capture both gradual degradation and non-degradation failures.

**Layers:**

1. **L0 – Telemetry Ingestion**
   - Inputs: Firmware-processed multi-sensor composites (temperature, voltage, current, bias currents).
   - Purpose: Standardize and parse heterogeneous telemetry.
   - Output: Cleaned sequences with timestamps aligned.

2. **L1 – Signal Conditioning**
   - Missing value repair, outlier suppression, normalization.
   - Hot-plug exclusion window (500ms) prevents false anomaly triggers on module insertion.

3. **L2 – Physics State Engine**
   - Thermal stress computation, laser aging integration (Arrhenius), electrical drift accumulation, solder fatigue Coffin-Manson integration.
   - Output: Physics state vector `[thermalStress, thermalMargin, fatigueIndex, laserAgingState, linkMargin, fecRUL_estimate, opticalImpairmentScore, voltageDriftStress, electricalAgingState, pamEyeMargin, siAnomalyScore, eqFaultPattern]`.

4. **L3 – Dual-Stream Fusion**
   - Stream A: Threshold-based anomaly detection.
   - Stream B: Residuals and FEC codeword monotonicity for early detection of deterministic errors.
   - Output: Aggregated anomaly vector.

5. **L4 – Sequence Forecasting**
   - Models: Chronos, PathTST.
   - Feature-first attention over physics state + anomaly evidence.
   - Forecasts future system state.

6. **L5 – Risk Classification**
   - Models: iTransformer with four specialized heads (C1-C4).
   - Multi-label outputs for simultaneous failure class detection.
   - Includes regression for drift rates and auxiliary metrics.

7. **L6 – Digital Twin / RUL Estimation**
   - Physics-conditioned particle filters, FEC slope extrapolation, or risk flag output for non-degradation failures.
   - Integrates multiple failure trajectories with confidence weighting.

8. **L7 – Scheduling & Dashboard Integration**
   - Uses L6 outputs to generate priority scores and maintenance recommendations.
   - Integrates with NetOps dashboard and Cisco Nexus API.

## Layer Mapping

| Layer | Model / Mechanism | Input | Output | Key Notes |
|-------|-----------------|-------|--------|-----------|
| L0 | Telemetry Parser | Raw DME metrics, gRPC payloads | Cleaned sequence | Time-aligned, schema validated |
| L1 | Filtering / Validation | L0 output | Conditioned sequence | Hot-plug gate, spike suppression |
| L2 | Physics Engine | L1 output | Physics state vector | Thermal, optical, electrical, SI/DSP primitives |
| L3 | Fusion Layer | L2 vector | Anomaly evidence | Dual-stream, cross-device aggregation |
| L4 | Temporal Forecast | L3 vector | Predicted sequence | Chronos, PathTST sequence models |
| L5 | Risk Classification | L4 sequence | P(C1..C4), drift regressor | iTransformer, multi-label |
| L6 | RUL Inference | L5 outputs | RUL distribution / risk flag | Digital Twin particle filter, FEC slope |
| L7 | Scheduler | L6 outputs | Maintenance recommendations | NetOps & Nexus Dashboard integration |
