# Viking Engine: Autonomous GPU Overclocking via Workload Quality

**Discovery: Code changes the behavior of the Windows frequency scheduler**

*Orakul Studio - Chernihiv, Ukraine 🇺🇦*

---

## Experimental Conditions

- GPU: RTX 4090
- Windows Mode: **P2 (power saving)**
- Core Unlocking: **None**
- Overclocking via OC Software: **None**
- Config: Rank 128, Alpha 64, 768×768 dataset

---

## What Happened

```
PowerShell → nvidia-smi:
Performance State: P2 ← power saving mode

Training Log:
[41.53W 210MHz] ← GPU sleeps until startup
[49.39W] 2550MHz] ← caching latencies
[243.47W 2790MHz] ← training started
[244.58W 2775MHz] ← step 41: 8.38 s/it
[276.87W 2775MHz] ← step 44: 8.35 s/it ← still dropping
```

**P2 — but the frequencies are 2775-2790MHz. This is almost the maximum boost clock of the 4090.**

---

## Why it works

NVIDIA's Boost Algorithm doesn't look at P-states, but at GPU utilization.

Standard trainer: GPU computes a layer → waits for weights → computes again.
During pauses, utilization drops → boost reduces frequencies.

Viking Engine with double buffering: **no pauses**. Transfer runs in parallel with compute. The GPU sees 100% utilization continuously, and boosts frequencies automatically, ignoring P2 limitations.

```
Default Trainer: ████░░░████░░████░░░████░░░
↑↑↑↑ Pauses reduce clock rates

Viking Engine: █████████████████████████
← GPU detects this → boost to maximum
```

---

## Result

| Parameter | Value |
|---|---|
| P-state Windows | P2 (power saving) |
| Unlock cores | No |
| Actual frequencies | 2775-2790 MHz |
| Power consumption | 240-280W (vs 450W TDP) |
| Speed ​​rank 128 | **8.35 s/it** (stabilizing) |

The GPU operates at maximum frequencies using **55% TDP** in power saving mode.

<img width="3840" height="2160" alt="Снимок экрана (372)" src="https://github.com/user-attachments/assets/2f8a5dda-61dd-441b-9efb-f8f162273455" />

[log_LubR128.txt](https://github.com/user-attachments/files/28821720/log_LubR128.txt)

---

## First run (unlimited)

In the first run without power saving, the GPU automatically entered **P0**—the highest performance state—without any manual intervention.

The Viking Engine creates such a dense and predictable compute load that the NVIDIA boost algorithm scheduler automatically boosts the GPU to its maximum state.

---

## Conclusion

**Viking Engine doesn't just speed up training.**
It changes how the GPU interacts with the hardware-level frequency scheduler.

The code itself overclocks the hardware—through the quality of the load, not through OC tools.

---

*Recorded: June 2026*
*Chernihiv, Ukraine 🇺🇦 · Orakul Studio · 🦊⚡*
