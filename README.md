# Hypha Energy v2

This repository contains the architecture, specifications, and domain documentation for the Hypha Energy v2 platform — a community energy system built on IoT telemetry, fair-split settlement, and blockchain governance.

---

## 📚 Documentation

### [Dictionary.md](./Dictionary.md)
The **Ubiquitous Language Dictionary** for the entire platform. Defines every domain term used across smart contracts, database schemas, APIs, and team conversations — from `Coverage Ratio` and `Fair-Split Algorithm` to `EnergySourceToken` and `Zero-Sum Invariant`. Start here if you are new to the project.

---

### [PriceOracle.md](./PriceOracle.md)
The **technical and governance specification for the Delphi Price Oracle** — the system that assigns a monetary value to every 15-minute energy interval. Covers the three-tier conceptual model (Bridge, Balancer, Democratizer), the Algorithmic Engine (Coverage Ratio, Alpha/Beta coefficients), the SQL schema (`oracle.*`), the settlement view, and the DAO governance workflow.

---

### [hypha-energy-v2-infrastructure-flow-responsibility-matrix.md](./hypha-energy-v2-infrastructure-flow-responsibility-matrix.md)
The **infrastructure plan and responsibility matrix** for the v2 POC. Defines the ownership split between Zek (data pipeline: edge, MQTT, ingestion, TimescaleDB, Price Oracle Worker) and Vlad (value engine: VPP logic, EMS, on-chain settlement). Includes a full component-by-component matrix comparing POC (Azure/C#) versus production paths (AWS and Azure).

---

### [simulator-hypha-mapping-v1.md](./simulator-hypha-mapping-v1.md)
The **Elysium-to-Hypha integration mapping**. Describes how data from the Elysium high-fidelity simulation platform (hourly production profiles, LCOE, payback years) maps to Hypha Energy's token model — automating Ebond/Epart issuance, Nano-PPA minting schedules, and the Cost-Price-Plus pricing baseline.