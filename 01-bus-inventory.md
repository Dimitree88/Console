# 01 — Bus inventory

Total: **7 stereo bus + 9 mono bus** = 16 distinct buses, i.e. 23
summing lines to route and solder.

---

## Stereo buses (7)

| # | Name | Fed by |
|---|------|--------|
| 1 | Main Mix | Channels (via 4-pos rotary), AUX returns (via 4-pos rotary), groups (via main on/off switch) |
| 2 | Group 1 | Channels, AUX returns (via 4-pos rotary) |
| 3 | Group 2 | Channels, AUX returns (via 4-pos rotary) |
| 4 | Group 3 | Channels, AUX returns (via 4-pos rotary) |
| 5 | AFL | All channels / AUX returns / groups (via AFL electronic switch, post-fader, post-pan/post-balance) |
| 6 | CUE | Channels, AUX returns, groups (via CUE dual-gang pot, pre-fader) |
| 7 | PFL stereo | AUX returns and groups (via PFL stereo switch, pre-fader). Mono channels do NOT feed this bus. |

---

## Mono buses (9)

| # | Name | Fed by |
|---|------|--------|
| 1 | AUX 1 pre | 24 channels + 3 groups (pre-fader gang of AUX1 dual-gang). AUX returns do NOT feed any AUX bus. |
| 2 | AUX 1 post | 24 channels + 3 groups (post-fader gang of AUX1 dual-gang). AUX returns do NOT feed any AUX bus. |
| 3 | AUX 2 pre | 24 channels + 3 groups |
| 4 | AUX 2 post | 24 channels + 3 groups |
| 5 | AUX 3 pre | 24 channels + 3 groups |
| 6 | AUX 3 post | 24 channels + 3 groups |
| 7 | AUX 4 pre | 24 channels + 3 groups |
| 8 | AUX 4 post | 24 channels + 3 groups |
| 9 | PFL mono | 24 mono channels (PFL switch, pre-fader) + 4 AUX master PFL pushes |

Note: **PFL mono and PFL stereo are separate buses.** They are merged
only downstream at the PFL summer inside the master monitor (see
`08-monitor.md`).

**No AUX return feeds back into any AUX bus** — deliberate, to avoid
feedback loops. The AUX returns do NOT have AUX sends.

---

## Logical buses (1)

These are NOT audio buses — they are digital wired-OR lines monitored
by firmware.

| # | Name | Fed by |
|---|------|--------|
| 1 | PFL DETECT | 24 mono channels (per-channel diode wired-OR from the PFL DPDT Sec2). Whether AUX masters / AUX returns / groups also feed this bus to drive the master-monitor source-select is currently TBD — see `10-open-tbd.md`. |

PFL DETECT goes HIGH whenever at least one PFL switch anywhere on
the console is engaged. Firmware reads it to drive the master
monitor source-select (§8.1) — switching the monitor source from
Main Mix to the Solo summer when any PFL is active.

---

## §11 — Counts (quick reference)

| Item | Count |
|------|-------|
| Mono input channels | 24 |
| Stereo AUX returns | 4 |
| Stereo groups | 3 |
| AUX masters (mono) | 4 |
| AUX buses (mono) | 8 (4 pre + 4 post) |
| Stereo buses | 7 (Main, G1, G2, G3, AFL, CUE, PFL stereo) |
| Mono buses | 9 (8 AUX + PFL mono) |
| Logical buses | 1 (PFL DETECT — wired-OR digital, not a summing line) |
| Total summing lines | 7 stereo × 2 lines + 9 mono × 1 line = **23** |
| Main outputs | 2 pairs (A + B), 4 TRS jacks total |
| Insert points | 24 (channels) + 3 (groups) + 1 (main mix) = 28 |
| Direct Outs | 24 (full-differential, on Input A of each mono channel) |
| Post-Fader Outs | 24 (channels, mono) + 6 (3 groups × L/R) = 30. (AUX returns do NOT have Post-Fader Outs.) |
| AUX master OUT jacks | 4 (one per AUX master, TRS impedance-balanced) |
| CUE master OUT jacks | 2 (L + R, TRS impedance-balanced) |
