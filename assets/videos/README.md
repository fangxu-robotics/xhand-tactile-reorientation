# Demo clip metadata

The demo videos are hosted on YouTube, not committed here — MP4s bloat a git repository and are slow
to clone:

- **[Main demo](https://youtu.be/teRxEV7sdbk)** (2:19) — the Stage 4 policy on four camera setups
- **[Policy comparison](https://youtu.be/-kGYakc2aPo)** (5:05) — all twelve clips, grouped by policy

What remains in this directory is one `*_summary.txt` per clip, written by the recording script. Each
records exactly what produced that clip:

```
task=Isaac-XHand-Reorient-Sponge-Left-Play-v0
checkpoint=.../xhand1_left_reorient_sponge_stage4/2026-09-08_22-52-22_yaw90_s4/model_6198.pt
arm_motion=none
cam=grid
target_face=green
video_length_steps=600 (dt=0.0333s)
steps=600
hold_success=49
dropped=1
time_out=0
episodes=50
```

These are the source of every per-clip number quoted in the README and in
[docs/demos.md](../../docs/demos.md).
