# Delphi Price Oracle
**Technical & Governance Specification**

---

## 1. Conceptual Framework

The Delphi Price Oracle is the financial heart of the energy community. It functions as a **three-tier system**:

### 🌉 The Bridge (Technical)
Connects the physical world of IoT telemetry (electrons) to the financial world of accounting. It validates 15-minute energy intervals and assigns each one a precise monetary value.

### ⚖️ The Balancer (Economic)
Solves the "timing" problem of renewables. Using a dynamic algorithm, it adjusts prices based on real-time supply and demand — incentivizing members to use energy when the community has a surplus.

### 🗳️ The Democratizer (Governance)
Removes centralized control. Every pricing rule and algorithm parameter is tied to a DAO Proposal ID, ensuring that the **community — not a developer — controls the economy**.

---

### 🌞 The "Sunny Wednesday" Scenario

> Imagine a sunny afternoon where solar production is at its peak but most members are away.

| Step | What Happens |
|------|-------------|
| **Oracle Sees** | A massive energy surplus (Coverage Ratio of 4.5x) |
| **Algorithm Acts** | Internal price automatically drops from €0.10 → €0.06 |
| **Result** | Smart appliances and EVs detect the price drop, begin charging, keeping green energy in the local grid and rewarding members with lower costs |

---

## 2. Energy Flow Mapping

The Oracle categorizes energy movements into specific **Flows** based on the community's business model. Every 15-minute interval is mapped to one of the types below to determine the applicable pricing logic.

| # | Flow Source | Flow Destination | Price Type | Calculation Logic | Default (€) |
|---|-------------|-----------------|------------|-------------------|-------------|
| 1 | Prosumer | Aggregator | `ProsumerCommunityPrice` | Algorithmic: Weighted average generation | 0.10 |
| 2 | Retailer | Prosumer | `ProsumerRetailerImportPrice` | Market: Spot market or Retailer Tariffs | 0.10 |
| 3 | Prosumer | Retailer | `ProsumerRetailerExportPrice` | Market: Spot market or Retailer Tariffs | 0.10 |
| 4 | Prosumer | Battery (In) | `ProsumerBatteryChargePrice` | Algorithmic + Premium: Base Price + Storage Fee | 0.10 |
| 5 | Prosumer | Battery (Out) | `ProsumerBatteryDischargePrice` | Algorithmic + Premium: Base Price + Storage Fee | 0.10 |
| 6 | Prosumer | DSO | `ProsumerDSONetworkFee` | Fixed: Situational fee (distance/station) | 0.10 |
| 7 | Prosumer | Energy Tax | `ProsumerNationalEnergyTax` | Fixed: Regulatory tax per interval | 0.10 |
| 8 | Community | Members | `CommunityGenerationPrice` | Algorithmic: Weighted average community supply | 0.10 |

---

## 3. The Algorithmic Engine

The engine calculates the **Effective Price (`P_eff`)** for each interval using the community's **Coverage Ratio (ρ)**.

### A. Coverage Ratio (ρ)

Represents the self-sufficiency of the community in a 15-minute block:

```
ρ = Total Community Generation (kW) / Total Community Consumption (kW)
```

### B. Price Function

The price is determined by a **piecewise function** governed by DAO-voted coefficients:

| Coefficient | Description |
|-------------|-------------|
| `P_base` | Target price for a balanced community |
| `α` (Alpha) | Rate of price **decrease** when supply > demand (Surplus) |
| `β` (Beta) | Rate of price **increase** when demand > supply (Deficit) |

**Algorithm:**

```
If ρ ≥ 1 (Surplus):  P_eff = P_base - (α × (ρ - 1))
If ρ < 1 (Deficit):  P_eff = P_base + (β × (1 - ρ))
```

---

## 4. Technical Database Structure

### A. Core SQL Schema

The Oracle uses a dedicated `oracle` schema to manage proposals, pricing parameters, and market feeds.

```sql
-- Governance Link: Connects prices to blockchain votes
CREATE TABLE oracle.proposals (
    proposal_id     TEXT PRIMARY KEY,
    community_id    TEXT NOT NULL,
    status          TEXT DEFAULT 'passed',
    enacted_at      TIMESTAMPTZ DEFAULT NOW()
);

-- Algorithmic Parameters
CREATE TABLE oracle.price_algorithms (
    algo_id         BIGSERIAL PRIMARY KEY,
    community_id    TEXT NOT NULL,
    base_price      NUMERIC(10, 4) NOT NULL,
    surplus_alpha   NUMERIC(10, 4) NOT NULL,
    deficit_beta    NUMERIC(10, 4) NOT NULL,
    storage_premium NUMERIC(10, 4) DEFAULT 0,
    source_proposal TEXT REFERENCES oracle.proposals(proposal_id)
);

-- External Market Feeds (Spot Prices)
CREATE TABLE oracle.market_prices (
    time            TIMESTAMPTZ NOT NULL,
    community_id    TEXT NOT NULL,
    price_type      TEXT NOT NULL, -- 'SPOT_MARKET', 'RETAILER_TARIFF'
    price_per_kwh   NUMERIC(10, 4) NOT NULL
);
```

### B. Settlement Logic (SQL View)

The `oracle.settlement_final` view acts as the **Single Source of Truth** for financial reporting.

```sql
CREATE OR REPLACE VIEW oracle.settlement_final AS
WITH community_state AS (
    -- Aggregating total production vs consumption for the interval
    SELECT 
        interval_start,
        community_id,
        SUM(CASE WHEN direction = 'generation' THEN interval_kwh ELSE 0 END) AS total_gen,
        SUM(CASE WHEN direction = 'consumption' THEN interval_kwh ELSE 0 END) AS total_cons
    FROM accounting.interval_readings
    GROUP BY 1, 2
)
SELECT 
    i.interval_start,
    i.community_id,
    i.meter_id,
    i.direction,
    i.interval_kwh,

    -- Calculation Engine
    CASE 
        -- 1. Dynamic Market Pricing for Retailer Flows
        WHEN i.direction IN ('retailer_import', 'retailer_export')
            THEN m.price_per_kwh

        -- 2. Algorithmic Pricing for Community Flows
        ELSE (
            CASE 
                WHEN (s.total_gen / NULLIF(s.total_cons, 0)) >= 1.0 
                    THEN a.base_price - (a.surplus_alpha * ((s.total_gen / NULLIF(s.total_cons, 0)) - 1))
                ELSE
                    a.base_price + (a.deficit_beta * (1 - (s.total_gen / NULLIF(s.total_cons, 0))))
            END
        ) + (CASE WHEN i.direction LIKE '%battery%' THEN a.storage_premium ELSE 0 END)
    END AS effective_price,

    a.source_proposal AS auth_proposal_id

FROM accounting.interval_readings i
JOIN community_state s
    ON i.interval_start = s.interval_start
    AND i.community_id = s.community_id
JOIN oracle.price_algorithms a
    ON i.community_id = a.community_id
LEFT JOIN oracle.market_prices m
    ON i.interval_start = m.time
    AND i.community_id = m.community_id;
```

---

## 5. Governance Workflow

The Oracle is **"Proposal-Locked"** to ensure full transparency. Every pricing change follows this lifecycle:

```
Draft → Vote → Enactment → Audit
```

| Stage | Description |
|-------|-------------|
| **1. Draft** | A new pricing strategy is proposed in the DAO (e.g., *"Increase Alpha to reward daytime usage"*) |
| **2. Vote** | Community members vote on the blockchain |
| **3. Enactment** | Once passed, the `proposal_id` and new parameters are updated in the Oracle database |
| **4. Audit** | Every bill contains an `auth_proposal_id`, allowing any member to verify their price was calculated according to the community's democratic will |