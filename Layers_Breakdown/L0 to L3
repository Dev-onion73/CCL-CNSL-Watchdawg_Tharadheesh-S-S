# L0 to L3 — Telemetry Ingestion, Signal Conditioning, Physics State, and Anomaly Aggregation

## **Overview**
This document describes layers **L0 to L3** of the QSFP-DD transceiver predictive monitoring pipeline. These layers form the foundation for processing raw telemetry into latent hardware health signals.

---

## **L0 — Telemetry Ingestion**

**Role:** Collect raw telemetry data from transceivers via Cisco DME schemas.

**Telemetry Sources:**
- Primary: In-house Cisco collector / long-term telemetry store.
- Fallback: Telegraf gRPC collectors (hot-swappable if primary unavailable).

**Key Metrics & DME Mapping:**

| Metric Name | Description | Metric_class | Data Type |
|-------------|-------------|--------------|-----------|
| temperature | Module temperature | pie:TransceiverSensor | float (°C) |
| laserTemperature | Laser junction temp | pie:TransceiverSensor | float (°C) |
| voltage | Module voltage | pie:TransceiverSensor | float (V) |
| biasCurrent | Laser bias current | pie:TransceiverSensor | float (A) |
| rxPower | Received optical power | pie:TransceiverSensor | float (mW/dBm) |
| preFECBERAverage | Average pre-FEC BER | tpmon:FecReport | uint32 |
| txPower | Transmit optical power | pie:TransceiverSensor | float (mW/dBm) |

**Processing Steps:**
1. **gRPC / MDT Input:** Pulls telemetry streams.
2. **Schema Parsing:** Maps metrics to DME classes.
3. **Timestamp Alignment:** Ensures uniform time indices across devices.

**Example:**  
A QSFP-DD transceiver reports `temperature = 42.3°C`, `voltage = 3.3V`, `preFECBERAverage = 12`. L0 aligns timestamps, associates metrics with metric_class, and outputs a cleaned stream for L1.

---

## **L1 — Signal Conditioning**

**Role:** Clean and normalize telemetry for downstream physics modeling.

**Steps:**
1. **Missing Value Repair:** Interpolation or physics-informed imputation.
2. **Outlier Rejection:** Remove telemetry points violating physical limits (e.g., negative voltage).
3. **Normalization:** Convert metrics to standard scales for cross-device analysis.

**Example:**  
Voltage readings `[3.3, 3.31, NaN, 3.29]` → missing value repaired → `[3.3, 3.31, 3.3, 3.29]`.

---

## **L2 — Physics State Engine**

**Role:** Map telemetry to latent hardware states using physics models.

**Models:**
- **Thermal Stress Model:**  
  \( T_{pred}(t) = T(t-1) + \Delta T \cdot k_{thermal} \)  
  Uses `temperature`, `laserTemperature`, `tecCurrent`.

- **Electrical Aging Model:**  
  \( V_{drift}(t) = V(t) - V_0 + f(I_{bias}, t) \)  
  Tracks biasCurrent, moduleVoltage, voltageWarnings.

- **Latent State Vector:** Combines thermal and electrical states as a vector for L3.

**Example:**  
Telemetry: `temperature = 42°C`, `biasCurrent = 30mA`, `voltage = 3.3V`.  
L2 computes latent vector: `[thermalStress=0.35, electricalDegradation=0.12]`.

---

## **L3 — Anomaly Aggregation**

**Role:** Detect anomalies across devices and metrics.

**Streams:**
- **Residual Stream:** Observed deviation from L2 physics projection.
- **Precursor Stream:** Early warnings based on BER, CD, SNR trends.

**Fusion:** Weighted aggregation to produce cluster-level anomaly scores.

**Example:**  
Residuals: `[0.05, 0.12, 0.07]`  
Precursor: `[0.2, 0.15, 0.18]`  
Aggregated anomaly vector → `[0.12, 0.14]` for L4 temporal forecasting.

---

**Summary Table:**

| Layer | Input | Models / Method | Output |
|-------|-------|----------------|--------|
| L0 | Raw telemetry | N/A | Cleaned, timestamped metrics |
| L1 | L0 output | Rule-based cleaning | Conditioned metrics |
| L2 | L1 output | Thermal/Electrical physics | Latent hardware state vector |
| L3 | L2 output | Residual/precursor fusion | Cluster-level anomaly vector |
