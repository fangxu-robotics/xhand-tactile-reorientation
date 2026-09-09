# Code

The Isaac Lab task, training scripts and replay tooling for this project are not published in this
repository yet.

What the code consists of:

- **Environment** — an Isaac Lab manager-based RL task
  (`Isaac-XHand-Reorient-Sponge-Left-v0`) with the 56/43/37-D observation tiers, the reward terms in
  [docs/methods.md](../docs/methods.md), and the four curriculum stages.
- **Arm-mounted variant** — `Isaac-XHand-Reorient-Sponge-Left-Franka-v0`, the same task with the hand
  bolted to a Franka Panda. Used only for replay; exposes the identical joint and observation layout
  so checkpoints transfer unchanged.
- **Asset pipeline** — extraction of the XHand1 left hand from RobotEra's STAR1 humanoid URDF and
  conversion to USD (12 revolute joints, 13 bodies).
- **Training** — `rsl_rl` PPO, run as detached `systemd` units per curriculum stage.
- **Replay and recording** — `*-Play-v0` variants with a two-tone sponge and camera presets (`grid`,
  `close`, `top`, `wide`), which produced every clip in the two demo videos, together with the
  per-clip replay statistics quoted in [docs/demos.md](../docs/demos.md).
- **Open-loop export** — `export_trajectory.py` records closed-loop flips, scores 12 candidate
  episodes by replaying each open-loop on 64 freshly randomized sponges, and exports the best
  commanded-target trajectory.
- **Deployment player** — `play_trajectory.py`, numpy-only (no Isaac, no PyTorch), with three
  backends: dry-run, XHand SDK, and ROS 2. Shipped together with the trajectory as a self-contained
  bundle for the robot PC.

Dependencies: Isaac Sim 5.1.0, Isaac Lab 0.54.4, `rsl_rl`, PyTorch, one CUDA GPU with ≥ 8 GB VRAM.
