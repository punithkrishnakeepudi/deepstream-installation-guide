# NVIDIA DeepStream 7.1 — Native Installation Guide for Ubuntu 24.04 LTS

<div align="center">

![NVIDIA DeepStream](https://img.shields.io/badge/NVIDIA-DeepStream%207.1-76b900?style=for-the-badge&logo=nvidia&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-24.04%20LTS-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![CUDA](https://img.shields.io/badge/CUDA-12.6-76b900?style=for-the-badge&logo=nvidia&logoColor=white)
![TensorRT](https://img.shields.io/badge/TensorRT-10.3+-76b900?style=for-the-badge&logo=nvidia&logoColor=white)

**A battle-tested, step-by-step guide for deploying NVIDIA DeepStream 7.1 natively on Ubuntu 24.04 (Noble Numbat) — including all known dependency fixes and workarounds.**

</div>

---

## 📌 Overview

[NVIDIA DeepStream SDK](https://developer.nvidia.com/deepstream-sdk) is a high-performance streaming analytics toolkit built on GStreamer, designed for AI-based multi-sensor processing and video analytics at the edge and in the cloud.

**DeepStream 7.1 officially targets Ubuntu 22.04.** This guide documents the exact steps and workarounds required to run it successfully on **Ubuntu 24.04 LTS**, addressing dependency shifts such as `libyaml-cpp` versioning and Python 3.12 compatibility.

> ✅ **Verified on:** NVIDIA GeForce GTX 1650 Mobile · Ubuntu 24.04 LTS · Driver 550 · CUDA 12.6

---

## 📋 Table of Contents

- [System Requirements](#-system-requirements)
- [Installation Steps](#️-installation-steps)
- [Troubleshooting](#️-troubleshooting)
- [Verification](#-verification)
- [Running a Sample](#-running-a-sample)
- [Contributing](#-contributing)
- [License](#-license)

---

## 💻 System Requirements

### Hardware

| Component | Minimum | Tested Configuration |
|-----------|---------|---------------------|
| **GPU** | NVIDIA dGPU (Turing or newer) | GeForce GTX 1650 Mobile |
| **RAM** | 8 GB | 16 GB recommended |
| **Storage** | 10 GB free | SSD recommended |

### Software

| Dependency | Version |
|------------|---------|
| **Operating System** | Ubuntu 24.04 LTS (Noble Numbat) |
| **NVIDIA Driver** | 550.54.14 or newer |
| **CUDA Toolkit** | 12.6 |
| **TensorRT** | 10.3 or newer |

---

## 🛠️ Installation Steps

### Step 1 — System Cleanup

Remove any previous or partial DeepStream installations to ensure a clean environment:

```bash
sudo rm -rf /opt/nvidia/deepstream*
sudo rm -rf /opt/nvidia/deepstream_data
```

---

### Step 2 — NVIDIA Driver & CUDA Setup

DeepStream 7.1 requires NVIDIA Driver 550 and CUDA 12.6.

```bash
# Update package lists
sudo apt update

# Install NVIDIA Driver 550
sudo apt install -y nvidia-driver-550

# Add the official CUDA 12.x repository for Ubuntu 24.04
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2404/x86_64/cuda-keyring_1.1-1_all.deb
sudo dpkg -i cuda-keyring_1.1-1_all.deb
sudo apt update

# Install CUDA Toolkit 12.6
sudo apt install -y cuda-toolkit-12-6
```

> 💡 **Tip:** Reboot your system after driver installation to ensure the GPU is fully initialized.

---

### Step 3 — Install Core Dependencies

DeepStream depends on GStreamer and several multimedia/development libraries:

```bash
sudo apt install -y \
    libssl-dev \
    libgstreamer1.0-0 \
    gstreamer1.0-tools \
    gstreamer1.0-plugins-good \
    gstreamer1.0-plugins-bad \
    gstreamer1.0-plugins-ugly \
    gstreamer1.0-libav \
    libgstrtspserver-1.0-0 \
    libjansson4 \
    libyaml-cpp-dev \
    gcc make git
```

---

### Step 4 — Install the DeepStream SDK

1. Download the Debian package from the **[NVIDIA NGC Portal](https://catalog.ngc.nvidia.com/orgs/nvidia/resources/deepstream)**.
2. Install it using `apt-get` to automatically resolve remaining dependencies:

```bash
sudo apt-get install -y ./deepstream-7.1_7.1.0-1_amd64.deb
```

---

## ⚠️ Troubleshooting

Installing DeepStream 7.1 on Ubuntu 24.04 introduces specific library mismatches not present on the officially supported 22.04. The following issues and fixes were identified during testing.

---

### Issue 1 — Missing `libyaml-cpp.so.0.7`

**Error:**
```
deepstream-app: error while loading shared libraries: libyaml-cpp.so.0.7: cannot open shared object file
```

**Root Cause:** Ubuntu 24.04 ships with `libyaml-cpp.so.0.8`, while DeepStream links against `0.7`.

**Fix:** Create a symbolic link to redirect the dependency:

```bash
sudo ln -s /usr/lib/x86_64-linux-gnu/libyaml-cpp.so.0.8 \
           /usr/lib/x86_64-linux-gnu/libyaml-cpp.so.0.7
```

---

### Issue 2 — GStreamer Plugin Not Found (`nvinfer`)

**Error:**
```
No such element or plugin 'nvinfer'
```

**Root Cause:** GStreamer's plugin registry is stale and does not reflect the newly installed NVIDIA plugins.

**Fix:** Clear the cache to force a full rescan:

```bash
rm -rf ~/.cache/gstreamer-1.0
```

---

### Issue 3 — Warnings About Missing Triton / Rivermax Libraries

**Warnings:**
```
Could not load libtritonserver.so
Could not load librivermax.so
```

**Explanation:** These are optional NVIDIA components:
- **Triton** — NVIDIA's Triton Inference Server (for multi-framework model serving)
- **Rivermax** — High-speed, low-latency networking SDK

If your use case does not require these, **the warnings can be safely ignored.** To fully resolve them, install the respective SDKs from the [NVIDIA Developer Portal](https://developer.nvidia.com).

---

## ✅ Verification

Confirm a successful installation by running the following checks:

```bash
# 1. Verify CUDA installation
nvcc --version

# 2. Verify DeepStream version
deepstream-app --version

# 3. Inspect core GStreamer plugins
gst-inspect-1.0 nvinfer
gst-inspect-1.0 nvtracker
```

All three commands should return without errors.

---

## 🏃 Running a Sample Pipeline

Run one of the built-in DeepStream sample applications to validate the full pipeline end-to-end:

```bash
cd /opt/nvidia/deepstream/deepstream-7.1/samples/configs/deepstream-app

deepstream-app -c source4_1080p_dec_infer-resnet_tracker_sgie_tiled_display_int8.txt
```

This sample runs a 4-stream, 1080p inference pipeline using a ResNet model with multi-object tracking.

---

## 🤝 Contributing

Contributions, corrections, and additional platform-specific guides are welcome!

1. Fork this repository
2. Create a feature branch (`git checkout -b fix/your-fix-name`)
3. Commit your changes (`git commit -m 'fix: describe your fix'`)
4. Push to the branch (`git push origin fix/your-fix-name`)
5. Open a Pull Request

---


<div align="center">

*Documented after a successful native deployment on Ubuntu 24.04 LTS.*
*If this guide helped you, consider giving it a ⭐ on GitHub.*

</div>
