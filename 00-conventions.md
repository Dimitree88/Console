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
- "Electronic switch" = signal-path switching element controlled by
  digital logic. In CONSOLE the standard implementation is a
  **signal relay** (see "Standard signal relay" below), not a CMOS /
  JFET analog switch IC. Used for SOLO, ACTIVE, AFL enable, CHANNEL
  SOURCE A/B select, master monitor source-select — anywhere the
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

### Default audio opamp

- Default audio-stage opamp throughout CONSOLE: **NE5532** (dual SOIC).
  Used in active stages of the channel strip, group strip, AUX returns,
  master sections, and monitor section unless a specific Block
  explicitly motivates a different choice.
- Rationale: the NE5532 is the historical workhorse of professional
  mixing consoles (SSL, Neve, Soundcraft, Trident lineage) and remains
  the best price/performance compromise for noise (5 nV/√Hz),
  drive capability (rated for 600 Ω loads at low distortion), distortion
  characteristics (no crossover), and cost. Its bipolar input bias
  current (~200 nA) is handled in each Block by sizing bias-return
  resistors and DC-block capacitors appropriately.
- Documented exceptions in the current design:
  - **OPA1679** (quad SOIC, CMOS input) in Block 1 of the mono channel
    (input buffers + All-Inverting balanced receivers). Chosen for low
    bias current at the high-impedance input nodes and for the 4-opamp-
    in-one-package density required by buffer + receiver count.
  - **OPA1632** for the differential output stage of any future
    fully-differential ADC drive (not currently in CONSOLE; reserved
    for ADAT project).
- **Budget-review item**: at the prototype/measurement stage, evaluate
  whether selected positions warrant an upgrade from NE5532 to
  **OPA1642** (JFET, lower CM-distortion, negligible bias current) or
  **OPA1656** (CMOS, significantly lower noise at 1.6 nV/√Hz). Strongest
  candidates are positions where input-stage impedance is high and
  varies (Block 5 active pan: wiper Z varies with pan position) or
  where common-mode swing is largest (Block 2 HPF Sallen-Key, Block 5
  active pan +input). See `10-open-tbd.md`.

### Opamp supply bypassing

- Every opamp IC gets **100 nF** from +15V → AGND and **100 nF** from
  −15V → AGND, placed as close as possible to the supply pins. This
  is a default that is not repeated in individual section
  descriptions.
- **Local bulk on each PCB power rail:** in addition to the per-IC
  100 nF above, every PCB that receives the ±15V (or +5V) rails
  carries a bulk capacitor on each rail to its respective ground
  (AGND for ±15V, DGND for +5V), placed near the local IC group.
  Typical value **4.7–10 µF**; exact value and dielectric (tantalum,
  polymer, ceramic X7R, low-ESR electrolytic) chosen per PCB at
  layout time, based on density of active devices and signal-path
  sensitivity. The per-IC 100 nF handles HF decoupling; the local
  bulk improves transient response and low-frequency PSRR. This
  default is not repeated in individual section descriptions.

### Signal-path coupling capacitors

- Default for series DC-block caps in the signal path: **bipolar
  electrolytic** (e.g., Nichicon Muse ES or equivalent). Film is
  acceptable where value and size allow, but not the default.

### Standard signal relay — FTR-B3GA4.5Z-B10

Default electronic switching element throughout CONSOLE:
**Fujitsu FTR-B3GA4.5Z-B10** — DPDT 2 form C signal relay, gold
overlay silver-nickel bifurcated contacts, 4.5 V standard
(non-latching) coil, surface mount, B10 tape & reel.

Used in every position where the audio signal must be interrupted
or rerouted under digital control:

- ACTIVE / SOLO mute (mono channels, AUX returns, groups);
- AFL enable (mono channels, AUX returns, groups);
- CHANNEL SOURCE A/B select on mono channels (replaces the
  mechanical DPDT — see §2.2);
- master monitor source-select (Main vs Solo summer, §8.1) and
  Solo summer AFL/PFL alternation (§8.2).

Why a relay rather than a CMOS analog switch:

- True dry-circuit signal path: contact resistance ≤ 75 mΩ (vs
  ≈ 17 Ω R_on for an ADG419-class CMOS), no charge injection,
  no CMOS-specific THD vs source-Z artefacts.
- Off-isolation: open contacts are physically separated and
  plastic-sealed; isolation > 80 dB at 1 MHz, effectively
  unbounded at audio.
- Min switching load 10 mV / 0.01 mA — well below any signal
  level encountered in the path.
- DPDT 2 form C lets one relay handle a stereo pair (L on one
  contact set, R on the other) wherever the switched signal is
  stereo, halving the relay count.
- Plastic-sealed body: no flux/dust ingress during PCB assembly.

Topology and drive:

- Mono switching position (one signal in, one signal out): both
  form-C contact sets are wired in parallel — COM1 ‖ COM2,
  NO1 ‖ NO2 — to halve the contact resistance and provide
  contact redundancy. NC contacts are wired according to the
  fail-safe direction needed (signal-to-AGND for shunt-mute, or
  to a default source for A/B selection — see per-block details).
- Stereo switching position (one stereo pair in, one stereo pair
  out): contact set 1 carries L, contact set 2 carries R; they
  are mechanically ganged inside the relay, so a single coil
  drives both sides perfectly synchronously.
- **Coil drive: from the +5 V rail.** The 4.5 V coil tolerates
  +5 V (≈ +10 % over nominal, well within the operating-voltage
  range shown on the datasheet). Coil current ≈ 34 mA at +5 V;
  rated power 140 mW (datasheet) is exceeded by ~25 %, accepted
  given the temperature-rise margin. A series resistor (~ 22 Ω)
  to drop ~ 0.7 V back to nominal 4.5 V is an option at layout
  time if measured coil temperature rise is excessive.
- **Flyback diode** (1N4148-class, exact part TBD) across the
  coil for inductive kickback when the driver releases.
- **Coil driver**: open-collector / open-drain sink capable of
  ~ 40 mA per coil (e.g., ULN2803-class octal Darlington with
  built-in flyback diodes, or discrete BJT/MOSFET per coil).
  Partitioning between centralized drivers and per-PCB drivers,
  and exact driver IC, are deferred (see `10-open-tbd.md`).
- **Polarity**: the coil is polarized (datasheet pins 1 and 8 =
  + and − respectively for energize). Layout must respect this.
- **Fail-safe at power-up**: at rest (coil unpowered) the NC
  contacts are closed and NO are open. The default position of
  each switch is therefore chosen so that an unpowered relay
  produces a "safe" state — channel muted, AFL off, monitor on
  Main Mix, CHANNEL SOURCE on Input A — rather than a
  surprising one.

Power-budget note: full count is approximately 88 relays
across the console (24 mono channels × 3 + 4 AUX returns × 2 +
3 groups × 2 + 2 in master monitor). Worst-case all-energized
power on the +5 V coil rail ≈ 12 W. Average is much lower
(MUTE is normally OFF / coil not energized, AFL is rare,
CHANNEL SOURCE picks one of two so on average ~50 %). PSU
sizing for the +5 V coil rail is part of the global power
design (see `10-open-tbd.md`).

### Front-panel switches with LED indicators

Two patterns coexist on the channel strip and on master sections:

**Pattern A — direct mechanical DPDT.** A front-panel toggle / push
switch has two sections: one commutes the audio signal, the other
commutes +5 V to a front-panel LED on DGND return. Used for HPF,
INSERT, PFL, and any mechanical switch whose state is set directly
by the operator and held mechanically.

**Pattern B — momentary pushbutton + logic latch + standard relay
(FTR-B3GA4.5Z-B10).** A single-contact front-panel pushbutton
(momentary) feeds a logic latch (firmware or simple flip-flop) that
energizes the relay coil. The relay's two form-C contact sets
commute respectively the audio signal and the +5 V LED indicator
current. Used for CHANNEL SOURCE A/B (see §2.2) and, in general,
for any switch whose state is also reflected in the digital control
domain (ACTIVE, SOLO, AFL — though those have no front-panel LED on
the channel strip itself, only in master sections).

LEDs are low-current standard parts in either pattern.

Color assignments per channel:

| Switch | LED behavior | Pattern |
|---|---|---|
| CHANNEL SOURCE | red (A selected) / green (B selected) | B (relay-driven) |
| HPF | orange (IN) | A (mechanical DPDT) |
| INSERT | blue (IN) | A (mechanical DPDT) |
| PFL | red (IN) | A (mechanical DPDT) |

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
  impedance-balanced output, default audio opamp, switch+LED pattern,
  etc.) point to this file and do not repeat the values at the point
  of use.
