# GAIa: Distributed Edge AI Sub-Cluster
### High-Performance Ollama Inference on Overclocked Raspberry Pi 5
**Part of the [Project Jupiter] Ecosystem**

## 🌌 Overview
GAIa (General AI assembly) is a specialized two-node sub-cluster engineered for private, localized Large Language Model (LLM) inference. This project represents a deep-dive into maximizing ARM-based silicon, moving away from the overhead of container orchestration toward a high-performance **Bare Metal** architecture.

GAIa resides in Sleds 1 and 2 of the **Project Jupiter** 4-bay tower, serving as the "Intelligence Layer" of the rack while its sibling nodes (Sleds 3-4) handle distributed scientific computing via BOINC.

## 🛠️ Hardware Specification (Nodes: Io & Europa)
The infrastructure is designed for 24/7 high-load stability with a focus on thermal headroom and power delivery.

| Component | Specification |
| :--- | :--- |
| **Compute Nodes** | 2x Raspberry Pi 5 (8GB RAM) |
| **Cooling** | 2x Official Pi 5 Active Coolers (Custom Fan Curves) |
| **Power** | 2x Waveshare PoE HAT (G) via Netgear PoE+ Managed Switch |
| **Network** | Physical Ports 1 & 2; Static Internal IP Assignment |
| **Storage** | 2x SanDisk 128GB Max Endurance MicroSD (High-Cycle SLC) |
| **Chassis** | UCTronics 4-Bay Tower (Occupying Sleds 1 & 2) |
| **Environment** | Dedicated Basement Rack (Sub-20°C Ambient Floor) |

## 🏗️ The "Bare Metal" Engineering Pivot
Originally conceived as a Docker/Kubernetes orchestrated cluster, GAIa underwent a significant architectural shift following stability testing with 8B-parameter models.

### 1. The Container Exit
To eliminate networking jitter and memory fragmentation, **Ollama was moved to a Bare Metal installation**. This allows the Linux kernel to manage the Pi 5's 8GB LPDDR4X-4267 SDRAM directly, preventing Out-of-Memory (OOM) errors during complex reasoning tasks.

### 2. Decoupled Storage (The Vault)
* **Node: Io (The Vault):** Functions as the cluster's high-speed "Librarian." It hosts a tuned **NFSv4 share** containing the model weights, allowing for stateless compute on secondary nodes.
* **Node: Europa (The Specialist):** A dedicated compute engine that mounts "The Vault" to run specialized coding models without local storage overhead.

### 3. Hybrid Frontend
While the AI engine runs on bare metal, **Open WebUI** is maintained within a lean Docker container. This "Hybrid" approach provides a modern, browser-based interface while ensuring the heavy lifting of inference remains unencumbered by container networking layers.

## ⚡ Performance Tuning & Thermal Engineering
GAIa is pushed beyond stock specifications to minimize token-generation latency.

* **Aggressive Overclock:** Both nodes are tuned to **2.6 GHz** with manual overvolting, providing a ~20% uplift in raw compute.
* **Inference Optimization:** * **Threads:** Pinned to **4** to align with the physical Cortex-A76 core count.
    * **Memory Management:** `num_ctx` is capped at **2048** to ensure the KV cache stays entirely within RAM, preventing performance-killing SD card swap thrashing.
* **Model Roster:**
    * **Io:** `llama3.2:3b` (Fast Utility) & `deepseek-r1:8b` (Complex Reasoning).
    * **Europa:** `qwen2.5-coder:7b` (Technical/Scripting).

## 📈 Monitoring (SITREP)
Real-time telemetry is handled via **Glances** in server mode, providing a unified dashboard for:
* **SoC Thermals:** Monitoring the delta between basement ambient and the 2.6GHz peak load.
* **I/O Wait:** Tracking the health of the NFS model offloading.
* **SITREP Function:** A custom `.bashrc` function providing instant hardware telemetry upon SSH login.

* ## 🛡️ Security & Hardening
With the cluster's move to a permanent basement location, the network stack was hardened for production-level stability:
* **Software Firewall:** Granular port-blocking and IP-whitelisting for inter-node communication.
* **Zero Trust NFS:** Restricted export rules ensuring model weights are only accessible to verified cluster nodes.

---

## 👨‍💻 Maintainer
**ScienzGuy** [@ScienzGuy](https://github.com/ScienzGuy)
