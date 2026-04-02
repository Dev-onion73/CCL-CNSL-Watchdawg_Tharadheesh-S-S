# L4 to L5 — Temporal Forecasting & Risk Classification

## **Overview**
Layers **L4 and L5** transform anomaly signals into future health forecasts and risk scores, enabling early intervention.

---

## **L4 — Temporal Forecasting**

**Role:** Predict future transceiver states.

**Models:**
- **Chronos:** Multi-step attention-based temporal forecasting.
- **PathTST:** Learns cross-variable temporal dependencies across transceivers.

**Input:** L3 aggregated anomaly vectors, historical metrics from feature store.

**Output:** Multi-step forecasts for health parameters.

**Example:**  
L3 anomaly vector `[0.12, 0.14]` → L4 predicts 24-hour trajectory:  
`temperature = [42.5, 42.7, 42.9]`, `preFECBER = [12, 15, 18]`.

---

## **L5 — Risk Classification**

**Role:** Convert forecasts into discrete risk scores.

**Model:**
- **iTransformer:** Integrates L4 forecasts for per-device risk assessment.

**Processing:**
1. **Feature Projection:** Map L4 outputs to classification space.
2. **Classifier Head:** Assign risk class (low / medium / high).

**Example:**  
Forecasts: `preFECBER = [12, 15, 18]` → risk score = 0.65 → classified as **Moderate Risk**.

**Outputs:** Risk-scored telemetry for RUL inference (L6).

**Summary Table:**

| Layer | Input | Model | Output |
|-------|-------|-------|--------|
| L4 | L3 anomaly vector | Chronos, PathTST | Multi-step health forecasts |
| L5 | L4 forecasts | iTransformer | Risk scores per transceiver / cluster |
