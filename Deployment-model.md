# Deployment Model

## Layered Architecture

We use **VM-based clusters** with a hierarchical pooling strategy:

| Layer | Deployment Scope | Cluster Strategy | Notes |
|-------|----------------|----------------|------|
| L0–L2 | Per rack edge node | 1:1 mapping with physical device cluster | Ensures immediate telemetry ingestion and physics computation |
| L3 | Cluster of multiple L0–L2 racks | Partial pooling | Aggregates residual and precursor streams across racks |
| L4–L5 | Room-level inference | Cluster compute nodes | Temporal inference and risk classification scaled horizontally |
| L6 | Centralized Digital Twin | High-power compute cluster | Multi-class RUL estimation, model simulations |
| L7 | Scheduler | Global orchestration VM pool | Integrates outputs, generates maintenance schedules |

---

## Scalability & Cost Efficiency

- Horizontal scaling: Add VM pools per additional rack/cluster
- Vertical scaling: Assign more CPU/memory per compute node for L4–L6
- Energy optimization: Batch telemetry processing, conditional model execution
- Cost efficiency: Each VM pool tailored to expected workload and sensor density

---

## Phase-Wise Deployment

1. **Phase 1:** QSFP-DD transceivers; validate pipeline (hot-swappable, safe blast radius = one link)
2. **Phase 2:** PSUs; monitor capacitor aging, baseline drift (N+1 redundancy)
3. **Phase 3:** Cooling fans; mechanical wear feeding thermal model
4. **Phase 4:** Nexus switching ASICs; solder fatigue; pipeline fully validated

---

### Telemetry Handling

- Primary: In-house long-term telemetry store  
- Fallback: Telegraf gRPC collector, only if primary unavailable
- L0 directs both to L1–L2 for preprocessing
