# 03 — AUX Returns — Stereo Input Channels (× 4)

There are **4 stereo AUX returns**, all identical in topology. They
are intended as stereo line-level inputs (effects returns, stereo
sources, etc.) that contribute to the internal buses but do **not**
have any output to the outside world.

Each AUX return follows the structure described below. References to
"the AUX return" or "this channel" mean a single one of the four —
multiply by 4 for the actual count.

---

## 3.1 Input stage

- Two balanced TRS inputs: **L** and **R**, each with its own
  differential buffer + unbalancing stage + variable gain.
- The gain control is a **stereo pot** (one knob, L and R gangs
  move together). No Direct Out.

## 3.2 MONO switch

Mechanical switch. When active, an opamp mixes L + R and the result
is replicated onto both L and R lines. When inactive, L and R remain
independent.

**Convention**: the channel always carries two lines (L and R)
throughout, even when MONO is engaged — it just happens that both
carry the same (premixed) signal in that case.

## 3.3 Meter buffers (× 2)

One buffer per line, feeding the LED meter bridge (stereo meter).

## 3.4 ACTIVE electronic switch (stereo)

A single relay-driven electronic switch (one **FTR-B3GA4.5Z-B10**
per AUX return — DPDT 2 form C, standard signal relay per
`00-conventions.md`) interrupts the stereo path for ACTIVE/SOLO
logic. Contact set 1 carries L, contact set 2 carries R; both are
mechanically ganged on a single coil so L and R commute
synchronously by construction. Coil energized ⇒ active; coil
de-energized (rest) ⇒ muted (fail-safe at power-up).

## 3.5 Stereo fader + post-fader amps

- Stereo fader (dual-gang).
- Post-fader amp: +10 dB on each of L and R.

## 3.6 Active balance

Instead of a mono pan, this is a **balance** control: attenuates one
side relative to the other. Active circuit, stereo pot.

## 3.7 Post-balance distribution

The AUX return is **internal-routing only** — there is no jack-out
destination. From the post-balance stereo signal, these destinations
tap in parallel:

- **4-pos rotary** → Main Mix / G1 / G2 / G3 (stereo). Selects
  exactly one of these four buses, mechanical rotary.
- **AFL electronic switch** (stereo) → AFL bus. One
  **FTR-B3GA4.5Z-B10** per AUX return; contact set 1 carries L,
  contact set 2 carries R, ganged on one coil. Per
  `00-conventions.md` "Standard signal relay".
- **CUE dual-gang** (pre-fader, stereo) → CUE stereo bus
  (L into CUE L, R into CUE R).
- **PFL stereo switch** (pre-fader) → PFL stereo bus.

## 3.8 Not present on AUX return

- No Direct Out.
- No HPF.
- No Insert.
- **No AUX sends** (AUX returns deliberately do not feed any AUX
  bus, to prevent obvious feedback loops).
- **No Post-Fader Output to the outside.** AUX returns contribute
  only to internal buses (Main / Groups / AFL / CUE / PFL stereo).
  If an external send of the AUX return content is desired, it must
  be taken from one of the buses it feeds (Main, a Group, etc.).

## 3.9 Implementation details

Status: **conceptual**, with one part-level decision already
fixed: the ACTIVE and AFL electronic switches both use the
**FTR-B3GA4.5Z-B10** signal relay per `00-conventions.md` —
2 relays per AUX return × 4 AUX returns = 8 relays from this
section. No topology details for the audio summing / fader /
balance stages chosen yet.

*(Many stages mirror the mono channel — once the mono channel is
implemented, the AUX return reuses most of its building blocks.
Record here only the parts that differ: the stereo gain pot, the
MONO opamp mixer, the active balance circuit. Note that the AUX
return strip is leaner than a mono channel: no HPF, no insert,
no Direct Out, no AUX sends, no Post-Fader Output — only the
internal routing destinations.)*
