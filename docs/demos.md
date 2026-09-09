# Demo index

Both videos are on YouTube (unlisted). Every clip in them is a raw policy replay — no cuts, no speed
changes, no cherry-picked episodes within a clip.

| Video | Length | Contents |
|---|---|---|
| **[Main demo](https://youtu.be/teRxEV7sdbk)** | 2:19 | The Stage 4 policy on all four camera setups |
| **[Policy comparison](https://youtu.be/-kGYakc2aPo)** | 5:05 | All twelve clips, grouped by policy |

The MP4s are deliberately not committed to this repository. What lives in
[`../assets/videos/`](../assets/videos) is the `*_summary.txt` for each clip: the exact task id,
checkpoint path, camera preset and replay statistics it was generated from.

All clips are 600 control steps (20 s at 30 Hz). Episodes are counted across the parallel
environments in the clip, so a 16-hand grid accumulates many more episodes than a single
arm-mounted hand.

## Naming

The `*_summary.txt` files follow the recording script's naming:

```
final_yaw90_<policy>_<platform>_<camera>_<target>
             │        │          │         └─ green = always a 180° flip; alternate = faces alternate
             │        │          └─ grid (16 hands) · close · top · wide
             │        └─ hand (fixed wrist) · franka (arm-mounted)
             └─ s4 = Stage 4 model_6198 · warm_r2 = Run B model_4699 · scratch_r2 = Run A model_3999
```

`yaw90` refers to the current grasp geometry: the sponge rotated 90° so its long axis lies along the
extended thumb.

## Video 1 — main demo (2:19)

[Watch](https://youtu.be/teRxEV7sdbk) · Stage 4 policy `model_6198` only.

| Time | Section |
|---|---|
| [0:00](https://youtu.be/teRxEV7sdbk) | The task |
| [0:19](https://youtu.be/teRxEV7sdbk?t=19) | 16 hands in parallel — every episode a 180° flip |
| [0:44](https://youtu.be/teRxEV7sdbk?t=44) | Same checkpoint on a Franka-mounted hand |
| [1:09](https://youtu.be/teRxEV7sdbk?t=69) | Top-down view |
| [1:34](https://youtu.be/teRxEV7sdbk?t=94) | Robustness — the arm keeps moving |
| [1:59](https://youtu.be/teRxEV7sdbk?t=119) | Results |

## Video 2 — policy comparison (5:05)

[Watch](https://youtu.be/-kGYakc2aPo) · all three policies, four camera setups each.

### Stage 4 — `model_6198`, full domain randomization

| Time | Setup | Episodes | Success | Dropped |
|---|---|---|---|---|
| [0:28](https://youtu.be/-kGYakc2aPo?t=28) | 16 fixed-wrist hands, green target — every episode a 180° flip | 50 | **49** | 1 |
| [0:48](https://youtu.be/-kGYakc2aPo?t=48) | Franka + XHand1, close-up | 5 | **5** | 0 |
| [1:08](https://youtu.be/-kGYakc2aPo?t=68) | Franka + XHand1, top-down | 5 | **5** | 0 |
| [1:28](https://youtu.be/-kGYakc2aPo?t=88) | Franka + XHand1, wide shot, arm swaying | 2 | 1 | 1 |

### Run B — `model_4699`, warm start from Stage 2

| Time | Setup | Episodes | Success | Dropped |
|---|---|---|---|---|
| [1:56](https://youtu.be/-kGYakc2aPo?t=116) | 16 fixed-wrist hands, green target | 35 | **35** | 0 |
| [2:16](https://youtu.be/-kGYakc2aPo?t=136) | Franka + XHand1, close-up | 6 | **6** | 0 |
| [2:36](https://youtu.be/-kGYakc2aPo?t=156) | Franka + XHand1, top-down | 6 | **6** | 0 |
| [2:56](https://youtu.be/-kGYakc2aPo?t=176) | Franka + XHand1, wide shot, arm swaying | 4 | 3 | 1 |

### Run A — `model_3999`, trained from scratch

| Time | Setup | Episodes | Success | Dropped |
|---|---|---|---|---|
| [3:25](https://youtu.be/-kGYakc2aPo?t=205) | 16 fixed-wrist hands, green target | 27 | 26 | 1 |
| [3:45](https://youtu.be/-kGYakc2aPo?t=225) | Franka + XHand1, wide shot, arm swaying | 2 | 2 | 0 |
| [4:05](https://youtu.be/-kGYakc2aPo?t=245) | Franka + XHand1, close-up | 0 | — | — |
| [4:25](https://youtu.be/-kGYakc2aPo?t=265) | Franka + XHand1, top-down | 0 | — | — |

Run A holds the sponge as reliably as Run B but works more slowly: in the last two clips it did not
finish a single episode inside the 20 s window, which is why those rows are empty. The clips still
show the behaviour — the hand is mid-flip when the recording ends.

## Still frames

| Frame | Shows |
|---|---|
| [`frame_hand_grid.jpg`](../assets/frames/frame_hand_grid.jpg) | 16 of the 4096 training hands, sponges at assorted orientations |
| [`frame_hand_grid_flip.jpg`](../assets/frames/frame_hand_grid_flip.jpg) | the same grid mid-flip |
| [`frame_franka_close.jpg`](../assets/frames/frame_franka_close.jpg) | Franka-mounted hand, close-up |
| [`frame_franka_wide.jpg`](../assets/frames/frame_franka_wide.jpg) | Franka-mounted hand, wide shot with the arm swaying |

## Earlier demos (not recorded to video)

From the Stage 2 policy (`model_699`), before the current grasp geometry — kept for reference:

| Run | Setup | Episodes | Success | Dropped |
|---|---|---|---|---|
| `fallback_hand_grid_tilt` | 16 fixed-wrist hands, yellow target, ±0.6 rad tilt | 97 | 62 | 35 |
| `fallback_franka_close_tilt04` | Franka + XHand1 close-up, ±0.4 rad tilt | 17 | 8 | 9 |
| `fallback_franka_close_tilt` | Franka + XHand1 close-up, ±0.6 rad tilt | 14 | 7 | 7 |
| `fallback_franka_wide_sway` | Franka + XHand1 wide shot, arm swaying | 13 | 6 | 7 |
| `interim1550_hand_grid` | old Stage 3 `model_1550`, green target, pre-reward-fix — the "throw" policy | 363 | **0** | 363 |

The last row is the raw-cosine reward bug from [engineering_log.md](engineering_log.md) item 4: the
policy discards the sponge on every single flip request.
