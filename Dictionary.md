# Ubiquitous Language Dictionary
**Hypha Energy v2 — Domain-Driven Design**

> This dictionary defines the shared language used across all technical, governance, and business conversations in the Hypha Energy platform. All team members, smart contracts, database schemas, and API names should use these terms consistently.
>
> **Sources:** `PriceOracle.md`, `hypha-energy-v2-infrastructure-flow-responsibility-matrix.md`, `simulator-hypha-mapping-v1.md`, and the canonical `ENERGY_SYSTEM_README.md` smart contract specification.

---

## A

**Aggregator**
The community operator entity (Hypha Energy) that facilitates energy flows between all community participants. On-chain, the Aggregator collects an `aggregatorFeeBps` percentage of all LOCAL revenue each settlement interval. Distinguished from the community treasury, which collects its own separate fee.

**Aggregator Fee (`aggregatorFeeBps`)**
A percentage of LOCAL source revenue (expressed in basis points) collected by the Aggregator (Hypha Energy operator) each interval. Deducted from the revenue pool before distributing earnings to source owners.
- Example: `aggregatorFeeBps = 300` → 3% of all solar + battery revenue goes to Hypha Energy.

**Algorithmic Engine**
The pricing component within the Price Oracle that calculates the `Cost Price Plus` for community energy flows using the Coverage Ratio and DAO-voted coefficients (Alpha, Beta, Base Price). Distinct from Market Pricing, which is used for Retailer flows.

**Alpha (α)**
A DAO-voted coefficient that controls how fast the internal price *decreases* when the community is in Surplus (`ρ ≥ 1`).
```
P_eff = P_base - (α × (ρ - 1))
```

---

## B

**Balancer (The)**
The conceptual economic role of the Price Oracle. It solves the renewable energy timing problem by adjusting prices based on real-time Coverage Ratio — incentivizing consumption during Surplus and conservation during Deficit.

**Base Price (`P_base`)**
The target price per kWh for a perfectly balanced community (Coverage Ratio = 1.0). All Algorithmic Engine adjustments are calculated relative to this value. Set by DAO vote and stored in `oracle.price_algorithms`.

**Basis Points (bps)**
The unit used to express ownership fractions and fee percentages in smart contracts. `10,000 bps = 100%`. Used in `ownershipBps`, `communityFeeBps`, and `aggregatorFeeBps` to avoid floating-point arithmetic on-chain.
- Example: Alice owns 30% of the solar park → `ownershipBps = 3000`.

**Battery**
A shared community energy storage asset. Has its own ownership table (independent from solar) and its own PPA price, which includes a markup for wear and storage losses. Controlled exclusively by the EMS via MQTT commands. Its meter reports `production` direction when discharging.

**Battery Charge Price (`ProsumerBatteryChargePrice`)**
The price applied when energy flows from a Prosumer or the grid into the Battery. Set in the Battery's PPA configuration on the blockchain. Includes a Storage Premium on top of the solar base price.

**Battery Discharge Price (`ProsumerBatteryDischargePrice`)**
The price applied when the Battery discharges energy to consuming members. Set in the Battery's PPA configuration. Priced higher than solar to cover battery wear and storage costs.

**Beta (β)**
A DAO-voted coefficient that controls how fast the internal price *increases* when the community is in Deficit (`ρ < 1`).
```
P_eff = P_base + (β × (1 - ρ))
```

**Bridge (The)**
The conceptual technical role of the Price Oracle. Connects the physical world of IoT telemetry (electrons measured in kWh) to the financial world of settlement (values in € / EURC).

---

## C

**Community**
A group of energy Members (consumers, Prosumers, and investors) sharing physical infrastructure and governed by a DAO. The fundamental organizational unit of the Hypha Energy platform. Identified by `community_id` in all schemas.

**Community Fee (`communityFeeBps`)**
A percentage of LOCAL source revenue (expressed in basis points) directed to the community treasury each interval. Deducted from the revenue pool before distributing earnings to source owners.
- Example: `communityFeeBps = 500` → 5% of all solar + battery revenue goes to the community treasury.

**Community Treasury**
The on-chain Safe (multisig) account that accumulates Community Fee income over time. Managed by the community governance structure.

**ConsumptionReading**
The on-chain data structure passed to `consumeEnergy()` / `settleInterval()` per settlement entry. Contains `deviceId`, `quantity` (energy amount), `pricePerKwh`, and `Source` enum. One member may have multiple `ConsumptionReading` entries per interval — one per energy source type.

```solidity
struct ConsumptionReading {
    uint256 deviceId;
    uint256 quantity;
    uint256 pricePerKwh;
    Source  source;  // LOCAL or IMPORT
}
```

**Coverage Ratio (ρ)**

- Surya suggested Actual Coverage RAtio and Predicted .
The ratio of total community generation to total community consumption within a single 15-minute Interval. The primary input to the Algorithmic Engine.
```
ρ = Total Community Generation (kWh) / Total Community Consumption (kWh)
```
- `ρ ≥ 1` → **Surplus** → price decreases via Alpha (α)
- `ρ < 1` → **Deficit** → price increases via Beta (β)

---

## D

**DAO (Decentralised Autonomous Organisation)**
The governance body of the community. All pricing parameters, algorithm coefficients, fee structures, and policy changes must be approved through a DAO Proposal and blockchain vote before they can be enacted in the Oracle.

**DAO Proposal**
A formal governance item submitted to the community for a blockchain vote. Every active pricing rule in the Oracle must reference a passed `proposal_id`. No pricing parameter is valid without this governance link. Lifecycle: `Draft → Vote → Enactment → Audit`.

**Deficit**
A community state where consumption exceeds generation (`ρ < 1`). Triggers a price increase via Beta (β) to incentivize reduced consumption or increased generation.

**Delphi Price Oracle**
The core system responsible for assigning a monetary value to every validated 15-minute Interval. Fulfils three conceptual roles: the Bridge (technical), the Balancer (economic), and the Democratizer (governance). The `oracle.settlement_final` view is its primary output.

**Democratizer (The)**
The conceptual governance role of the Price Oracle. Every pricing rule is locked to a DAO Proposal ID, ensuring the community — not any developer — controls the economy.

**Device**
An on-chain identity representing a physical meter point. A Member may own multiple devices. Mapped on-chain via `deviceToMember[]`. Identified by `deviceId` (uint256) in smart contract calls.

**Direction**
The attribute on a meter reading that categorises the type of energy flow:
- `consumption` — member is consuming electricity
- `production` — source (solar, battery discharging) is generating electricity
- `import` — energy flowing in from the external grid

**DSO (Distribution System Operator)**
The regulated utility company that operates the physical electricity network. Charges a fixed Network Fee (`ProsumerDSONetworkFee`) for use of its infrastructure.

---

## E

**Ebond**
A debt token representing a fixed-income instrument in the community's capital structure. Maturity and interest rate are derived from `simulated_payback_years` and `annual_opex_euro` from the Elysium simulation. Investors receive periodic interest payments.

**Cost Price Plus (`P_eff`)**
The final calculated price per kWh for a given Interval, as output by the Algorithmic Engine. Used as the settlement price for community energy flows.
```
Surplus (ρ ≥ 1):  P_eff = P_base - (α × (ρ - 1))
Deficit (ρ < 1):  P_eff = P_base + (β × (1 - ρ))
```

**[Elysium](https://design.hypha.energy/)**
The third-party high-fidelity energy simulation platform.

**EMS (Energy Management System)**
The software component that physically controls

**Energy Credit**
The positive-balance on-chain representation of what the community owes a member. Stored as an ERC-20 token (`EnergyToken`) visible in any crypto wallet. Earned when a member's ownership revenue exceeds their consumption charges within an interval.

**Energy Source**
A discrete supply of electricity in the community with its own price and ownership table. Each source is independently accounted for in the three-pass algorithm. Current source types:
- `LOCAL` — solar park (cheapest, first allocated)
- `BATTERY` — community battery (second allocated, higher price for wear)
- `IMPORT` — external grid (last resort, most expensive)

**EnergyPPAImplementation**

--Edgar thinks i t is NFT managed my smart contract, managing state changes.---
The primary smart contract (Solidity, UUPS proxy pattern) that stores all member balances, ownership configuration, and settlement logic. Exposes `consumeEnergy()` for VPP settlement submissions and `settleOwnDebt()` for member debt repayment.

**EnergySettlement**
A standalone helper smart contract that handles EURC stablecoin transfers for debt settlement. Members call `settleOwnDebt(eurcAmount)` or `settleDebt(debtor, eurcAmount)` to repay negative balances with real money.

**Epart**
An equity token representing an ownership share in the community's energy assets. The CAPEX not covered by Ebonds is tokenized as Eparts for members and investors. Holders earn revenue proportional to their ownership share each settlement interval.

**EURC**
The Euro-pegged stablecoin (issued by Circle, natively on Base) used for all on-chain financial settlements. 6 decimal places. Conversion to internal contract units: `internal_amount = eurc_amount / 10,000`.

**Export**
The sale of surplus community energy back to the external grid when production exceeds consumption. The Export device is treated as a consuming entity in the VPP settlement — its revenue (at the feed-in tariff price) enters the LOCAL revenue pool and is distributed to solar owners alongside consumption charges.

**Export Price (`gridExportPriceCt`)**
The feed-in tariff price paid by the grid to the community for exported surplus energy. Sourced from external APIs. Lower than the Import Price by design.

---

## F

**Fair-Split Algorithm**
The three-pass algorithm executed by the VPP every 15 minutes to determine each member's energy allocation and cost. Replaces the previous sequential (order-dependent) on-chain settlement logic. Guarantees that a member's bill depends only on their consumption, their ownership, and available energy — never on their position in an array.

**Feed-In Tariff**
See *Export Price*.

---

## G

**Governance Workflow**
The mandatory lifecycle for all Oracle pricing changes: `Draft → Vote → Enactment → Audit`. No pricing parameter can be changed without completing this workflow and associating a `proposal_id`.

**Grafana**
The monitoring and observability platform connected to TimescaleDB and PostgreSQL. Monitors meter health (via `reading_count` gaps), interval summaries, and settlement status. Alerts on missing readings, failed settlements, zero-sum violations, and low gas balance.

**Grid**
The national electricity network. Used as the energy source of last resort when community solar + battery production cannot cover total consumption. The grid connection has its own smart meter with `direction = 'import'`. Grid energy is the most expensive source.

**Grid Import Price (`gridImportPriceCt`)**
The price per kWh for energy imported from the external grid. Sourced from external retail tariff or spot market APIs. Applied only after all LOCAL and BATTERY sources are exhausted (Pass 3 of the Fair-Split Algorithm).
- Different components, Peak Tariff, Taxes , Suriya will update
---

## H

**Handover DB**
The PostgreSQL/TimescaleDB database instance that serves as the boundary between the data pipeline (raw ingestion and aggregation) and the VPP/EMS logic.

**HiveMQ Cloud**
The managed MQTT broker platform used for telemetry from edge meters. Supports TLS client certificate authentication per meter. Free tier supports fewer than 100 devices.

**Hourly Production Profile (`hourly_production_8760`)**
An array of 8,760 values (one per hour of the year) from the Elysium simulation representing projected solar/battery output. Used as the oracle baseline for Nano-PPA minting — ensuring token availability matches the simulated physical availability of energy.

**Hypha Onboarding Tool**
Internal tooling that ingests Elysium API output to suggest the optimal split between Eparts and Ebonds for a new community project and to initialize smart contract parameters.

---

## I

**Import Balance**
An on-chain tracker for the community's cumulative electricity bill owed to the external grid. Charges that carry `source = IMPORT` accumulate here instead of entering the LOCAL revenue pool. Paid separately to the energy retailer outside the smart contract.

**Ingestion Service**
The always-on C# (or Node.js) service that subscribes to the MQTT broker and writes every meter message into `raw_readings`. Uses batched `COPY` inserts for performance. Handles MQTT QoS 1 duplicate delivery via `ON CONFLICT DO NOTHING`.

**Interval**
A 15-minute energy measurement window. The fundamental unit of time for all accounting, pricing, and settlement calculations. Boundaries fall at :00, :15, :30, and :45 of every hour.

**Interval Reading**
A validated, aggregated energy measurement for a specific meter within a specific 15-minute Interval. Stored permanently in `accounting.interval_readings` (or `interval_readings`). The primary input to the VPP Fair-Split Algorithm.

```sql
CREATE TABLE interval_readings (
    interval_start  TIMESTAMPTZ  NOT NULL,
    meter_id        TEXT         NOT NULL,
    community_id    TEXT         NOT NULL,
    energy_wh       INTEGER      NOT NULL,
    direction       TEXT         NOT NULL,
    reading_count   INTEGER      NOT NULL,
    PRIMARY KEY (interval_start, meter_id)
);
```

**Investor**
A Member who holds ownership in one or more community energy sources but has no physical meter and does not consume electricity. Earns revenue purely from ownership share. Example: Eve holds 10% solar + 25% battery but pays nothing for consumption.

**IoT Hub (Azure)**
The Azure IoT Hub instance retained as a strategic asset for future non-MQTT protocols (AMQP, HTTPS), industrial bridging (Modbus, OPC-UA via inverters), and Device Twin persistent state management. Not the primary broker for the current POC.

---

## L

**LCOE — Levelized Cost of Energy (`optimized_lcoe_euro_kwh`)**
The minimized cost per kWh after Elysium's intelligent hardware sizing optimization. The transparent "cost" foundation for the Cost-Price-Plus pricing model.

**LOCAL (Source enum)**
The `Source` enum value in the smart contract representing any internally generated energy — solar park, battery discharge, or any community-owned generation asset. Revenue from LOCAL sources is pooled and distributed to that source's owners after fees.

**Load Profile (`load_profile_category`)**
The consumer behaviour pattern assigned by Elysium's Load Builder (e.g., Residential, Industrial, Mixed-Use). Used during simulation to stress-test accounting logic across diverse community compositions before hardware deployment.

---

## M

**Market Price**
An externally sourced price (Spot Market rate or Retailer Tariff) used to price grid import and export flows. Fetched by the Price Oracle Worker Service from external APIs and stored in `oracle.market_prices`.

**Member**
An individual or entity participating in the energy community. Members may be consumers, Prosumers, or pure Investors. Each member is identified by an Ethereum wallet address on-chain and by `meter_id` in the database pipeline.

**MemberPPA**
The on-chain data structure representing a member's participation configuration: `memberAddress`, `deviceIds[]`, `ownershipBps`, `isActive`, and `metadataHash`.

**Meter**
A physical or virtual smart device that measures instantaneous power flow (`powerW` in watts) at a specific point in the network. Sends MQTT readings every 10 seconds. Identified by `meter_id` in database schemas and `deviceId` on-chain.

**MQTT (Message Queuing Telemetry Transport)**
The lightweight publish-subscribe messaging protocol used by all smart meters to transmit readings. Topic pattern: `community/{communityId}/meter/{meterId}/reading`. QoS 1 (at-least-once delivery). Data is transient — held only in broker memory until consumed by the Ingestion Service.

---

## N

**Nano-PPA (Power Purchase Agreement)**
A micro-contract representing the right to consume 1 kWh of energy within a 15-minute window. Minted based on the `hourly_production_8760` Elysium simulation profile. Ensures token availability in the app matches the simulated physical availability of energy.

**National Energy Tax (`ProsumerNationalEnergyTax`)**
A fixed regulatory tax per Interval applied to applicable prosumer energy flows, as mandated by national legislation.

**Network Fee (`ProsumerDSONetworkFee`)**
A fixed fee charged by the DSO for use of the physical distribution network. May vary based on distance or substation proximity.

---

## O

**Ownership Fraction (`o(s,m)`)**
The decimal fraction representing member `m`'s ownership of source `s`. Used in all three passes of the Fair-Split Algorithm. Stored on-chain as `ownershipBps` (basis points). All ownership fractions for a given source must sum to 1.0 (10,000 bps).

**Oracle Schema**
The dedicated PostgreSQL schema (`oracle.*`) housing all Oracle-managed data: `oracle.proposals`, `oracle.price_algorithms`, `oracle.market_prices`, and the `oracle.settlement_final` view.

---

## P

**Pass 1 — Ownership Allocation**
The first step of the Fair-Split Algorithm. Each member is allocated their ownership percentage of each energy source (cheapest first: solar → battery). Members consume their own shares first; any share not consumed becomes Surplus.
```
share(s,m)   = o(s,m) × P(s)
used(s,m)    = min(share(s,m), remaining(m))
surplus(s,m) = share(s,m) - used(s,m)
deficit(m)   = remaining(m) after all sources
```

**Pass 2 — Surplus Redistribution**
The second step of the Fair-Split Algorithm. Per-source surplus is redistributed proportionally to deficit members who own a stake in that source. A member with no ownership in a source cannot receive its surplus.
```
pool(s)  = Σ_m surplus(s,m)
W(s)     = Σ_{m ∈ D(s)} o(s,m)   -- only deficit members with ownership
alloc    = pool(s) × o(s,m) / W(s)
```
This is the key mechanism that makes the system order-independent and fair.

**Pass 3 — Grid Import**
The third step of the Fair-Split Algorithm. Any remaining deficit after Passes 1 and 2 is covered by importing electricity from the external grid at `gridImportPriceCt`.
```
grid(m) = deficit(m)   -- for all m where deficit(m) > 0
```

**POC (Proof of Concept)**
The current sprint mindset: prioritise speed and validation over production hardening. Explicitly excluded from this phase: automated billing, ML load forecasting, fiat gateways, and hardened edge security.

**PPA (Power Purchase Agreement)**
An agreement that defines the price and terms for purchasing energy from a specific source. In Hypha Energy, PPA prices for LOCAL sources (solar, battery) are set when the community is created and stored on-chain in the smart contract's source registry. They do not change with market conditions.

**Price Algorithm**
A DAO-governed set of coefficients (`base_price`, `surplus_alpha`, `deficit_beta`, `storage_premium`) stored in `oracle.price_algorithms` and referenced by `source_proposal`. Used by the Algorithmic Engine to calculate the Effective Price.

**Price Oracle Worker Service**
The scheduled C# service (Azure WebJob / Azure Function) responsible for fetching external market prices (spot and retail tariffs) from external APIs and writing them to `oracle.market_prices`. Acts as the Source of Truth for market-rate variables.

**Proposal-Locked**
The design principle that every active pricing rule in the Oracle must reference a valid, passed `proposal_id`. No pricing parameter can change without completing the Governance Workflow.

**Prosumer**
A community Member who both produces (generates) and consumes electricity, typically through rooftop solar. Interacts with Aggregator, Retailer, Battery, DSO, and Tax flows.

---

## R

**Raw Readings**
High-frequency telemetry data captured from MQTT at 10-second intervals and stored in `raw_readings`. Retained for 90 days via TimescaleDB's retention policy, then automatically deleted. Input to the 15-minute Aggregation pipeline.

```sql
CREATE TABLE raw_readings (
    time          TIMESTAMPTZ  NOT NULL,
    meter_id      TEXT         NOT NULL,
    community_id  TEXT         NOT NULL,
    power_w       REAL         NOT NULL,
    direction     TEXT         NOT NULL
);
```

**Reading Count (`reading_count`)**
The number of 10-second MQTT messages aggregated into a single Interval Reading. Expected value: ~90 per meter per 15-minute interval. A count significantly below 90 indicates missing meter data or connectivity issues, triggering a Grafana alert.

**Retailer**
The external licensed electricity supplier or utility company connected to the wider national grid. Prosumers can import from or export to the Retailer when community supply/demand is imbalanced. Retailer prices are sourced from external Market Price APIs.

**Revenue Pool**
The aggregate monetary value collected from consuming members within an interval for a specific energy source. Distributed in order: Community Fee → Aggregator Fee → remainder to source owners by `ownershipBps`.

---

## S

**Settlement**
The process of calculating and recording the final monetary value of all energy flows within a 15-minute interval, leading to on-chain balance updates for all members. Executed by the VPP and recorded permanently on the Base blockchain.

**Settlement Batch (`settlement_batches`)**
The PostgreSQL record of exactly what was sent to the blockchain per interval, including the full `entries` JSON and transaction metadata. Kept permanently as a receipt. Status transitions: `pending → submitted → confirmed` or `failed`.

**Settlement Final (`oracle.settlement_final`)**
The SQL view that acts as the Single Source of Truth for financial reporting. Joins `interval_readings`, community state, `price_algorithms`, and `market_prices` to produce the Effective Price for each Interval.

**Single Source of Truth**
The principle that `oracle.settlement_final` (for the Delphi Oracle path) and `settlement_batches` (for the VPP path) are the authoritative records for financial settlement. No other calculation supersedes them.

**Smart Meter**
A physical IoT device attached to a wire at each energy point (member household, solar park, battery, grid connection). Measures instantaneous power flow in watts and transmits readings over MQTT every 10 seconds.

**Source Registry**
The on-chain store (in the v2 `EnergyPPAv2` contract) where each energy source's `EnergySourceToken` address and PPA price are stored transparently. The VPP reads this to determine prices and ownership fractions.

**Source Type**
The classification of an energy source used in `ConsumptionReading`. Current values:
- `LOCAL` — any community-owned generation (solar, battery)
- `IMPORT` — electricity purchased from the external grid
- `BATTERY` — used in off-chain VPP calculations to distinguish battery from solar within LOCAL

**Storage Premium**
An additional fee added on top of the Base Price for energy flows through the Battery asset. Covers battery wear, degradation, and storage losses. Governed by DAO vote and stored in `oracle.price_algorithms.storage_premium`.

**Surplus**
A community state where generation exceeds consumption (`ρ ≥ 1`), or the per-member condition where a member's ownership share of a source exceeds their own consumption needs. Surplus energy is redistributed in Pass 2 of the Fair-Split Algorithm.

**Sunny Wednesday Scenario**
The canonical illustrative example: on a sunny afternoon with peak solar production and low member presence, the Coverage Ratio spikes (e.g., ρ = 4.5), the internal price drops significantly, and smart appliances/EVs detect the low price and begin charging — keeping green energy in the local grid.

---

## T

**TimescaleDB**
The time-series PostgreSQL extension used for the Handover DB. Enables efficient hypertable storage, `time_bucket()` aggregation, and automatic retention policies for high-frequency meter data.

**Three-Pass Algorithm**
See *Fair-Split Algorithm*, *Pass 1*, *Pass 2*, *Pass 3*.

---

## V

**VPP (Virtual Power Plant)**
The backend software component (Node.js/TypeScript) that runs the Fair-Split Algorithm every 15 minutes.

---

## W

**Weather Pattern (`weather_pattern_id`)**
The meteorological dataset identifier used in the Elysium simulation. Allows the Hypha system to calibrate real-time generation performance expectations against the simulated baseline.

---

## Z

**Zero-Sum Invariant**
The mathematical guarantee enforced by the `ensureZeroSum` modifier on every smart contract settlement transaction. The sum of all member balances, import balance, export balance, and settled balance must equal exactly zero. If the VPP's math is incorrect, the entire transaction reverts on-chain.
```
Σ_m balance[m] + importBalance + exportBalance + settledBalance == 0
```
This makes it impossible to record a financially inconsistent settlement on the blockchain.

**Zero-Sum Principle**
The accounting rule that the total energy generated within the community must equal the total energy consumed plus exported within any given Interval. Validated at both the database level (Interval Readings) and the smart contract level (Zero-Sum Invariant).

**zigbee2mqtt**
The open-source bridge software deployed on the physical edge gateway to translate Zigbee radio signals from smart meters and IoT sensors into MQTT messages forwarded to the cloud broker.
