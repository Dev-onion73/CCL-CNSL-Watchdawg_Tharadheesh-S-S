# Modelling Pipeline Architecture (L0–L7)

This document details the **full predictive pipeline** from raw telemetry to actionable maintenance recommendations.

---

## Layer-Wise Architecture & Function

| Layer | Purpose | Models / Approach | Data Flow Highlights |
|-------|---------|-----------------|--------------------|
| **L0: Telemetry Connector** | Ingest telemetry from QSFP-DD transceivers | gRPC/MDT ingestion, schema parsing, timestamp alignment | Inputs raw device metrics; forwards preprocessed data to L1 |
| **L1: Signal Conditioning** | Clean, normalize, repair telemetry | Missing value repair, outlier rejection, scaling | Ensures robust inputs for physics modeling |
| **L2: Physics State Engine** | Compute latent physical states | Thermal, electrical, aging, solder fatigue models | Outputs hidden states for L3 aggregation |
| **L3: Anomaly Aggregator** | Fuse signals across devices and racks | Dual-stream fusion: residual + precursor streams | Identifies anomalies within clusters |
| **L4: Temporal Inference** | Forecast future states | Chronos, PathTST | Predicts evolution of physical stress over time |
| **L5: Risk Classification** | Map forecasts to health scores | iTransformer | Produces multi-class health risk probabilities |
| **L6: RUL Inference** | Estimate remaining useful life | Digital Twin (physics + particle filter / ADT) | Computes expected degradation timeline |
| **L7: Scheduler / Decision Layer** | Generate actionable tasks | Priority scoring, maintenance window selection | Integrates with NetOps / Nexus Dashboard for alerting |

---

### Data Flow

1. L0 ingests telemetry → passes to L1  
2. L1 cleans, normalizes → passes to L2  
3. L2 computes hidden physics states → feeds L3  
4. L3 fuses multi-device anomalies → passes to L4  
5. L4 forecasts sequences → feeds L5  
6. L5 classifies risk → passes to L6  
7. L6 computes RUL → L7 schedules interventions → updates dashboards

---

### Granular Notes

- L2 Physics includes:
  - Thermal stress: \(dT/dt = (P_{dissipated} - P_{cooling})/C_{thermal}\)
  - Aging: \(AgingRate(T) = A \cdot \exp(-E_a/(kT))\)
  - Electrical drift: \(V_{stress} = |V - V_{nominal}| / (V_{tolerance}/2)\)
  - Latent states used in L3 dual-stream fusion

- L4 Chronos / PathTST handles multivariate time series and cross-device correlations

- L5 iTransformer captures long-range dependencies for risk classification across failure classes

- L6 Digital Twin fuses physics projections with observed telemetry for RUL
