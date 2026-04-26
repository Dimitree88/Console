# 06 — CUE Master

Simple stereo master for the CUE bus:

1. **Stereo summer** — sums L and R separately (L contributions →
   CUE L summer, R contributions → CUE R summer).
2. **Stereo gain pot** — one knob, both gangs ganged (L and R move
   together).
3. **CUE OUT** — 2× TRS impedance-balanced (L + R).

No PFL on the CUE master (deliberate — you'd monitor CUE via its own
output if needed).

Intended use: musicians' headphones feed (wet or dry monitor mix for
the performers).

---

## 6.4 Implementation details

Status: **conceptual**. No topologies, parts, or values chosen yet.

*(Simpler than AUX master — just a stereo summer + stereo gain stage.
Likely shares the summing amp topology decided in `05-aux-master.md`.)*
