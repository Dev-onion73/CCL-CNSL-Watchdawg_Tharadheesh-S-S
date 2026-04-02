
---

## **QSFP-DD Transceiver Telemetry: Cisco DME Schema Classes**

Cisco’s transceiver telemetry exposes metrics via **DME metric_class**, which provides structured, standardized access for ingestion, monitoring, and predictive modeling. Each metric feeds specific layers in the predictive health pipeline (L0–L7).

---

### **1. Thermal / Temperature Metrics**

| Metric Name           | Description                | Metric_class (DME)      | Data Type  | Pipeline Layer | Usage                             |
| --------------------- | -------------------------- | ----------------------- | ---------- | -------------- | --------------------------------- |
| `temperature`         | Module ambient temperature | `pie:TransceiverSensor` | Float (°C) | L2             | Thermal stress, physics state     |
| `laserTemperature`    | Laser junction temperature | `pie:TransceiverSensor` | Float (°C) | L2             | Laser aging in physics model      |
| `moduleTemperature`   | PCB/module temp            | `tpmon:OpticsReport`    | Uint32     | L2             | Digital twin input                |
| `temperatureBadRange` | Out-of-range flag          | `tpmon:OpticsAlarm`     | Enum16     | L1             | Early alert for thermal anomalies |

---

### **2. Electrical / Power Metrics**

| Metric Name       | Description                   | Metric_class (DME)      | Data Type | Pipeline Layer | Usage                          |
| ----------------- | ----------------------------- | ----------------------- | --------- | -------------- | ------------------------------ |
| `voltage`         | Module voltage                | `pie:TransceiverSensor` | Float (V) | L2             | Electrical drift, digital twin |
| `moduleVoltage`   | Voltage at module             | `tpmon:OpticsReport`    | Uint32    | L2             | Physics model input            |
| `voltageWarnings` | Voltage anomaly flags         | `tpmon:OpticsAlarm`     | Uint32    | L1             | Alert triggering               |
| `biasCurrent`     | Laser bias current            | `pie:TransceiverSensor` | Float (A) | L2             | Aging and stress monitoring    |
| `tecCurrent`      | Thermoelectric cooler current | `pie:TransceiverSensor` | Float (A) | L2             | Thermal regulation modeling    |
| `currentBadRange` | Out-of-range current          | `tpmon:OpticsAlarm`     | Enum16    | L1             | Risk scoring & alerts          |

---

### **3. Optical / Signal Metrics**

| Metric Name           | Description                   | Metric_class (DME)      | Data Type | Pipeline Layer | Usage                              |
| --------------------- | ----------------------------- | ----------------------- | --------- | -------------- | ---------------------------------- |
| `cdMax` / `cdMin`     | Chromatic dispersion          | `tpmon:OpticsThreshold` | Sint32    | L3             | Optical signal health              |
| `dgdMax` / `dgdMin`   | Differential group delay      | `tpmon:OpticsThreshold` | Uint32    | L3             | Physics-informed anomaly detection |
| `pdlMax` / `pdlMin`   | Polarization-dependent loss   | `tpmon:OpticsThreshold` | Uint32    | L3             | Link integrity monitoring          |
| `pcrMax` / `pcrMin`   | Polarization change rate      | `tpmon:OpticsThreshold` | Uint32    | L3             | Signal degradation detection       |
| `osnrMax` / `osnrMin` | Optical SNR                   | `tpmon:OpticsThreshold` | Uint32    | L3             | Residual error detection           |
| `residualDispersion`  | Residual chromatic dispersion | `pie:TransceiverSensor` | Uint32    | L3             | Digital twin optical health        |

---

### **4. Transmission / BER Metrics**

| Metric Name              | Description                 | Metric_class (DME)      | Data Type      | Pipeline Layer | Usage                             |
| ------------------------ | --------------------------- | ----------------------- | -------------- | -------------- | --------------------------------- |
| `preFECBERAverage`       | Average pre-FEC BER         | `tpmon:FecReport`       | Uint32         | L3             | Early warning of link degradation |
| `preFECBERCurrent`       | Current pre-FEC BER         | `tpmon:FecReport`       | Uint32         | L3             | Anomaly trend detection           |
| `preFECBERMax` / `Min`   | Peak/min pre-FEC BER        | `tpmon:FecReport`       | Uint32         | L3             | Historical trend analysis         |
| `preFECBERPriorPeriod`   | Previous period pre-FEC BER | `tpmon:FecReport`       | Uint32         | L3             | Temporal analysis for forecasting |
| `fERAverage`             | Frame error rate average    | `tpmon:FecReport`       | Uint32         | L3             | Link performance baseline         |
| `fERCurrent`             | Current frame error rate    | `tpmon:FecReport`       | Float          | L3             | Real-time anomaly detection       |
| `fERMaximum` / `Minimum` | Peak/min FER                | `tpmon:FecReport`       | Uint32         | L3             | Input to risk scoring             |
| `rxPower`                | Received optical power      | `pie:TransceiverSensor` | Float (mW/dBm) | L2–L3          | Digital twin, physics modeling    |
| `txPower`                | Transmit optical power      | `pie:TransceiverSensor` | Float (mW/dBm) | L2–L3          | Digital twin, stress detection    |
| `SNR`                    | Signal-to-noise ratio       | `pie:TransceiverSensor` | Uint32         | L3             | Anomaly detection                 |

---

### **5. Module / Hardware Fault Metrics**

| Metric Name       | Description                 | Metric_class (DME)  | Data Type | Pipeline Layer | Usage                     |
| ----------------- | --------------------------- | ------------------- | --------- | -------------- | ------------------------- |
| `rxLOS` / `txLOS` | Loss of signal              | `tpmon:OpticsAlarm` | Uint32    | L3             | Immediate fault detection |
| `txAdaptEqFault`  | Tx adaptive equalizer fault | `tpmon:OpticsAlarm` | Uint32    | L3             | Signal anomaly            |
| `moduleFaults`    | Module-level faults         | `tpmon:OpticsAlarm` | Uint32    | L1             | Alert generation          |
| `txFault`         | Tx-side faults              | `tpmon:OpticsAlarm` | Uint32    | L3             | RUL input                 |
| `auxWarning`      | Module auxiliary warning    | `tpmon:OpticsAlarm` | Uint32    | L1             | Early warning             |
| `vendorWarning`   | Vendor-specific warning     | `tpmon:OpticsAlarm` | Uint32    | L1             | Vendor-defined thresholds |

---

### **6. Event & Timestamp Metrics**

| Metric Name    | Description             | Metric_class (DME)      | Data Type | Pipeline Layer | Usage                           |
| -------------- | ----------------------- | ----------------------- | --------- | -------------- | ------------------------------- |
| `timestamp`    | Event timestamp (epoch) | `pie:TransceiverSensor` | Uint32    | L0             | Ingestion / alignment           |
| `eventId`      | Event identifier        | `pie:PIE`               | Uint32    | L0             | Time-series correlation         |
| `healthStatus` | RCA health value        | `pie:OpticsHealth`      | Enum      | L1–L2          | Input to digital twin / physics |
| `reason`       | Event reason string     | `pie:OpticsHealth`      | String    | L1             | Logging / audit                 |

---

### **Integration Notes**

1. **Primary collector:** Cisco in-house telemetry collector for all DME streams.
2. **Fallback:** Telegraf gRPC collector if in-house collector unavailable.
3. **Usage in Pipeline:**

   * L0: Raw ingestion, timestamp alignment.
   * L1: Filtering and missing-value repair.
   * L2: Physics-informed state engine (thermal, electrical, optical).
   * L3: Anomaly detection, cross-device fusion.
   * L4–L5: Forecasting (Chronos, PathTST), classification (iTransformer).
   * L6: Digital Twin → RUL.
   * L7: Scheduling, NetOps & Nexus dashboard integration.


