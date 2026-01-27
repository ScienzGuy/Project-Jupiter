# GAIa: Distributed Edge AI Pi Cluster
## High-Performance Ollama Implementation on Overclocked Pi 5

### 🌌 Overview
GAIa (General AI assembly) is a specialized two-node sub-cluster engineered for private, localized Large Language Model (LLM) inference. The cluster is comprised of two Raspberry Pi 5 nodes—Io and Europa—operating in a unified bare-metal environment.

The project represents a deep-dive into maximizing ARM-based silicon, moving away from the overhead of container orchestration toward a high-performance **Bare Metal** architecture for the inference engine. This design ensures the cluster can handle 7B and 8B parameter models with minimal token-generation latency.

### 🛠️ Hardware Specification (Nodes: Io & Europa)
GAIa is tuned for bursty, high-load AI workloads with a focus on maximum thermal headroom.

| Component | Specification |
| :--- | :--- |
| **Compute Nodes** | 2x Raspberry Pi 5 (8GB RAM) |
| **Cooling** | 2x Official Pi 5 Active Coolers (Custom Fan Curves) |
| **Power** | 2x Waveshare PoE HAT (G) via PoE+ Managed Switch |
| **Network** | Static Internal IP Assignment; Dedicated Physical Ports |
| **Storage** | 2x SanDisk 128GB Max Endurance MicroSD (High-Cycle SLC) |
| **Environment** | Dedicated Basement Rack (Sub-20°C Ambient Floor) |

### 🏗️ The "Bare Metal" Engineering Pivot
GAIa underwent a significant architectural shift following stability testing. We abandoned Kubernetes and Docker for the core engine due to performance concerns and restrictive container security policies.

**1. The Hybrid Architecture**
While the **Ollama** inference engine runs on bare metal to allow the kernel direct access to the 8GB LPDDR4X RAM, the **Open WebUI** frontend is maintained within a lean Docker container. This ensures the heavy lifting of inference remains unencumbered by container networking layers while providing a modern interface.

**2. Decoupled Storage (The Vault)**
* **Node: Io (The Vault):** Functions as the "Librarian." It hosts a tuned **NFSv4 share** containing the model weights, allowing for stateless compute on secondary nodes.
* **Node: Europa (The Specialist):** A dedicated compute engine that mounts "The Vault" to run specialized coding models without local storage overhead.

### ⚡ Performance Tuning & Thermal Engineering
GAIa is pushed beyond stock specifications to minimize the "time-to-first-token" and overall inference latency.

* **Aggressive Overclock:** Both nodes are tuned to **2.6 GHz** with manual overvolting. This provides a ~20% uplift in raw compute necessary for fluid LLM interactions.
* **Thermal Management:** Despite the overclock, the sub-20°C ambient basement floor and custom fan curves keep the SoC **under 75°C (167°F)** during peak reasoning tasks.
* **Inference Optimization:**
    * **Threads:** Pinned to **4** to align with the physical Cortex-A76 core count.
    * **Memory:** `num_ctx` is capped at **2048** to ensure the KV cache stays entirely within RAM, preventing SD card swap thrashing.

### 📈 Monitoring (SITREP)
* **Real-Time Telemetry:** Handled via **Glances** in server mode, monitoring the delta between basement ambient and the 2.6GHz peak load.
* **SITREP Function:** A custom `.bashrc` function providing instant hardware telemetry and NFS mount health upon SSH login.
* **Model Roster:**
    * **Io:** `llama3.2:3b` (Utility) & `deepseek-r1:8b` (Complex Reasoning).
    * **Europa:** `qwen2.5-coder:7b` (Technical/Scripting).

### 🛡️ Security & Hardening
* **Zero Trust NFS:** Restricted export rules ensuring model weights are only accessible to verified cluster nodes.
* **Software Firewall:** Granular port-blocking and IP-whitelisting for inter-node communication.

---

Maintained by Scienz_Guy | 2026
