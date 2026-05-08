# 04 — Group — Stereo, × 3

---

## 4.1 Group summing bus

Upstream of each group is a **stereo summer** (simple summing amp,
L and R separately, fixed gain). It sums all contributions routed to
that group from channels and AUX returns (via their 4-pos rotary
switches).

---

## 4.2 Insert

- **Send** — always active, impedance-balanced TRS.
- **Return** — TRS jack, unbalanced fixed-gain receiver (simpler than
  the balanced receivers on channels — just single-ended on each of
  L and R).
- **Insert switch** — mechanical. Alternates the channel's input
  between the summing-bus output and the return jack. When "IN", the
  return overrides the summing bus entirely.

---

## 4.3 MONO switch

Same as on AUX return — premix L+R to both lines.

---

## 4.4 Meter buffers (× 2), ACTIVE electronic switch (stereo)

Same as AUX return. The ACTIVE switch is a single
**AGQ210A03** latching signal relay (DPDT 2 form C, 1-coil
latching) handling L and R simultaneously on its two contact sets
— see §3.4 and `00-conventions.md` "Standard signal relay".

---

## 4.5 Stereo fader + post-fader amp (+10 dB) + active balance

Same as AUX return.

---

## 4.6 Post-balance distribution

Differences vs AUX return:

- **No 4-pos rotary**. Groups can only be routed to the **Main Mix**.
  A **mechanical on/off switch** enables or disables the Main send.
- **AFL** electronic switch (stereo) → AFL bus. One
  **AGQ210A03** per group; contact set 1 carries L, contact
  set 2 carries R, ganged on one coil. Per `00-conventions.md`
  "Standard signal relay".
- **4 × AUX dual-gang pots** with an important twist: each AUX has
  its own **mode switch**, and there are therefore **4 mode switches
  per group** (one per AUX). Each mode switch is shared between the
  pre-gang and the post-gang of that AUX. Modes:
  - **Mode 1 ("split")**: per AUX "n", in this mode L of the group
    feeds AUX n (the "left" of a stereo AUX pair) and R feeds the
    adjacent (paired) AUX. Globally, with all four AUX mode switches
    in mode 1, the effect is: L → AUX 1 and AUX 3, R → AUX 2 and
    AUX 4. So per-AUX, only one of L or R reaches that AUX (the
    other side of the stereo signal goes to the paired AUX).
  - **Mode 2 ("premix")**: L + R are summed and the mono result
    feeds the AUX (same on pre and post gangs).
  - Each mode switch acts identically on both gangs of its AUX (pre
    and post carry the same mode).
- **CUE dual-gang** (pre-fader, stereo) → CUE stereo bus.
- **PFL stereo switch** (pre-fader) → PFL stereo bus.
- **Post-Fader Outputs × 2** — L and R, TRS impedance-balanced.

---

## 4.7 Not present on Group

- No HPF.
- No Direct Out (groups aren't input channels).
- No 4-pos rotary (groups only go to Main).

---

## 4.8 Implementation details

Status: **conceptual**, with one part-level decision already
fixed: the ACTIVE and AFL electronic switches both use the
**AGQ210A03** latching signal relay per `00-conventions.md` —
2 relays per group × 3 groups = 6 relays from this section. No
topology details for the audio summing / insert / fader / balance
stages chosen yet.

*(Key unique pieces to record when implemented: the per-AUX mode
switch network — how split vs premix is physically realized in the
dual-gang pot wiring — and the group summing amp upstream.)*
