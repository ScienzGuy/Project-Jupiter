# Pi Project: GAIa - Distributed Edge AI Cluster
### Overclocked Raspberry Pi 5 Ollama Cluster with NFS Model Offloading



## 1. Executive Summary
Project GAIa is a high-availability, distributed AI inference cluster built on the Raspberry Pi 5 platform. This project implements a decoupled storage-compute architecture, separating AI model storage ("The Vault") from AI inference ("The Processor"). By serving model weights over a tuned Network File System (NFS), the cluster achieves stateless compute capabilities, maximizes SD card longevity, and allows for near-instant scaling of inference nodes.

## 2. System Architecture
The cluster utilizes a master-worker topology optimized for high-throughput model loading and low-latency local inference:

* **The Vault (Node: Io):** Acts as the cluster orchestrator and primary storage gateway. It hosts a 128 GB (137.4 GB) high-speed partition and serves model weights via a tuned **NFSv4** share.
* **The Processor (Node: Europa):** A dedicated compute node optimized for performance. It features a stable **2.6 GHz** (2600 MHz) CPU overclock and a **900 MHz** GPU clock to accelerate tensor operations.
* **The Interface:** A containerized deployment of **Open WebUI**, orchestrated via Docker on Io, providing a unified management plane for the distributed Ollama API.

---

## 3. Hardware Configuration

| Component | Specification (Io - The Vault) | Specification (Europa - The Processor) |
| :--- | :--- | :--- |
| **Model** | Raspberry Pi 5 (8 GB) | Raspberry Pi 5 (8 GB) |
| **CPU Clock** | 2.4 GHz (Stock) | **2.6 GHz (Overclocked)** |
| **GPU Clock** | 800 MHz (Stock) | **900 MHz (Overclocked)** |
| **Memory** | 8 GB LPDDR4X-4267 | 8 GB LPDDR4X-4267 |
| **Storage** | 128 GB NFS Model Vault | Stateless / NFS Mounted |
| **Cooling** | Active Air Cooling | Active Air Cooling + Heat Sinks |
| **Power** | PoE+ (802.3at) | PoE+ (802.3at) |

---

## 4. Key Engineering Features
* **Decoupled Model Residency:** By mounting `/var/lib/ollama` over NFS, multi-gigabyte model weights are stored once but accessible by any node in the cluster. This architecture prevents redundant downloads and protects flash storage from heavy write cycles.
* **Thermal Management:** Europa is tuned to a 2.6 GHz performance profile. Managed active cooling ensures a thermal ceiling of **65°C** (149°F), preventing thermal throttling during sustained LLM inference.
* **SITREP Telemetry:** A custom Bash-based monitoring suite provides real-time visibility into the cluster's internal state, including PMIC status, clock speeds, and active model residency.

## 5. Deployment & Telemetry

### The SITREP Dashboard
To implement the Project Jupiter SITREP dashboard, append the following function to your `~/.bashrc`. This provides an instantaneous health check for any node in the cluster.

```bash
# Project Jupiter SITREP Function
labstatus() {
    echo -e "\033[1;34m--- NODE HEALTH: $(hostname) ---\033[0m"
    echo -ne "CPU Load: " && awk '{print $1, $2, $3}' /proc/loadavg
    echo -ne "Temp:      " && vcgencmd measure_temp
    echo -ne "CPU Clock: " && echo "$(($(vcgencmd measure_clock arm | cut -d'=' -f2) / 1000000)) MHz"
    
    echo -e "\n\033[1;32m[ACTIVE MODELS]\033[0m"
    if pgrep -f "ollama runner" > /dev/null; then
        echo -e "\033[0;32mModel: Loaded & Active\033[0m"
    else
        echo "AI Engine: Idle"
    fi
}
```

## 6. Current Benchmarks
* **Primary Inference Models:** DeepSeek-R1 8B (Distilled), Llama 3.1 8B.
* **Operational Temperature:** ~42°C (107.6°F) Idle | ~64°C (147.2°F) Peak Load.
* **Throughput:** Optimized for ~3–4 tokens per second on 8B parameter models.

## 7. Technical Challenges & Resolutions

### Challenge: Inter-Node Administrative Visibility
**Issue:** Standard terminal-based management made it difficult to monitor both Io and Europa simultaneously without multiple SSH sessions. Installing a full Desktop GUI was rejected to preserve system resources for LLM inference.
**Resolution:** Deployed **Cockpit** on Io and linked Europa as a managed node. This provides a lightweight, web-based SITREP for both nodes, allowing for real-time monitoring of CPU load and temperature across the cluster from a single dashboard.

### Challenge: NFS Permission & Service Persistence
**Issue:** During the initial setup of "The Vault," the Ollama service on Europa could not write to the NFS mount due to UID/GID mismatches between the two nodes, and the `cockpit.socket` failed to initialize on boot.
**Resolution:** 1. Synchronized the `piadmin` user IDs across both nodes and updated the `/etc/exports` configuration on Io to use `no_root_squash`.
2. Reinstalled the `cockpit-pinger` and `cockpit-bridge` packages on Europa to ensure the service socket correctly listens for requests from the Io head node.

### Challenge: Thermal Throttling at 2.6 GHz
**Issue:** Under sustained load (Llama 3.1 8B inference), Europa's SoC temperature would climb toward the 80°C (176°F) throttle point.
**Resolution:** Implemented a custom firmware-level fan curve via `config.txt` and optimized the physical placement of the cluster to ensure high-volume airflow, successfully capping peak loads at **65°C** (149°F).
