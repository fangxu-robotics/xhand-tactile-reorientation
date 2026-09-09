# Data

## `training_logs/`

Per-iteration `rsl_rl` training logs, parsed from the run directories under `logs/`. One row per
iteration; all rate columns are fractions of completed episodes in that iteration across the 4096
parallel environments.

| File | Run | Iterations | Rows |
|---|---|---|---|
| `stage1_run1_prefix.csv` | Stage 1, before the drop-threshold fix | 0–226 | 227 |
| `stage1_run2.csv` | Stage 1 | 0–316 | 317 |
| `stage2.csv` | Stage 2 | 300–699 | 400 |
| `stage3_chain.csv` | earlier Stage 3, short axis along thumb | 699–3198 | 2500 |
| `stage4_chain.csv` | earlier Stage 4 continuation of the same chain | 3198–3943 | 746 |
| `yaw90_scratch.csv` | Run A, first segment | 0–1766 | 1767 |
| `yaw90_scratch_r2.csv` | Run A, resumed after the OOM kill | 1750–3999 | 2250 |
| `yaw90_warm.csv` | Run B, first segment | 699–2439 | 1741 |
| `yaw90_warm_r2.csv` | Run B, resumed after the OOM kill | 2400–4699 | 2300 |
| `yaw90_s4.csv` | Stage 4, continued from Run B | 4699–6198 | 1500 |

Columns:

```
iteration            training iteration index
total                iteration index the run counts up to
hold_success         fraction of episodes that completed the 2 s hold
dropped              fraction of episodes ended by the drop condition
time_out             fraction of episodes that hit the 360-step cap
r_face_alignment     mean per-step face-alignment reward, (1+cos)/2 in [0, 1]
r_hold_bonus         mean one-time +250 success bonus
r_dropped_penalty    mean one-time −50 drop penalty
r_hold_progress      mean hold-progress shaping reward
r_grasp_contact      mean contact-fraction reward
r_excess_force       mean penalty for contact forces above 15 N
r_joint_limits       mean joint-limit penalty
mean_reward          mean total episode return
mean_ep_len          mean episode length in control steps (30 Hz, cap 360)
iter_time_s          wall-clock seconds for the iteration
steps_per_s          environment steps per second
invalid_state        count of invalid physics states
value_loss           PPO critic loss
surrogate_loss       PPO clipped surrogate loss
action_std           current policy action standard deviation
```

`curves.json` holds the smoothed series (`series`) and the last-100-iteration means (`summary`) used
to generate the figures in `assets/figures/` and the tables in the README.

## Not included

Model checkpoints (`model_*.pt`), USD assets, and raw TensorBoard event files are not published here.
Ask if you need them.
