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
- **+3.3V** separate rail for LED indicators, MCU / digital logic,
  and standard-signal-relay coil drive (SOLO / ACTIVE / AFL
  electronic switches, firmware, front-panel pushbuttons).

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
  100 nF above, every PCB that receives the ±15V (or +3.3V) rails
  carries a bulk capacitor on each rail to its respective ground
  (AGND for ±15V, DGND for +3.3V), placed near the local IC group.
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

### Standard signal relay — AGQ210A03

Default electronic switching element throughout CONSOLE:
**Panasonic AGQ210A03** — DPDT 2 form C, **1-coil latching**
signal relay, AgPd + Au-clad contacts, **3 V coil** (1-coil
latching, pulse-driven), surface-mount terminal A type, sealed
package, Telcordia / FCC Part 68 surge withstand.

Used in every position where the audio signal must be interrupted
or rerouted under digital control:

- ACTIVE / SOLO mute (mono channels, AUX returns, groups);
- AFL enable (mono channels, AUX returns, groups);
- CHANNEL SOURCE A/B select on mono channels (replaces the
  mechanical DPDT — see §2.2);
- master monitor source-select (Main vs Solo summer, §8.1) and
  Solo summer AFL/PFL alternation (§8.2).

Why a relay rather than a CMOS analog switch:

- True dry-circuit signal path: contact resistance ≤ 100 mΩ
  initial (vs ≈ 17 Ω R_on for an ADG419-class CMOS), no charge
  injection, no CMOS-specific THD vs source-Z artefacts.
- Off-isolation: open contacts are physically separated and
  plastic-sealed; effectively unbounded at audio.
- Min switching load 10 µA / 10 mV DC — well below any signal
  level encountered in the path.
- DPDT 2 form C lets one relay handle a stereo pair (L on one
  contact set, R on the other) wherever the switched signal is
  stereo, halving the relay count.
- Plastic-sealed body: no flux/dust ingress during PCB assembly.

Why latching (over a non-latching standard signal relay):

- **Coil current flows only during set/reset pulses** (~10 ms);
  average dissipation on the coil rail is essentially zero. The
  worst-case continuous coil-rail budget (~10 W across 88 relays)
  that a non-latching plan would impose is replaced by a small
  peak transient handled by local bulk capacitance.
- **State held mechanically** — the audio path is immune to
  glitches on the coil supply, brown-outs, or MCU resets.

Topology and drive:

- Mono switching position (one signal in, one signal out): both
  form-C contact sets are wired in parallel — COM1 ‖ COM2,
  NO1 ‖ NO2 — to halve the contact resistance and provide
  contact redundancy. NC contacts are wired according to the
  reset-state direction needed (signal-to-AGND for shunt-mute,
  or to a default source for A/B selection — see per-block
  details).
- Stereo switching position (one stereo pair in, one stereo pair
  out): contact set 1 carries L, contact set 2 carries R; they
  are mechanically ganged inside the relay, so a single set
  (or reset) pulse drives both sides perfectly synchronously.
- **Coil drive: bipolar pulse**, ≥ 10 ms duration per Panasonic
  guideline (operate / release time itself ≤ 4 ms; the longer
  pulse covers temperature and operating-condition variation).
  A pulse of one polarity across the polarized coil **sets** the
  relay (NO contacts make); the opposite polarity **resets** it
  (NC contacts make). State persists with the coil unpowered.
  Two control lines per relay at the firmware level (SET, RESET).
- **Coil supply rail: +3.3 V** (the project's logic rail). The
  3 V coil tolerates +3.3 V — ≈ +10 % over rated, well within the
  150 % max allowable specified by the datasheet — and the
  pulse-only duty cycle makes the small overdrive thermally
  irrelevant. No dedicated rail or series dropping resistor is
  needed; the bipolar driver feeds the coil directly from the
  +3.3 V plane.
- **Coil polarity matters** (the coil is polarized — see the
  Panasonic schematic, pins 1 = + and 8 = − for "set"). Layout
  must respect the + / − pin assignment.
- **Coil driver**: bipolar / half-H-bridge per relay. Candidates
  include Toshiba TBD62783A (high-side) + TBD62083A (low-side)
  pair, dedicated octal latching-relay drivers
  (e.g. MAX4820 / MAX4821), or a per-relay discrete NPN+PNP
  pair. Driver IC choice and flyback / freewheeling protection
  are deferred (see `10-open-tbd.md`).
- **PCB partitioning — stackable two-board pattern.** Each relay
  sits on the audio PCB; its coil is exposed via a **3-pin
  connector** (two pins for the coil terminals, **center pin for
  DGND**). That connector mates with a **lower stackable PCB**
  carrying all the digital / logic components that drive the
  relay (bipolar coil driver, MCU lines, local +3.3 V bulk on
  DGND, etc.). The two PCBs are coupled via standard
  Arduino-style pin headers — the audio PCB holds zero digital
  logic. **On the audio PCB, DGND is not connected to anything**:
  it is a local copper island that exists only as a return path
  for the relay-coil current pulses, joining the system DGND
  exclusively through the 3-pin connector / lower PCB. Coil
  switching transients therefore stay confined to the lower
  (digital) board and never share copper with the audio path.
- **State at power-up is indeterminate.** A latching relay holds
  whatever state it was last commanded into; transport or
  installation impact may also change the reset position
  (Panasonic explicit guideline). At every cold boot, firmware
  must initialize every relay by issuing explicit set / reset
  pulses to drive each one to a known state. The desired boot
  state of each relay is the same "safe" state that a
  non-latching design would have produced via NC contacts:
  channel muted, AFL off, monitor on Main Mix, CHANNEL SOURCE
  on Input A — see per-block details. Boot-init is a firmware
  contract, not a hardware-guaranteed property.
- **Master-output hard-mute during firmware init**: until
  firmware finishes initializing all relays to their reset
  state, audio output to the room must be held silent by a
  separate master-stage hardware mute. Topology of that
  master mute is deferred to the master-section design (see
  `10-open-tbd.md`).

Power-budget note: full count is approximately 112 relays
across the console (24 mono channels × 4 — CHANNEL SOURCE, MUTE,
AFL, DIRECT OUT SELECT — + 4 AUX returns × 2 + 3 groups × 2 +
2 in master monitor). Pulse energy per state
change ≈ 1.2 mJ (3.3 V × 37 mA × 10 ms, with the 3 V coil
tolerated at +3.3 V per "Coil supply rail" above). Average
coil-rail dissipation in normal operation is essentially zero.
Peak transient if every relay in the console were toggled
simultaneously ≈ 4.1 A for 10 ms (112 × 37 mA) — handled by
local bulk on the +3.3 V rail per PCB, with firmware staging
changes (e.g. N relays per stage) when this matters.

### Front-panel switches with LED indicators

Two patterns coexist on the channel strip and on master sections:

**Pattern A — direct mechanical DPDT.** A front-panel toggle / push
switch has two sections: one commutes the audio signal, the other
commutes +3.3 V to a front-panel LED on DGND return. Used for HPF,
INSERT, PFL, and any mechanical switch whose state is set directly
by the operator and held mechanically.

**Pattern B — momentary pushbutton + MCU + bipolar pulse driver +
standard latching relay (AGQ210A03).** A single-contact front-panel
pushbutton (momentary) is read by firmware; on each press, firmware
issues a set or reset pulse (≥ 10 ms) to the relay coil's bipolar
driver, toggling the relay's mechanically latched state. The
relay's two form-C contact sets commute respectively the audio
signal and the +3.3 V LED indicator current — the LED tracks the
latched contact position regardless of coil energization. Used
for CHANNEL SOURCE A/B (see §2.2).

**Pattern C — momentary pushbutton with integrated LED + MCU GPIO.**
A front-panel pushbutton with an integrated LED. The button press is
read by firmware as a GPIO input; the integrated LED is driven
directly by a firmware GPIO output — not by a relay contact set.
May additionally command a relay coil as a secondary action:
ACTIVE/MUTE (orange) commands the MUTE relay; SOLO (red) commands
the AFL relay of this channel and optionally the MUTE relays of other
channels (SIP mode); REC ARM (red) does not command any audio relay —
it generates a MIDI or equivalent signal toward the DAW. See §2.11.

LEDs are low-current standard parts in all patterns.

Color assignments per channel:

| Switch | LED color | Count | LED drive | Pattern |
|---|---|---|---|---|
| CHANNEL SOURCE | red (A) / green (B) | 2 | relay contact set 2 | B |
| ACTIVE/MUTE | orange (active) | 1 | MCU GPIO | C |
| SOLO | red (solo active) | 1 | MCU GPIO | C |
| REC ARM | red (armed) | 1 | MCU GPIO | C |
| HPF | orange (IN) | 1 | mech switch sec.2 | A |
| INSERT | blue (IN) | 1 | mech switch sec.2 | A |
| PFL | red (IN) | 1 | mech switch sec.2 | A |
| Output PRE-POST | TBD (one per position) | 2 | mech switch sec.2 | A |

Per-channel LED total: **10** (2 relay-contact + 3 MCU GPIO + 5 mech-switch).
Per-channel MCU GPIO LED outputs: **3** (ACTIVE/MUTE, SOLO, REC ARM).
Per-channel MCU GPIO button inputs: **4** (CHANNEL SOURCE, ACTIVE/MUTE, SOLO, REC ARM).

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
