# 🚗 HPN — Hierarchical Policy Network

> **HPN** — A Hierarchical Policy Network Architecture to Resolve Autonomous Driving Corner Cases, Combining Modular Interpretability & End-to-End Performance

**HPN (Hierarchical Policy Network)** is a novel hybrid autonomous driving architecture powered by digital twin simulation, designed to overcome the inherent tradeoffs between traditional modular pipelines and monolithic end-to-end driving models. It addresses long-tail rare corner cases in self-driving by splitting driving tasks into tiered expert primitive policies managed by a high-level Arbiter, balancing full safety verifiability, model interpretability, and high driving performance. This framework is validated on CARLA and mainstream autonomous driving digital twin simulators, supporting all common urban, highway, and emergency driving scenarios.

<p align="center">
  <img width="900" alt="HPN" src="https://github.com/user-attachments/assets/8551ba22-8ada-405e-a97a-9d3726c94b7b" />
</p>
<p align="center">
  <em>Figure 1: HPN framework overview</em>
</p>

## 📚 Table of Contents

- [Project Background & Core Motivation](#project-background--core-motivation)
- [Core HPN Architecture Overview](#core-hpn-architecture-overview)
- [Four-Tier Driving Primitive Hierarchy](#four-tier-driving-primitive-hierarchy)
- [Project Key Features](#project-key-features)
- [HPN Theoretical Advantages](#hpn-theoretical-advantages)

## Project Background & Core Motivation

Two mainstream autonomous driving architectures have irreconcilable drawbacks:

### 1. Modular Driving Pipeline

**Pros:**
- ✅ Fully interpretable
- ✅ Explicit hard safety constraints
- ✅ Parallel team development
- ✅ Easy fault localization

**Cons:**
- ❌ Fragile error cascading
- ❌ Suboptimal overall driving performance
- ❌ Poor adaptability to complex unstructured scenes

### 2. Monolithic End-to-End Network

**Pros:**
- ✅ Holistic learning
- ✅ Superhuman driving potential
- ✅ Minimal manual rule engineering

**Cons:**
- ❌ Black-box logic
- ❌ Severe long-tail scenario degradation
- ❌ Impossible formal safety verification
- ❌ Unknown root causes of accidents

### 💡 HPN Breaks This Dilemma

HPN is **neither purely modular nor fully monolithic**. It adopts a **team-of-experts hierarchical paradigm** to retain the interpretability and safety controllability of modular systems while inheriting the strong environmental adaptability of end-to-end models, specifically optimized for rare, safety-critical driving corner cases.

---

## Core HPN Architecture Overview

The complete HPN system consists of 4 core functional modules, built upon a shared BEV perception backbone:

### 🧠 1. Shared Perception Backbone (SPB)
Fuses raw camera, LiDAR, radar, GPS and vehicle dynamics signals into unified BEV bird's-eye view world representation, serving as the single environmental truth source for all downstream modules.

### 🎯 2. Arbiter (High-Level Decision Brain)
A high-level classification network that takes BEV features + navigation goals, selects the matching expert driving primitive policy according to current driving state, and outputs candidate policy proposals.

### 🛡️ 3. Policy Gateway & Verifier (PGV)
Deterministic rule-based safety guardrail module. Hard-coded traffic laws and safety constraints validate Arbiter-proposed policies; invalid dangerous policies are rejected and blocked from vehicle actuation.

### 🚀 4. Tiered Expert Policy Team
Multiple lightweight end-to-end expert models, each exclusively trained on a dedicated driving primitive. Policies are divided into 4 priority tiers with strict preemption logic to prioritize rare safety-critical tail scenarios.

<p align="center">
  <img width="550" alt="hpn_data_flow" src="https://github.com/user-attachments/assets/aa800459-61b7-4c92-94e9-ba7815a38849">
</p>
<p align="center">
  <em>Figure 2: HPN overall data flow diagram</em>
</p>

---

## Four-Tier Driving Primitive Hierarchy

> **Priority order: Tier 0 > Tier 1 > Tier 2 > Tier 3**
> "Highest priority to the least frequent but most critical events."

| Tier | Name | Scenario Type | Core Policies | Preemption Rule |
|:---:|:---|:---|:---|:---|
| **Tier 0** | System State Policies | System status management | Off, Standstill, Fault | Overrides all other tiers |
| **Tier 1** | Safety Override Policies | Emergency collision avoidance | Emergency Brake, Evasive Steer | Overrides Tier 2 & Tier 3 |
| **Tier 2** | Transitional Maneuver Policies | Short-term dynamic maneuvers | Lane Change, Merge, Intersection Traverse, Parking | Overrides Tier 3 only |
| **Tier 3** | Steady-State Default Policies | Routine cruising | Lane Following, Car Following, Intersection Approach, Creep | Lowest priority, preempted by all upper tiers |

### 📋 Complete Primitive List

#### 👑 Tier 0 — System State Policies

- **P0.1 Off** — System shutdown, vehicle manually controlled
- **P0.2 Standstill** — System activated but vehicle stationary
- **P0.3 Fault** — Detected system malfunction

> ⚡ **Preemption**: Overrides **all** other tiers

#### 🛡️ Tier 1 — Safety Override Policies

- **P1.1 Emergency Brake** — Maximum braking based on **TTC (Time-to-Collision)** to avoid forward collisions
- **P1.2 Evasive Steer** — Emergency lateral avoidance when braking alone cannot avoid collision

> ⚡ **Preemption**: Overrides **Tier 2 & Tier 3**

#### 🔄 Tier 2 — Transitional Maneuver Policies

- **P2.1 Lane Change** — Change from current lane to adjacent lane (including overtaking)
- **P2.2 Merge** — Merge from ramp or termination lane into main road traffic flow
- **P2.3 Intersection Traverse** — Navigate through signalized intersections (straight/turn/U-turn)
- **P2.4 Parking** — Low-speed, high-precision parking in designated spaces

> ⚡ **Preemption**: Overrides **Tier 3** only

#### 🛣️ Tier 3 — Steady-State Default Policies

- **P3.1 Intersection Approach** — Approach intersections with yield and parking control
- **P3.2 Car Following** — Maintain safe distance from leading vehicle (longitudinal + lateral)
- **P3.3 Lane Following** — Default state, keep driving in lane center
- **P3.4 Creep** — Extremely low-speed creeping for visibility in poor light/weather or congestion

> ⚡ **Preemption**: Lowest priority, preempted by **all** upper tiers

> 📌 **Digital Twin Note**: Each Tier 3 primitive policy is trained with **two separate digital twin models**, enabling parallel simulation-based validation and real-world deployment alignment.

### ✨ Key Design Advantages

- Rare but safety-critical tail scenarios (Tier 0/1/2) receive higher execution priority, solving the problem of long-tail underfitting in single end-to-end models
- Each expert only learns their own exclusive scenario distribution, without statistical noise interference
- HPN can be represented as a finite state machine (FSM), where the driving elements represent the system state, and transitions between states are controlled by the Arbiter
- **Strict Preemption**: Higher-tier policies (Tier 0/1/2) can forcibly preempt lower-tier execution at any time. This ensures that rare but critical tail events (e.g., emergency braking) always take priority over routine driving behaviors, regardless of what the lower-tier expert model suggests.

---

## Project Key Features
> *What HPN offers — a quick overview of system capabilities*

| Feature | Description |
|:---|:---|
| ⚖️ **Hybrid Architecture** | Combines modular safety verifiability with end-to-end environmental adaptability |
| 🧩 **Team-of-Experts** | Multiple specialized models, each mastering one driving primitive |
| 🛡️ **Safety Gateway** | PGV enforces hard-coded traffic rules before any vehicle actuation |
| 🔍 **Full Interpretability** | Arbiter explicitly outputs the active primitive; every decision is traceable |
| 🧠 **Shared BEV Backbone** | Unified multi-sensor fusion avoids redundant perception computation |
| 🔄 **FSM-Driven Logic** | State transitions between primitives are clear, deterministic, and auditable |
| 📈 **Built-In Evaluation** | Collision rate, success rate, TTC metrics, and long-tail robustness statistics |

---

## HPN Theoretical Advantages

> *Why HPN works — the foundational principles behind the architecture*

---

### **① Solves the Long-Tail Defect**  
Imbalanced driving data is split across independent expert models. Tail scenarios are no longer suppressed by the overwhelming majority of routine driving data — each expert masters its own distribution.

### **② Retains Formal Safety Guarantees**  
The PGV layer adds hard safety constraints that cannot be overridden by any neural network output — a guarantee pure end-to-end systems cannot provide.

### **③ Enables Complete Traceability**  
The Arbiter explicitly selects the active primitive. Any accident or failure can be pinpointed to a specific component: BEV perception → Arbiter decision → Expert policy → PGV validation.

### **④ Supports Modular Evolution**  
Each tier's expert can be updated, replaced, or retrained independently — without revalidating the entire driving stack. This enables parallel team development just like traditional modular pipelines.

### **⑤ Reduces Computational Overhead**  
A single shared BEV perception backbone serves all experts, avoiding redundant multi-sensor fusion computation across the entire system.

---

