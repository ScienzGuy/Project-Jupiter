# Project Jupiter: Distributed Edge AI Cluster
### Overclocked Raspberry Pi 5 Ollama Cluster with NFS Model Offloading

## Overview
Project Jupiter is a high-performance Raspberry Pi 5 cluster designed for private, localized Large Language Model (LLM) inference. This project demonstrates a decoupled architecture where AI model storage (The Vault) is separated from AI compute (The Processor) to maximize SD card longevity and system scalability.

## Architecture
- **Head Node (Io):** Acting as the "Vault," this node manages a 128GB high-speed partition and serves model weights via a tuned NFSv4 share.
- **Compute Node (Europa):** An overclocked Raspberry Pi 5 (2.6 GHz CPU / 900 MHz GPU) that acts as the primary inference engine.
- **Interface:** Open WebUI orchestrated via Docker, connecting to the distributed Ollama API.

## Key Features
* **Decoupled Storage:** Models are stored on a centralized NFS mount to prevent SD card wear and allow instant model access for new cluster nodes.
* **Aggressive Overclocking:** Stable 2.6 GHz performance profile with active cooling, maintaining sub-65°C (149°F) temps under full load.
* **Custom Monitoring:** A bespoke Bash-based SITREP dashboard providing real-time telemetry on thermals, PMIC status, clock speeds, and active model residency.
* **Stateless Compute:** Nodes can be added or replaced without re-downloading multi-gigabyte model weights.

## Performance
- **Model:** Llama 3.2 (3B)
- **Clock Speed:** 2600 MHz
- **Thermal Delta:** +35°C from idle to peak load.
- **Power Delivery:** PoE+ (802.3at) with active fan curves.

## Installation & Usage

### 1. The Monitoring Dashboard
To add the Project Jupiter SITREP dashboard to your environment, append the following functions to your `~/.bashrc`:

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
