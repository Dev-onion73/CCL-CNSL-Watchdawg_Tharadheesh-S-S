# Telemetry & Metrics for Predictive Health Monitoring

## Sources

- **Primary:** Cisco in-house telemetry collector  
- **Fallback:** Telegraf gRPC collector pool  
- L0 connects to both, prioritizing in-house collector

---

## Metrics and DME Schemas

**QSFP-DD transceivers:**

- **Temperature metrics:** `temperature`, `laserTemperature`
- **Laser bias:** `biasCurrent`
- **Power metrics:** `txPower`, `rxPower`
- **Signal quality:** `preFECBER`, `postFECBER`

**Why these metrics?**

- Laser temperature → thermal stress modeling (L2)  
- Bias current → aging accumulation (L2)  
- Tx/Rx power → optical degradation detection (L2, L3)  
- FEC BER → residual errors for anomaly aggregation (L3)

---

## Integration into Pipeline

- L0 ingests raw telemetry → L1 cleans & aligns  
- L2 derives physics states → L3 aggregates anomalies  
- Data continuously stored for L6 digital twin simulations and historical feature analysis
