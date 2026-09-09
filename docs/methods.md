# Methods

## Task definition

| | |
|---|---|
| Object | Rigid cuboid, 10 × 6.5 × 3.2 cm, 30 g. Two visual faces (green scrub / yellow foam), identical physics. |
| Goal | Each episode names a target face. Turn the sponge until that face points down and hold it, grasped, for 2 continuous seconds. |
| Success | Requested face within 20° of straight down **and** ≥ 1 contact sensor touching the sponge, for 60 consecutive control steps (2 s at 30 Hz). |
| Drop | Sponge more than 25 cm from the palm. Ends the episode. |
| Episode cap | 360 control steps (12 s). |

When the requested face starts up, the episode is a full 180° flip about the sponge's long axis. When
it starts down, the task reduces to maintaining the grasp.

## Assets

No hand-only asset is published for the XHand1, so the left hand was **extracted from RobotEra's STAR1
humanoid URDF** and converted to USD: 12 revolute joints, 13 bodies.

Two environments share that asset and the same observation layout:

- **Fixed-wrist** — the hand alone, base fixed, palm up. Used for all training.
- **Franka-mounted** — the same hand bolted to a Franka Panda. Used only for replay/demonstration.
  Checkpoints transfer with no retraining and no modification.

## Observation space

56 dimensions, defined in three tiers so the same environment supports a privileged teacher, a
deployable student, and a proprioception-only ablation:

| Tier | Dims | Contents |
|---|---|---|
| Proprioception | 37 | joint positions, velocities, last action, target-face encoding |
| + contact | 43 | fingertip and palm contact forces (6) |
| + privileged (teacher) | 56 | object pose, linear/angular velocity, face normals (13) |

All training reported here uses the full 56-D privileged teacher. The 43-D student is future work.

## Action space

12 **relative** joint-position deltas, clipped to 0.1 rad per control step, applied on top of the
current targets. Relative targets mean a zero action leaves the hand compliant rather than snapping
to a nominal pose. A random control delay of 0–2 steps is applied.

## Reward

| Term | Weight | Notes |
|---|---|---|
| Face alignment | ×1.0 | `(1 + cos θ)/2`, where θ is the angle between the requested face normal and straight down. **Not** raw cosine — see [engineering_log.md](engineering_log.md) item 4. |
| Hold progress | ×2.0 | fraction of the 2 s hold completed |
| Grasp contact fraction | ×0.5 | fraction of contact sensors touching the sponge |
| Success bonus | +250 | one-time, on completing the hold |
| Drop penalty | −50 | one-time, on termination by drop |
| Penalties | — | contact force > 15 N, object linear/angular velocity, joint-limit violation, action rate |

## Algorithm

PPO from `rsl_rl`:

```
envs                4096
steps/env/iter      24          →  98 304 env-steps per iteration
epochs              5
minibatches         4
clip                0.2
gamma               0.998
lambda              0.95
learning rate       adaptive from 1e-3, KL target 0.01
entropy coef        0.002
network             actor & critic MLP 512·256·128, ELU
obs normalization   on
initial action std  1.0
```

Physics at 120 Hz, control at 30 Hz, environments spaced 0.5 m apart, headless.
Throughput on one RTX 4090: ~126k env-steps/s, ~1 s per iteration.

## Curriculum

Four stages, each resumed from the previous checkpoint.

| Stage | Spawn roll | Target face | Iterations |
|---|---|---|---|
| S1 | ±0.1 rad | yellow only | 300 |
| S2 | ±0.6 rad | yellow only | 400 |
| S3 | any roll about the long axis (±180°) | both | 4000 |
| S4 | as S3, with mass, friction, actuator gains, control delay and initial-grasp offsets at their widest ranges | both | 1500 |

Stages 1 and 2 together finish in about 12 minutes. Stage 3 is where the compared runs spend their
time. Stage 4 verifies the policy survives full randomization — the robustness a hardware attempt
needs.

## Domain randomization

Object mass, friction, collider offsets, actuator stiffness and damping, observation noise, control
delay, initial object pose, initial grasp. Object scale randomization is implemented but disabled for
speed.

## Grasp geometry

The current configuration places the sponge's **long axis along the extended thumb**, with the four
fingers wrapping the long side, so the flip is a roll about the axis the fingers can drive. The thumb
default pose was opened up to make room. This single change is what took Stage 3 from 23 % to 60–70 %
hold success — see [engineering_log.md](engineering_log.md) item 6.

## Open-loop trajectory export (hardware prep)

For a first real-hand demo without perception, the *commanded* finger targets of successful simulated
flips were exported and replayed open-loop on 64 freshly randomized sponges; the most replayable of 12
candidate episodes was kept.

Rate-limiting the targets, or replaying measured joint angles instead of commands, both fail — the
over-shooting targets are what produces the squeeze.

| Policy | Open-loop success in sim |
|---|---|
| Stage 4 `model_6198` | **48 %** |
| Run B `model_4699` | 34–38 % |

The trajectory is exported with a numpy-only player (no Isaac, no PyTorch) offering three backends —
dry-run, XHand SDK, and ROS 2 — so the bundle runs on the robot PC unmodified.
