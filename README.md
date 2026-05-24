# IoT Data Simulation & Blockchain Ledger

A simulation of environmental IoT sensors deployed across the Philippines whose readings are stored on a local Ethereum (Ganache) blockchain through a Solidity smart contract using Web3.py.

This project was built for **MO-IT148 — Application Development and Emerging Technologies (Group 8)**.

## Overview

The project has two halves:

1. **IoT data generation** ([iot_simulation_data.ipynb](iot_simulation_data.ipynb)) — simulates 30 days of daily readings from 20 sensors spread across 12 provinces in Luzon, Visayas, and Mindanao, producing a 600-row dataset of air, climate, soil, and water measurements.
2. **Blockchain ledger** ([main.ipynb](main.ipynb)) — connects to a local Ganache node, loads a deployed smart contract, and writes/reads IoT records as on-chain transactions.

## Repository Contents

| File | Description |
| --- | --- |
| `iot_simulation_data.ipynb` | Generates the simulated IoT dataset and exports `iot_data.csv`. |
| `main.ipynb` | Connects to Ganache, loads the smart contract, stores a record, and reads it back. |
| `iot_data.csv` | 600 simulated daily sensor records (output of the simulation notebook). |
| `requirements.txt` | Python dependencies (Web3, pandas, numpy, python-aqi, etc.). |
| `.gitignore` | Ignores virtual envs, IDE folders, `.env`, and caches. |

## Simulated Dataset

Each record represents one sensor's reading on one day and includes:

- **Identification & location** — `Sensor ID`, `Location` (province), `Latitude`, `Longitude`
- **Air quality** — `CO2` (ppm), `PM2.5`, `PM10` (µg/m³), `O3`, `NO2` (ppb), computed `AQI` and `AQI Level`
- **Climate** — `Temperature` (°C), `Humidity` (%)
- **Soil** — `Soil Moisture` (%) and category (`Dry` / `Optimal` / `Wet`)
- **Water** — `pH`, `Turbidity` (NTU)

### Design choices

- **Geographic spread:** 4 provinces per region; ±0.1° jitter on coordinates so sensors don't all sit on the centroid. Each sensor keeps a fixed location across all days.
- **Sensor distribution:** 7 in Luzon, 7 in Visayas, 6 in Mindanao (20 total).
- **Time window:** 30 daily readings starting **March 1, 2026** → **March 30, 2026** (600 records total).
- **Realistic ranges:** values bounded by typical/WHO/EPA ranges (see notebook for constraints).
- **AQI:** computed via [`python-aqi`](https://pypi.org/project/python-aqi/) (EPA standard) from PM2.5, PM10, O₃ (8h), NO₂ (1h); falls back to a pollutant mean if computation fails.

## Smart Contract

The blockchain notebook expects a Solidity contract (deployed via Remix to Ganache) exposing:

```solidity
function storeData(string _deviceId, string _dataType, string _dataValue) external;
function getRecord(uint256 index) external view returns (uint256, string, string, string);
function getTotalRecords() external view returns (uint256);
event DataStored(uint256 timestamp, string deviceId, string dataType, string dataValue);
```

Each on-chain record stores `timestamp`, `deviceId`, `dataType`, and `dataValue`.

## Getting Started

### Prerequisites

- Python 3.10+
- [Ganache](https://archive.trufflesuite.com/ganache/) running locally at `http://127.0.0.1:8545`
- A smart contract deployed to that Ganache instance (e.g. via [Remix](https://remix.ethereum.org/)) — copy its address and ABI into `main.ipynb`
- Jupyter Notebook / VS Code

### Installation

```bash
git clone https://github.com/kurtsanor/iot-data-simulation.git
cd iot-data-simulation
python -m venv .venv
source .venv/bin/activate    # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

### Run the simulation

Open and run `iot_simulation_data.ipynb`. It will print a preview and write `iot_data.csv` to the project root.

### Push data to the blockchain

1. Start Ganache and deploy the smart contract.
2. In `main.ipynb`, replace `contract_address` with the deployed contract's address and update the `abi` if needed.
3. Run all cells. The notebook will:
   - Confirm the Ganache connection
   - Load the contract
   - Store a sample record (`TEST001`, `Temperature`, `22.5°C`)
   - Print `getTotalRecords()` and the first stored record

Expected console output:

```
✅ Connected to Ganache successfully!
✅ Connected to Smart Contract at 0x69896666923D6FCAaF32a495D07d362049584F97
✅ Dummy data stored on blockchain!
Total Records: 1
First Stored Record: [<timestamp>, 'TEST001', 'Temperature', '22.5°C']
```

## Course

MO-IT148 — Application Development and Emerging Technologies · **Group 8**
