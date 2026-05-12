# Ubiquitous Language Dictionary
**Hypha Energy v2 — Domain-Driven Design**

> This dictionary defines the shared language used across all technical, governance, and business conversations in the Hypha Energy platform. All team members, smart contracts, database schemas, and API names should use these terms consistently.

---

## A

**Aggregator**
The community entity that acts as the central counterparty for energy flows between Prosumers. Receives energy sold by Prosumers into the community pool and distributes it to consuming members.

**Alpha (α)**
A DAO-voted coefficient that controls how fast the internal price *decreases* during a surplus. Part of the Algorithmic Engine's piecewise price function.

---

## B

**Base Price (`P_base`)**
The target price per kWh for a perfectly balanced community (Coverage Ratio = 1.0). All algorithmic price adjustments are calculated relative to this value. Governed by DAO vote.

**Battery Charge Flow**
An energy movement from a Prosumer to a community Battery asset. Priced using the Algorithmic Base Price plus a Storage Premium.

**Battery Discharge Flow**
An energy movement from a community Battery asset back to a Prosumer. Priced using the Algorithmic Base Price plus a Storage Premium.

**Beta (β)**
A DAO-voted coefficient that controls how fast the internal price *increases* during a deficit. Part of the Algorithmic Engine's piecewise price function.

**Bridge (The)**
The conceptual role of the Price Oracle that connects the physical world of IoT telemetry (electrons/kWh) to the financial world of accounting (€ values).

---

## C

**Community**
A group of energy members (Prosumers and consumers) sharing physical infrastructure and governed by a DAO. The fundamental organizational unit of the Hypha Energy model.

**Community Generation Price (`CommunityGenerationPrice`)**
The price applied to the aggregate energy supply provided by the community to its members. Calculated as a weighted average of community supply using the Algorithmic Engine.

**Coverage Ratio (ρ)**
The ratio of total community generation to total community consumption within a 15-minute Interval. Represents the community's real-time self-sufficiency.
```
ρ = Total Community Generation (kW) / Total Community Consumption (kW)
```
- `ρ ≥ 1` → Surplus state → price decreases
- `ρ < 1` → Deficit state → price increases

**Cost-Price-Plus**
The pricing philosophy for members: the `optimized_lcoe_euro_kwh` (Levelized Cost of Energy) serves as the transparent cost base, to which a predefined margin is added for cooperative sustainability.

---

## D

**DAO (Decentralised Autonomous Organisation)**
The governance body of the community. All pricing parameters, algorithm coefficients, and policy changes must be approved through a DAO Proposal vote before they can be enacted in the Oracle.

**DAO Proposal**
A formal governance item submitted to the community for a blockchain vote. Every active pricing rule in the Oracle must reference a passed Proposal ID, ensuring no parameter is changed without community consent.

**Deficit**
A community state where consumption exceeds generation (`ρ < 1`). Triggers a price increase via Beta (β) to incentivize reduced consumption or increased generation.

**Delphi Price Oracle**
The core system responsible for assigning a monetary value (€) to every validated 15-minute energy Interval. It is the Single Source of Truth for all financial settlement calculations. See also: *Bridge*, *Balancer*, *Democratizer*.

**Democratizer (The)**
The conceptual governance role of the Price Oracle. All pricing rules are locked to a DAO Proposal ID, removing centralized developer control and giving the community authority over its own economy.

**Direction**
The attribute of an energy reading that categorizes which type of Energy Flow it represents (e.g., `generation`, `consumption`, `retailer_import`, `retailer_export`, `battery_charge`, `battery_discharge`).

**DSO (Distribution System Operator)**
The regulated utility company that owns and operates the physical electricity network (poles, wires, transformers). Charges a fixed Network Fee for use of its infrastructure.

---

## E

**Ebond**
A debt token representing a fixed-income instrument in the community's capital structure. Maturity and interest rate are derived from `simulated_payback_years` and `annual_opex_euro` from the Elysium simulation.

**Effective Price (`P_eff`)**
The final calculated price per kWh for a given Interval, as output by the Algorithmic Engine. This is the value stored and used for all financial settlement.

**Elysium**
The third-party high-fidelity energy simulation platform. Its API output (e.g., `hourly_production_8760`, `optimized_lcoe_euro_kwh`) feeds into the Hypha Onboarding Tool and provides the economic baseline for token issuance and pricing.

**EMS (Energy Management System)**
The control logic layer (owned by Vlad) responsible for dispatching real-time commands to community assets (e.g., EVs, batteries, smart appliances) based on price signals from the Oracle.

**Epart**
An equity token representing an ownership share in the community's energy assets. The remaining CAPEX not covered by Ebonds is tokenized as Eparts for members and investors.

**EUROC**
The Euro-pegged stablecoin used for all on-chain financial settlements within the Hypha Energy platform.

---

## F

**Fair-Split Math**
The revenue distribution calculation (owned by Vlad) that determines how settlement proceeds from energy flows are allocated among community members proportionally.

---

## G

**Generation**
The production of electricity by a Prosumer's assets (e.g., solar panels). A `direction` value in the interval readings schema.

**Governance Workflow**
The mandatory lifecycle for all pricing changes: `Draft → Vote → Enactment → Audit`. No price parameter can be changed without completing this workflow.

**Grafana**
The monitoring and observability platform used to visualize community energy data and system health. Public dashboards expose key operational metrics.

---

## H

**Handover DB**
The PostgreSQL/TimescaleDB database instance that acts as the official boundary between Zek's data pipeline (ingestion) and Vlad's logic layer (VPP/EMS). The `interval_readings` table is the primary handover artifact.

**HiveMQ Cloud**
The MQTT broker platform used for standard telemetry heartbeat from edge devices at a 10-second frequency.

**Hypha Onboarding Tool**
The internal tooling that ingests Elysium API output to suggest the optimal split between Eparts and Ebonds for a new community project.

---

## I

**Ingestion Worker**
The C# service (Azure WebJob) responsible for capturing MQTT telemetry from the broker and writing it to the `raw_readings` table in the Handover DB.

**Interval**
A 15-minute energy measurement window. The fundamental unit of time for all accounting, pricing, and settlement calculations in the platform.

**Interval Reading**
A validated, summarized energy measurement for a specific Meter within a specific 15-minute Interval. Stored in `accounting.interval_readings` and used as the primary input to the Algorithmic Engine.

**IoT Hub (Azure)**
The Azure IoT Hub instance retained as a strategic asset for future non-MQTT protocol support (AMQP, HTTPS), industrial bridging (Modbus, OPC-UA), and Device Twin state management.

---

## L

**LCOE (Levelized Cost of Energy) — `optimized_lcoe_euro_kwh`**
The minimized cost per kWh as calculated by Elysium's simulation after intelligent hardware sizing. Serves as the transparent "cost" foundation for the Cost-Price-Plus pricing model.

**Load Profile**
A pattern describing a member's energy consumption behavior over time (e.g., Residential, Industrial, Mixed-Use). Used in simulation to stress-test accounting logic across diverse community compositions.

---

## M

**Market Price**
An externally sourced price (Spot Market rate or Retailer Tariff) used to price Retailer flows. Stored in `oracle.market_prices` and fetched by the Price Oracle Worker Service.

**Member**
An individual or entity participating in the energy community. Members may be pure consumers or Prosumers.

**Meter**
A physical or virtual device that measures energy flow at a specific point in the network. Identified by `meter_id` in all data schemas.

---

## N

**Nano-PPA (Power Purchase Agreement)**
A micro-contract representing the right to consume 1 kWh of energy within a 15-minute window. Minted based on the `hourly_production_8760` simulation profile as the oracle baseline.

**National Energy Tax (`ProsumerNationalEnergyTax`)**
A fixed regulatory tax applied per Interval to applicable energy flows, as mandated by national legislation.

**Network Fee (`ProsumerDSONetworkFee`)**
A fixed fee charged by the DSO for use of the distribution network. May vary based on distance or substation proximity.

---

## O

**Oracle Schema**
The dedicated PostgreSQL schema (`oracle.*`) that houses all Oracle-managed data: `oracle.proposals`, `oracle.price_algorithms`, and `oracle.market_prices`.

---

## P

**POC (Proof of Concept)**
The current sprint mindset. Prioritizes speed and validation over production-hardening. Certain features (automated billing, ML forecasting, fiat gateways, edge security) are explicitly excluded during this phase.

**Price Algorithm**
A set of DAO-governed coefficients (`base_price`, `surplus_alpha`, `deficit_beta`, `storage_premium`) stored in `oracle.price_algorithms` and used by the Algorithmic Engine to calculate the Effective Price.

**Price Oracle Worker Service**
The C# scheduled service responsible for fetching external market prices (spot and retail tariffs) and writing them to the Oracle database. Acts as the Source of Truth for market-rate variables.

**Proposal-Locked**
The design principle that every active pricing rule in the Oracle must reference a valid, passed `proposal_id` from the DAO. No pricing parameter is valid without this governance link.

**Prosumer**
A community member who both produces (generates) and consumes energy. Prosumers interact with the Aggregator, Retailer, Battery, DSO, and Tax flows.

**Prosumer Battery Charge Price (`ProsumerBatteryChargePrice`)**
Price for energy flowing from a Prosumer into a Battery asset. Calculated as Algorithmic Base Price + Storage Premium.

**Prosumer Battery Discharge Price (`ProsumerBatteryDischargePrice`)**
Price for energy flowing from a Battery asset to a Prosumer. Calculated as Algorithmic Base Price + Storage Premium.

**Prosumer Community Price (`ProsumerCommunityPrice`)**
Price for energy flowing from a Prosumer into the community Aggregator. Calculated as a weighted average generation price using the Algorithmic Engine.

**Prosumer Retailer Export Price (`ProsumerRetailerExportPrice`)**
Price for energy flowing from a Prosumer to the external Retailer grid. Set by Spot Market or Retailer Tariff.

**Prosumer Retailer Import Price (`ProsumerRetailerImportPrice`)**
Price for energy flowing from the external Retailer grid to a Prosumer. Set by Spot Market or Retailer Tariff.

---

## R

**Raw Readings**
Unprocessed telemetry data captured from MQTT at 10-second frequency and stored in the `raw_readings` table. Input to the 15-minute summarization pipeline.

**Retailer**
The external utility company or licensed electricity supplier connected to the wider grid. Prosumers can import from or export to the Retailer when community supply/demand is imbalanced.

**Revenue Sharing Agreement**
The mechanism by which income from energy flows (particularly grid exports) is distributed proportionally among Epart holders.

---

## S

**Settlement**
The process of calculating and recording the final monetary value of all energy flows within a period, used as the basis for on-chain payment in EUROC.

**Settlement Final (`oracle.settlement_final`)**
The SQL view that acts as the Single Source of Truth for all financial reporting. Joins interval readings, community state, price algorithms, and market prices to produce the Effective Price for each Interval.

**Single Source of Truth**
The principle that `oracle.settlement_final` is the only authoritative source for financial settlement data. No other system or calculation supersedes it.

**Simulator (Elysium / Existing Simulated Community)**
The data generator used during POC to produce diverse consumption and generation patterns, enabling stress-testing of the Energy Accounting logic before real hardware is deployed.

**Spot Market**
The real-time electricity exchange where energy is priced based on live supply and demand. One of the two sources for Market Prices used in Retailer flows.

**Storage Premium**
An additional fee added to the Base Price for energy flows involving a Battery asset, reflecting the cost of storage services. Governed by DAO vote.

**Surplus**
A community state where generation exceeds consumption (`ρ ≥ 1`). Triggers a price decrease via Alpha (α) to incentivize consumption of locally available green energy.

---

## T

**TimescaleDB**
The time-series extension to PostgreSQL used for the Handover DB, enabling efficient storage and querying of high-frequency energy interval data.

---

## V

**VPP (Virtual Power Plant)**
The aggregated control and dispatch logic (owned by Vlad) that coordinates the community's distributed energy assets to respond to grid signals or optimize community economics.

---

## Z

**Zero-Sum Principle**
The accounting rule that the total energy generated within the community must equal the total energy consumed plus exported, within any given Interval. Used to validate the integrity of interval readings.

**zigbee2mqtt**
The open-source bridge software deployed on the physical edge gateway to translate Zigbee radio signals from smart meters and IoT sensors into MQTT messages.
