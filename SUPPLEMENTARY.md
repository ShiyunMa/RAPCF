# Supplementary Materials

Due to space limitations, this document provides detailed DRL hyperparameters along with non-confidential information for the test systems and core models used in the study on proactive mitigation of reverse power flow (RPF) under PV-surplus events.

---

## 1. DRL Hyperparameters

| Hyperparameter | Value |
|---|---|
| Algorithm | Soft Actor-Critic (SAC) |
| Learning rate (actor) | 3 × 10⁻⁴ |
| Learning rate (critic) | 3 × 10⁻⁴ |
| Discount factor (γ) | 0.99 |
| Soft update coefficient (τ) | 0.005 |
| Replay buffer size | 1 × 10⁶ |
| Batch size | 256 |
| Hidden layers | 2 |
| Hidden units per layer | 256 |
| Activation function | ReLU |
| Optimizer | Adam |
| Entropy coefficient (α) | Auto-tuned |
| Training steps | 1 × 10⁶ |
| Warm-up steps | 1 × 10⁴ |
| Update frequency | 1 (per environment step) |
| Gradient clipping | 1.0 |

---

## 2. Test System Information

### 2.1 Distribution Network

| Parameter | Value |
|---|---|
| Base voltage | 12.66 kV |
| Number of buses | 33 |
| Nominal frequency | 50 Hz |
| Total active load | 3.715 MW |
| Total reactive load | 2.300 MVAR |
| PV penetration level | Up to 150% of peak load |

### 2.2 PV Integration

| Parameter | Value |
|---|---|
| Number of PV units | 5 |
| PV bus locations | Buses 6, 13, 18, 25, 30 |
| Rated PV capacity per unit | 500 kW |
| Inverter power factor range | −0.95 to +0.95 |
| PV output profile | Real irradiance data (15-min resolution) |

### 2.3 Simulation Environment

| Parameter | Value |
|---|---|
| Simulation tool | OpenDSS (via `dss_python`) |
| Time step | 15 minutes |
| Episode length | 96 steps (24 hours) |
| Training episodes | ~10,400 |
| Evaluation episodes | 365 (one full year) |

---

## 3. Core Model Architectures

### 3.1 Actor Network

```
Input layer  : state dimension (s)
Hidden layer 1: 256 units, ReLU
Hidden layer 2: 256 units, ReLU
Output layer : action dimension (a), Tanh
```

The actor outputs a squashed Gaussian policy. Actions are re-scaled from [−1, 1] to the physical control range of each PV inverter reactive power setpoint.

### 3.2 Critic Network (Twin Q-networks)

Two independent critic networks with identical structure are used to mitigate overestimation bias (TD3/SAC style):

```
Input layer  : state + action dimension (s + a)
Hidden layer 1: 256 units, ReLU
Hidden layer 2: 256 units, ReLU
Output layer : 1 unit (Q-value), linear
```

### 3.3 State Space

| Feature | Description |
|---|---|
| Bus voltages | Per-unit voltage magnitude at all 33 buses |
| PV active power | Normalised active power output of each PV unit |
| PV reactive power | Normalised reactive power output of each PV unit |
| Time encoding | sin/cos encoding of time-of-day |

### 3.4 Action Space

| Feature | Description |
|---|---|
| PV reactive power setpoints | Continuous, one per PV unit; normalised to [−1, 1] |

### 3.5 Reward Function

The reward at each time step is defined as:

```
r = −λ_v · Σ max(0, |V_i − 1.0| − Δ_v)²
  − λ_r · Σ |RPF_j|
  − λ_p · Σ Q_k²
```

| Symbol | Description | Value |
|---|---|---|
| λ_v | Voltage violation penalty weight | 100 |
| Δ_v | Voltage deviation tolerance (p.u.) | 0.05 |
| λ_r | RPF penalty weight | 10 |
| λ_p | Reactive power usage penalty weight | 0.1 |
