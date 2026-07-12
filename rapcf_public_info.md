# RAPCF Public Information
This document summarizes the public, non-confidential experiment settings for RAPCF. Raw grid models, customer-level traces, identifiable device names, and proprietary solver files should not be published.

## 1. DRL Hyperparameters
| Item | Value |
| --- | --- |
| seed | 22 |
| device | cuda |
| horizon | 96 |
| learning rate | 1e-05 |
| feature dimension | 64 |
| full-day episode | True |
| policy | MultiInputPolicy |
| training episodes | 50000 |
| evaluation frequency | 50 |
| RL backend | Stable-Baselines3 + sb3-contrib |
| feature extractor | HMPGNN-based topology encoder |

| n_epochs | 3 |
| gamma | 0.99 |
| n_steps | 96 |
| batch_size | 32 |
| vf_coef | 0.7 |
| ent_coef | 0.003 |
| gae_lambda | 0.98 |
| clip_range | 0.15 |
| target_kl | 0.01 |
| max_grad_norm | 0.5 |
| normalize_advantage | True |

## 2. Shaoxing real-world distribution system Summary
| Item | Count / Value |
| --- | --- |
| AC lines | 25 |
| feeders | 21 |
| transformers | 11 |
| PV units | 2 |
| BESS units | 4 |
| DR resources | 3 |
| tie switches | 15 |
| sectionalizing switches | 48 |
| total switch controls | 63 |
| valid switch-pair candidates | 161 |
| operating samples | 5844 |
| train / test split | 5760 / 84 |
| feature cache files | 5844 |

Control-resource limits:
| Resource | Public parameters |
| --- | --- |
| BESS | soc_init=0.5, soc_min=0.15, soc_max=0.9, eta_ch=0.95, eta_dis=0.95, p_max=0.3 p.u. |
| DR | p_max=1.0, e_shift_max=1.0, min_step=4 |
| PV | p_max=2.5 |
| time interval | 0.25 h |
| voltage limit | [0.95, 1.05] p.u. |
| line active-power limit | 1.0 p.u. |

## 3. Core Model Architecture
| Module | Public architecture |
| --- | --- |
| HMPGNN node encoder | Linear(9,64)-ReLU-Linear(64,64) |
| HMPGNN edge encoder | Linear(8,64)-ReLU-Linear(64,64) |
| message passing | 2 layers, mean aggregation |
| message MLP | Linear(192,64)-ReLU-Linear(64,64)-ReLU |
| update MLP | Linear(128,64)-ReLU-Linear(64,64) |
| graph pooling | mean pooling over nodes |
| SB3 feature output | 64 dimensions |
| RGNN readout | edge readout with 5-step risk output including current step and four future steps |

## 4. Reward Decomposition
The implementation reward is an expanded normalized form of Eq. (3), where each term maps to safety/economic/proactive components.

Step reward weights:
| Term | Weight |
| --- | --- |
| rev_mitigation | 1.0 |
| voltage | 5.0 |
| overflow | 5.0 |
| island_addition | 5.0 |
| loop_addition | 5.0 |
| outage_addition | 5.0 |
| action_eco | 0.01 |
| switch_action | 2 |
| bess_action | 0.1 |
| dr_action | 0.1 |
| Pv_action | 0.2 |
| act_pro | 0.5 |
| solved_step | 0.01 |

Daily reward weights:
| total_mitigation | 1.0 |
| total_safety | 1.0 |

All reward terms are normalized before weighted aggregation. In addition, each step reward is divided by 96, corresponding to the number of dispatch intervals in a full-day episode, so that the accumulated episode reward can be expressed on a normalized daily scale.

## 5. Information Not Recommended for Public Release
- Raw `.etext` grid model files.
- Customer-level load traces or identifiable customer/resource names.
- Raw Excel/JSON files containing true device IDs, geographical information, or operation-sensitive topology details.
- Proprietary power-flow solver binaries or licensed interfaces.
- Any data that can identify a real feeder, transformer, switch, customer, or dispatch strategy.