# Elysium to Hypha Energy Integration Mapping 

This document defines the data structures and mapping logic for integrating Elysium’s high-fidelity simulation features into the Hypha Energy token flow ecosystem.

## 1. API Output Fields based on engineering report

The following fields incorporate Elysium’s advanced simulation capabilities, transitioning from static estimates to dynamic, hourly profiles.

### Dynamic Performance & Load Profiling
* **`hourly_production_8760`**: An array of 8,760 values representing projected solar/battery output for every hour of the year.
* **`load_profile_category`**: The specific consumer pattern generated via Elysium’s **Load Builder** (e.g., Residential, Industrial, or Mixed-Use).
* **`weather_pattern_id`**: The meteorological data set used, allowing the Hypha system to calibrate real-time performance expectations.

### Advanced Economic Modeling
* **`optimized_lcoe_euro_kwh`**: The minimized Levelized Cost of Energy after Elysium’s automated component sizing.
* **`simulated_payback_years`**: The projected timeline for full capital recovery, used to set financial milestones.
* **`grid_tariff_schedule`**: The local grid rates used for calculating the delta between P2P sharing and grid export revenue.

### Hardware Configuration
* **`optimized_pv_kwp` / `optimized_battery_kwh`**: The ideal hardware sizes suggested by Elysium’s **Intelligent Optimization**.

---

## 2. Strategic Mapping to Hypha Flow

The enriched data from Elysium automates several key components of the Hypha Energy model.

### Capitalization (Ebonds & Eparts)
* **Logic**: Use the **Economic Modeling** data to automate token issuance.
* **Mapping**: 
    * **Ebonds**: The `simulated_payback_years` and `annual_opex_euro` define the maturity and interest rate for debt tokens.
    * **Eparts**: The remaining CAPEX not covered by Ebonds is tokenized as equity shares for members and participants.

### Operational Minting (Nano-PPAs)
* **Logic**: Move from flat monthly estimates to hourly minting ceilings.
* **Mapping**: The **`hourly_production_8760`** profile acts as the oracle baseline for the minting of **Nano-PPAs** (1 kWh per 15m contracts).
* **Benefit**: This ensures that the availability of consumption tokens in the app matches the simulated physical availability of energy.

### Pricing (Cost-Price-Plus)
* **Logic**: Establish a transparent, data-backed price for members.
* **Mapping**: The **`optimized_lcoe_euro_kwh`** serves as the "cost" foundation.
* **Benefit**: Members pay this base cost plus a predefined margin (in EUROC) to ensure the cooperative's sustainability and investor returns.

### Revenue Sharing & Grid Interaction
* **Logic**: Automate the distribution of income based on Elysium’s **Energy Flow Optimization**.
* **Mapping**: The ratio of local consumption vs. grid export determines the flow of EUROC into the **Revenue Sharing Agreement**. 
* **Outcome**: High self-consumption rates identified by Elysium directly increase the yield for `Eparts` holders.

---

## 3. Integration Workflow

1.  **Simulation Phase**: Use Elysium’s **Load Builder** and **8760-hour Simulation** to design the optimal community hardware setup.
2.  **Onboarding Phase**: The **Hypha Onboarding Tool** ingests the Elysium API output to suggest the ideal split between `Eparts` and `Ebonds`.
3.  **Deployment Phase**: Smart contracts are initialized with the `hourly_production_8760` profile and `optimized_lcoe` as immutable parameters.
4.  **Operational Phase**: Real-time smart meter data is compared against the Elysium simulation to trigger automated payouts and Nano-PPA distribution.
