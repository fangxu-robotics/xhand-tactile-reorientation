# Engineering log

Nine changes, in the order they were made. Every one of them was found by probing the simulation
directly — reading body poses, dumping contact matrices, replaying single episodes — rather than by
tuning PPO hyperparameters. The PPO configuration in [methods.md](methods.md) is essentially the
Isaac Lab default and was never the bottleneck.

---

### 1 · Environment — the hand was hanging fingers-down

The asset's identity rotation put the palm sideways and the fingers pointing at the floor, so what the
reward called a "grasp" was really a hook. Probing body poses gave the base-frame axes, and a fixed
quaternion now maps fingers forward, palm up.

**Effect:** the sponge rests in the palm — 1 cm of settle over 2 s, palm contact 0.285 N, exactly the
sponge's weight.

---

### 2 · Termination — the drop threshold fired on a resting sponge

"Dropped" meant more than 15 cm from the hand base. The resting sponge already sat 10 cm away, so any
motion at all terminated the episode. Raised to 25 cm.

**Effect:** Stage 1 went from 100 % drops and 2.6-step episodes to 85 % hold success in 300
iterations. This is the dashed grey line in `fig2_curriculum_stage12`, and the single largest step in
the whole project.

---

### 3 · Observation — palm contact read 20 kN

The vendor collision meshes embed the finger knuckles inside the palm hull. With self-collision on a
fixed base, the unfiltered contact reading produced enormous forces. The observation now reads the
sponge-filtered force matrix, and self-collision is disabled.

**Effect:** contact observation back on the correct 0–15 N scale.

---

### 4 · Reward — the policy learned to throw the sponge

The alignment reward was a raw cosine in [−1, 1]. When the requested face started up, the policy
earned about −1 per step for 240 steps, versus a one-time −50 for dropping. Dropping on purpose was
the arithmetically optimal strategy, and the policy found it: a green-only test gave **0 successes in
363 episodes**.

Fix: `(1 + cos)/2`, so the reward is bounded in [0, 1] and never worse than zero; the angular-velocity
penalty was also cut five-fold.

**Effect:** drops on flip requests stopped being optimal. This is the clearest example in the project
of a reward bug that looks like a learning failure.

---

### 5 · Curriculum — the sponge never spawned mid-flip

Spawn tilt was ±0.2 rad, so the policy never observed the intermediate orientations it would have to
pass through to complete a flip. Spawn roll is now uniform over ±180°, matching Isaac Lab's Allegro
reference.

**Effect:** success climbed from 15 % to 22 % — real, but still nowhere near a working flip. This is
what pointed at the grasp geometry rather than the learning setup.

---

### 6 · Grasp geometry — long axis along the thumb · 2026-09-08 19:20

With the sponge's short axis along the thumb, the fingers had no long face to roll against. The sponge
was rotated 90° so the four fingers wrap its long side and the flip becomes a roll about that axis;
the thumb default pose was opened up to make room. Episode length grew from 8 s to 12 s.

**Effect:** both Stage 3 runs reach 60–70 % hold success with under 15 % drops (`fig1_stage3_curves`),
against 23 % / 56 % for the short-axis chain. This is the current configuration.

---

### 7 · Infrastructure — both runs killed by the memory watchdog · 2026-09-08 19:43

Two 4096-environment simulators in one terminal reached 23 GB on a 32 GB machine with no swap;
`systemd-oomd` killed the desktop session and took both runs with it. Runs were resumed sequentially
from their last checkpoints as detached systemd services, with swap added.

**Effect:** no loss — one simulator alone steps at 126k env-steps/s, while two sharing the GPU get 51k
each (`fig3_throughput`), so sequential runs cost nothing in total wall-clock time. The hollow markers
in `fig1_stage3_curves` are the resume points.

---

### 8 · Curriculum — Stage 4 under full randomization · 2026-09-08 22:52

Run B's checkpoint was continued for 1500 iterations with the widest mass, friction, actuator-gain,
control-delay and initial-grasp-offset ranges.

**Effect:** 70 % hold success, 9 % drops — no loss against Stage 3. Demo videos: 49 of 50 flips on the
hand grid, 10 of 10 on the arm at the close and top cameras.

---

### 9 · Hardware prep — open-loop trajectory export · 2026-09-09

For a first real-hand demo without perception, the commanded finger targets of successful simulated
flips were exported and replayed open-loop on 64 freshly randomized sponges. The most replayable of 12
candidate episodes was kept.

Two things that do **not** work: rate-limiting the targets, and replaying measured joint angles
instead of commands. In both cases the sponge is never gripped hard enough — the over-shooting
commanded targets are the squeeze.

**Effect:** the Stage 4 policy reaches 48 % open-loop success in sim, against 34–38 % for the Run B
policy. Exported with a numpy-only player — no Isaac, no PyTorch — with dry-run, XHand SDK and ROS 2
backends, packaged as a single deployment bundle for the robot PC.
