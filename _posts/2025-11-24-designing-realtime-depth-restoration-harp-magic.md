---
title: "Designing Real-Time Depth Restoration Modules for Multi-LiDAR Systems: HARP and MaGIC"
categories:
  - Research Notes
tags:
  - []
toc: true
toc_sticky: true
 
date: 2025-11-24
last_modified_at: 2026-07-13
---

This post shares the motivation, design philosophy, and architectural decisions behind **DRIM**, published in *IEEE Robotics and Automation Letters (RA-L) 2025*.

Rather than summarizing the paper, I explain why I designed the two key modules—**HARP** and **MaGIC**—how they address the challenges of multi-LiDAR interference, and the reasoning behind the final architecture.

---

## 1. The Core Problem: Why Existing Restoration Fails
In practical robotics and smart manufacturing, deploying multiple RGB-D (LiDAR-based) cameras is essential to expand the Field of View (FoV) and eliminate blind spots.
However, when these devices operate simultaneously, they inevitably experience cross interference.

For example, scanning-based LiDAR cameras (like the Intel RealSense L515) suffer from depth artifacts when lasers from neighboring cameras cross paths, resulting in severely corrupted depth maps.
<img width="1157" height="484" alt="image" src="https://github.com/user-attachments/assets/53ade9da-b972-48ef-8907-6862de9b1519" />

Through deep analysis of this phenomenon, the anomalies were categorized into two distinct types:
- Sensor Artifacts: Inherent invalid depth zones caused by the single camera's scanning nature. **(Must be PRESERVED)**
- Interference Artifacts: Corrupted depth values caused by cross-device laser overlapping. **(Must be RESTORED)**

Standard image restoration models treat both as generic noise and try to "erase" everything.
As a result, they ruin the structural integrity of natural sensor gaps while failing to clean up the actual interference.
The core limitation was the lack of a mechanism that allows the model to distinguish what to touch and what to preserve.

To solve this, the research demanded an architecture that explicitly decouples these two artifacts and handles them with targeted restoration strategies.

---

## 2. My Core Architectural Contributions & Design Intent
Below is the overall pipeline of the network architecture I designed.
<img width="1280" height="580" alt="image" src="https://github.com/user-attachments/assets/eec29fb9-58cf-4ed8-82c0-68667104e643" />

Here is a breakdown of how I designed and engineered the model to achieve both real-time execution (~33 FPS) and high-fidelity depth restoration.

### 1) HARP (High-resolution Attention Refinement Path)
> Goal: Maintain fine-grained pixel-level structures while achieving strict real-time performance.

Deploying deep learning in industrial robotics demands real-time processing speed.
However, preserving high-resolution features and running at high speeds are fundamentally at odds. 

Multi-device interference manifests as extremely thin, linear patterns (1–2 pixel wide fine artifacts).
When a standard encoder downsamples the input (e.g., H/4 -> H/8 -> H/16 -> H/32), these tiny, critical patterns are completely washed out before reaching the decoder.

To overcome this trade-off, I proposed and designed HARP:
- **Shallow Non-Downsampling Path:** I added a parallel auxiliary path that processes the original image scale without any downsampling.
- **Intended "Shallow" Design:** To keep the computational overhead to an absolute minimum, I restricted it to a very lightweight residual block structure. This kept the model running at a smooth 33 FPS.
- **CBAM (Convolutional Block Attention Module):** Full self-attention on raw resolution is computationally prohibitive for real-time systems. Instead, I integrated CBAM to selectively capture essential spatial and channel-wise edge features of linear interference without bloating the inference time.
- **Dual-Decoder Routing:** I routed the high-resolution features from HARP to both the Mask Decoder and the Depth Decoder. This drastically boosted the precision of both the boundary segmentation and the ultimate structural restoration.

### 2) MaGIC (Mask-Guided Interference Correction)
> Goal: Explicitly guide the restoration process using a multi-class semantic prior.

MaGIC represents the core philosophy of our approach: Restoration must be guided by semantic understanding.
I built this module to bridge semantic routing and actual pixel reconstruction.

- **3-Class Probability Formulation:** I formulated the task to categorize pixels into Background, Sensor Artifacts, or Interference to force the network to explicitly understand the physical nature of the anomalies.
- **Multi-Scale Mask Injection:** Instead of treating the predicted mask as a simple final output, I utilized the 3-channel probability map as an explicit guide. I injected these maps directly into the intermediate stages of the Depth Decoder via Multi-Scale Channel-wise Concatenation.
- **Why Concatenation Over Attention?** During the prototyping phase, I experimented with various gating and attention modules to inject the mask. However, since the artifacts have distinct spatial topologies (thin lines and specific shapes), attention maps tended to diffuse the boundaries. I discovered that direct concatenation directly passes the raw shape and probability distributions into the restoration stream without any information loss, forcing the decoder to selectively reconstruct only the interference zones.

---

## 3. Results & Real-World Impact

By anchoring the network with these custom modules, the model achieved exceptional real-world performance that raw baseline models could not match:
- **Quantitative Superiority:** Secured a tight overall RMSE of 0.0199m and a localized Interference RMSE of 0.0774m.
- **Real-Time Efficiency:** Clocked in at ~33 FPS (0.0299s per frame) on standard hardware, outperforming heavy baseline restoration networks in both speed and quality.
- **Robust Generalization:** Demonstrated high resilience under challenging, cross-device scenarios involving multiple mixed-model sensors.

### Engineering Takeaway
Hardware-based synchronization (cabling, active triggering) has been the industry standard for multi-LiDAR setups, but it suffers from physical constraints, frame drops, and painful deployment processes.

While establishing the first multi-LiDAR interference dataset was an essential milestone for the community, the true value was unlocked by deploying a network specifically tailored to the underlying physics of the sensor interference.

By designing HARP to preserve micro-structures and MaGIC to semantically isolate target anomalies, I demonstrated that complex, device-level physical interference can be effectively mitigated in real-time through intelligent, hardware-aware software architecture.
