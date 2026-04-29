# My Thoughts on v2 Infrastructure

---

## 1. Project Context & Reference
For the core logic regarding energy accounting and the smart contract architecture this infrastructure supports, please refer to:
* **[Energy System Readme (GitHub)](https://github.com/hypha-dao/hypha-web/blob/energy-accounting/packages/storage-evm/contracts/ENERGY_SYSTEM_README.md)**

---

## 2. The Execution Split
* **Zek (Foundation):** Owns the data lifecycle from the physical edge to the integration database, including the simulated community environment and the price synchronization services.
* **Vlad (Value Engine):** Owns the fair-split math, the EMS control logic, and the final on-chain settlement logic as defined in the technical spec.

---

## 3. Extended Implementation Plan (Zek's Scope)

### Step 0 & 0b: The Physical Edge & Simulator
I will coordinate with **R&C and Grexx** to configure the `zigbee2mqtt` bridge on the edge gateway. For the telemetry baseline, I am utilizing the **existing Simulated Community Data Generator**. Since it already features diverse consumption and generation patterns (diverse profiles for different types of members), we can immediately stress-test the **Energy Accounting** logic. This allows us to verify the zero-sum principle across varying community behaviors—like solar-heavy members versus high-load consumers—before hardware is fully deployed.

### Step A: The Price Oracle
I will implement a scheduled **C# Worker Service** to fetch real-time spot and retail energy prices. This service acts as the "Source of Truth" for the market variables, enabling the **Market Mechanism** to calculate revenue distribution based on actual market fluctuations.

### Step 1 & 2: Brokerage & Strategic Assets
The primary data backbone utilizes **HiveMQ Cloud** for standard MQTT telemetry heartbeat at a 10-second frequency. However, I am **keeping and parking the Azure IoT Hub instance** for future collaborations. 

While we lead with MQTT for speed, IoT Hub remains our strategic asset for:
* **Non-MQTT protocols** (AMQP, HTTPS).
* **Industrial bridging** (Modbus, OPC-UA) for future inverter integrations.
* **Device Twins** for persistent offline state management.

I am deploying a **C# Ingestion Worker** (via Azure WebJobs) to capture the heartbeat into the `raw_readings` table.

### Step 3: The Integration Point (Database)
I am responsible for the **Postgres/TimescaleDB** instance, which serves as our official handover line. My pipeline will perform a 15-minute scheduled summarization of `raw_readings` into an `interval_readings` table. This summary table is the primary input for Vlad’s **VPP Logic**.

---

## 4. Responsibility Matrix: POC vs. Production Options

| Step | Function | Owner | POC (Azure/C#) | Production (AWS Path) | Production (Azure Path) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **0** | **Physical Edge** | **Zek** | **zigbee2mqtt** | Managed IoT Gateway | Azure IoT Edge |
| **0b** | **Data Gen** | **Zek** | **Existing Patterns** | Real Meters | Real Meters |
| **A** | **Price Oracle** | **Zek** | **C# WebJob** | AWS Equivalent | Azure Functions |
| **1** | **MQTT Broker** | **Zek** | **EMQX Platform / IoT Hub** | HiveMQ/Mosquito + IoT Core | IoT Hub (S-Tier) |
| **2** | **Ingestion** | **Zek** | **EMQX Platform / IoT Hub** | AWS Equivalent | Azure Functions |
| **3** | **Handover DB** | **Zek** | **Postgres / Timescale** | AWS RDS (Managed) | Azure DB (Flexible) |
| **4** | **VPP Logic** | Vlad | Node.js | AWS Lambda | Azure Functions |
| **5** | **EMS Logic** | Vlad | Node.js | AWS Lambda | Azure Functions |
| **6** | **Blockchain** | Vlad | Base Chain | Base (KMS) | Base (Key Vault HSM) |
| **7** | **Settlement** | Vlad | Base Contracts | Embedded Wallets | Embedded Wallets |
| **M** | **Monitoring** | **Zek** | **Grafana Cloud** | Grafana Enterprise | Managed Grafana |

---

## 5. Exclusions (Not in this Sprint)
To maintain speed and a "POC Mindset," the following are strictly out of scope:
* **Automated Billing/Identity:** Manual reconciliation and flat-file records for now; automated systems are deferred.
* **Advanced Load Forecasting:** No predictive ML models for consumption/generation.
* **Fiat Gateways:** Settlements are restricted to on-chain assets (EURC).
* **Hardened Edge Security:** Enterprise-grade hardware security (HSM/TPM) is deferred.

---

## 6. To Be Discussed: Production Environment Selection
As we move from POC to Production, we need to decide between pivoting to **AWS** or staying on **Azure**. 

### **The AWS Argument**
* **Event-Driven Cost:** Using **AWS Lambda** for VPP/EMS logic means we pay only for the seconds the calculation runs (every 15m), rather than paying for idle time.
* **IoT Core Ecosystem:** AWS IoT Core provides a superior "Rules Engine" and automated X.509 certificate management for thousands of meters.
* **Security:** **AWS KMS** allows us to sign blockchain transactions without developers ever seeing the private keys.

### **The Azure Argument**
* **Ecosystem Synergy:** Tighter integration for our existing **C#/.NET** foundation and WebJobs.
* **Unified Management:** If we stay, we move to **Azure Functions** for serverless scale and **Azure Database for PostgreSQL (Flexible Server)** for automated failover.
* **Hardened Signing:** **Azure Key Vault (Managed HSM)** provides FIPS 140-2 Level 3 security for on-chain settlements.

**Discussion Goal:** Decide which ecosystem offers the best credits, managed reliability, and long-term operational ease for the team.
