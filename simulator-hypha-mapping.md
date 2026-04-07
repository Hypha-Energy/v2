# Simulator to Hypha Energy Integration Mapping

This document outlines the field definitions and mapping logic for integrating simulation data from the Elysium web app into the energy community token flow.

## 1. Elysium API Output Fields

The following fields represent the core data required to initialize an energy community based on simulation results.

### Hardware Configuration
* **`pv_capacity_kwp`**: Total peak power of the solar installation (e.g., 50.0).
* **`battery_storage_kwh`**: Total usable energy storage capacity (e.g., 20.0).
* **`inverter_power_kw`**: Total power rating of the system inverters.
* **`asset_details`**: Array of specific component metadata (brand, model, efficiency).

### Financial Data
* **`total_capex_euro`**: Total upfront investment required for hardware and installation.
* **`annual_opex_euro`**: Estimated yearly costs for maintenance and insurance.
* **`lcoe_euro_kwh`**: Levelized Cost of Energy (the baseline generation cost over the asset's life).

### Performance Metrics
* **`projected_annual_gen_kwh`**: Total estimated energy production for the first year.
* **`self_consumption_ratio`**: Percentage of energy consumed locally versus exported.
* **`export_limit_kw`**: Maximum grid export capacity allowed at the site.

---

## 2. Token Flow Mapping Logic

The simulation data from Elysium maps directly to the capitalization and operational layers of the token model.

### Capitalization (Eparts & Ebonds)
* **Source**: `total_capex_euro`
* **Mapping**: The CAPEX is used to determine the total value of tokens issued to fund the installation.
* **Logic**: 
    * [cite_start]**Eparts**: Represent participation and ownership rights[cite: 3]. [cite_start]These are issued to members and participants in exchange for capital[cite: 3].
    * [cite_start]**Ebonds**: Debt financier contributions are mapped to bond tokens[cite: 3, 4].

### Asset Initialization
* **Source**: `pv_capacity_kwp`, `battery_storage_kwh`
* **Mapping**: These fields define the **Energy Assets** block.
* [cite_start]**Logic**: The hardware is purchased on the global market using the raised capital (EUROC)[cite: 4]. [cite_start]These physical assets provide the economical consumption value recorded in the system[cite: 6].

### Operational Generation (Nano-PPAs)
* **Source**: `projected_annual_gen_kwh`
* **Mapping**: Sets the minting ceiling for **Nano-PPAs**.
* [cite_start]**Logic**: Assets generate 1 kWh of value per 15-minute interval, which is calculated and distributed as Nano-PPAs[cite: 6].

### Pricing & Returns
* **Source**: `lcoe_euro_kwh`, `self_consumption_ratio`
* **Mapping**: Used to set the **Cost-Price-Plus** logic.
* **Logic**: 
    * [cite_start]Members return 1 Nano-PPA per kWh consumed and pay a "cost-price-plus" rate[cite: 8]. 
    * The `lcoe_euro_kwh` provides the "cost" baseline for this calculation.
    * [cite_start]Excess energy not used for self-consumption is exported to the grid[cite: 9].
    * [cite_start]Income from grid exports and local consumers provides the returns for external investors[cite: 10].

---

## 3. Integration Workflow

1.  **Simulate**: Run the hardware and financial setup in Elysium to generate the report.
2.  [cite_start]**Onboard**: Import the Elysium API output into the onboarding tool to assist in deciding token distribution options[cite: 1].
3.  [cite_start]**Tokenize**: Automatically calculate the required supply of Eparts and Ebonds to cover the simulated CAPEX[cite: 3].
4.  [cite_start]**Monitor**: Connect smart meters to track real-time generation against the simulated `projected_annual_gen_kwh` to trigger Nano-PPA minting[cite: 12].
