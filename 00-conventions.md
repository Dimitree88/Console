# 00 — Conventions

Global design conventions and electrical/implementation conventions for
CONSOLE. These apply throughout the project unless a specific section
explicitly overrides them.

This file is referenced by every other section. Read it before reading
any of the section files (`02-mono-channel.md`, etc.).

---

## Design conventions (signal-flow vocabulary)

- All balanced line outputs are **impedance-balanced TRS** (cold pin
  terminated, not actively driven) unless stated otherwise.
- Balanced inputs use a **differential buffer** (two high-impedance
  buffers, one for hot, one for cold) followed by an **unbalancing
  stage**.
- "Variable gain" = potentiometer controlling the gain of an active
  stage.
- "Fixed gain" = no user control, gain set by circuit.
- "Electronic switch" = analog switch (e.g., JFET / CMOS) controlled
  by digital logic. Used for SOLO, ACTIVE, AFL enable — anywhere the
  signal path is interrupted or rerouted under firmware control.
- "Mechanical switch" = front-panel toggle / push / rotary, direct
  user control.
- "Dual-gang pot" = one knob, two independent pot sections (gangs) —
  used for AUX sends (one gang picks up pre-fader, the other
  post-fader simultaneously) and for stereo CUE sends.
- "Stereo pot" = one knob, two ganged sections driving L and R in
  parallel (ganged behavior, not independent).
- Where the signal "becomes stereo" is noted explicitly. Before that
  point, the channel carries a single mono line; after that point, two
  lines (L and R) always travel together even if the content is
  momentarily mono.

---

## Electrical & implementation conventions

### Power supplies

- **±15V** rails for the audio signal path (all opamps, active
  switches in the audio path).
- **+5V** separate rail for LED indicators and any future digital
  logic (SOLO / ACTIVE / AFL electronic switch controls, firmware).

### Grounds

- **AGND** (audio ground) — reference for all signal-path active and
  passive components.
- **DGND** (digital / indicator ground) — reference for LED indicators
  and future digital logic.
- **Chassis** — mechanical enclosure, reference for EMI filter caps
  and for TRS jack sleeves only.

AGND / DGND / Chassis are kept galvanically separate on a per-PCB
basis. The system-level joining scheme (star point at PSU, ferrite
bead, direct bond, etc.) is a global decision — see `10-open-tbd.md`.

### Passive component packages

- Default: **0603 SMD** for resistors and for capacitors under 1 µF.
- Capacitors ≥ 1 µF are free from the 0603 constraint — typically
  bipolar electrolytic (signal-path coupling) or polarized
  electrolytic (power-rail bulk), SMD or radial as volume requires.

### IC packages

- Default: **SOIC** for all opamps and analog switches.

### Opamp supply bypassing

- Every opamp gets **100 nF** from +15V → AGND and **100 nF** from
  −15V → AGND, placed as close as possible to the supply pins. This
  is a default that is not repeated in individual section
  descriptions.

### Signal-path coupling capacitors

- Default for series DC-block caps in the signal path: **bipolar
  electrolytic** (e.g., Nichicon Muse ES or equivalent). Film is
  acceptable where value and size allow, but not the default.

### Front-panel switches with LED indicators

Channel-strip mechanical switches that have an "active state"
indication use a **DPDT** with one section commuting the audio signal
and the other section commuting +5V to a front-panel LED on DGND
return. LEDs are low-current standard parts.

Color assignments per channel:

| Switch | LED behavior |
|---|---|
| CHANNEL SOURCE | red (A selected) / green (B selected) |
| HPF | orange (IN) |
| INSERT | blue (IN) |
| PFL | red (IN) |

(Additional switches with LEDs encountered later will be added here.)

### EMI filter — standard topology for TRS jacks

Default for all TRS jacks (input, output, send, return, direct out,
etc.): **T-balanced double-pi filter** on tip and ring lines.

Per stage (and there are two stages per jack):

- tip → 470 pF → midpoint M ← 470 pF ← ring
- M → 47 pF → chassis
- series ferrite 330 Ω @ 100 MHz on each line

So two stages cascaded gives, per jack: 4 × 470 pF + 2 × 47 pF +
2 × ferrite. NPO/C0G ceramics for the small caps.

All EMI filter shunt caps reference **chassis**, not AGND.

Override: individual jacks may use a different topology if explicitly
motivated and recorded in the relevant section's Implementation
details.

### TRS jack sleeve termination

The sleeve of every TRS jack on the console is connected **directly
to chassis**, NOT to AGND. This is the Muncy "Pin 1" rule — applied
throughout to prevent shield-induced RFI / hum currents from entering
the audio signal ground.

Mechanical contact between the jack body and the chassis panel
provides one path; PCB copper to a chassis-only pour adds a second
path with low impedance.

### Impedance-balanced output — default topology

Default for impedance-balanced TRS outputs:

- **75 Ω in series on the tip** (build-out / output impedance).
- **Dummy network on the cold**: 75 Ω in series followed by a node
  with **47 µF to AGND in parallel with 47 kΩ to AGND**.

This default is recommended for consistency across the console.
Individual outputs may use different values or topology if explicitly
motivated and recorded in the relevant section's Implementation
details.

**Known limitation:** tip and cold impedances are not perfectly matched
at low frequencies because the cold's 47 µF has nontrivial Xc below
~100 Hz. This degrades downstream CMR exactly where mains hum lives.
See `10-open-tbd.md` for the open issue and possible mitigations.

---

## Notation in section files

- Section numbers (§2.3, §2.10, …) refer to subsections within the
  section file with that prefix. E.g. §2.3 is in `02-mono-channel.md`.
- "Block N" inside an Implementation details subsection refers to a
  chunk of the signal path that was finalized in a single
  implementation conversation. Blocks accumulate as the design
  progresses.
- References to global conventions (power, grounds, EMI filter,
  impedance-balanced output, switch+LED pattern, etc.) point to this
  file and do not repeat the values at the point of use.
