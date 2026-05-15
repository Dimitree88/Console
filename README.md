# CONSOLE — Reference Specification

> Analog mixing console design — full signal-flow reference and
> implementation record. This is a multi-file documentation project;
> this README is the entry point.
>
> **For Claude sessions:** working rules and session-start procedure
> live in `CLAUDE.md`, which is loaded automatically.

---

## What this project is

CONSOLE is a fully analog mixing console. 24 mono input channels,
**4 stereo AUX returns**, 3 stereo groups, 4 AUX masters, CUE master,
Main Mix master with parallel SSL-style bus compression, monitor
section with AFL/PFL solo logic.

Firmware is out of scope; this documents the analog signal flow and
the implementation decisions made during circuit design.

---

## Mental model (one paragraph)

CONSOLE is a 24+4+3 analog desk (24 mono channels, 4 stereo AUX
returns, 3 stereo sub-groups) feeding a Main stereo bus with
parallel SSL-style compression. Channel strips follow a conventional
flow (balanced in → gain → HPF → insert → meter and PFL taps →
MUTE electronic switch → fader → +10 dB post-amp → pan → routing)
with two distinctive design choices: (1) dual balanced inputs A/B
with a full-differential Channel Output from Input A for ADC tracking;
(2) every channel has 4 AUX sends implemented as dual-gang pots that
simultaneously drive a pre-fader and a post-fader mono bus, giving
8 AUX buses total. The 4 stereo AUX returns are routing-only
(no Post-Fader Output to the outside, no Insert, no AUX send) — they
contribute only to internal buses (Main / Groups / AFL / CUE / PFL
stereo). Groups offer per-AUX split-or-mono mode switches so a
stereo group can feed stereo AUX pairs either as L/R split or as
mono premix. The master section's distinguishing features are the
parallel compression on the main bus (dry fader + SSL-style
compressor + wet fader → summed output) and an "in front balance"
pot that mixes an attenuated main mix behind the AFL so the engineer
always hears the context. Digital control is limited to hard-muting /
enabling electronic switches for ACTIVE, SOLO, and AFL — no VCAs,
no recall, no automation. Talkback, tone oscillators, and master
master meters are deliberately absent.

---

## File map

| File | Contents |
|------|----------|
| `CLAUDE.md` | working rules for Claude — required reading at session start, loaded automatically |
| `README.md` | this file — entry point, file map, project status snapshot |
| `00-conventions.md` | design conventions + electrical/implementation conventions (power supplies, grounds, packages, default audio opamp, default topologies for EMI filter, impedance-balanced output, switches+LEDs, …) |
| `01-bus-inventory.md` | enumeration of every bus in the console (stereo + mono) and what feeds each one |
| `02-mono-channel.md` | §2 — mono input channel signal flow + Implementation details (Blocks 1, 2, …) |
| `03-aux-return.md` | §3 — stereo AUX returns (×4, identical) |
| `04-group.md` | §4 — stereo groups (×3) |
| `05-aux-master.md` | §5 — AUX master (×4) |
| `06-cue-master.md` | §6 — CUE master |
| `07-main-mix.md` | §7 — Main Mix master + parallel compression |
| `08-monitor.md` | §8 — Master monitor section |
| `09-not-present.md` | §9 — features deliberately NOT in CONSOLE |
| `10-open-tbd.md` | §10 — items deliberately unresolved at this stage |
| `11-digital-io.md` | §11 — running count of relays, LEDs, and pushbuttons; keep in sync with design changes |

(Sections §11 "Counts" lives at the end of `01-bus-inventory.md`.
The one-paragraph mental model lives at the top of this README.)

---

## Section file structure

Every main section file (`02-mono-channel.md` through `08-monitor.md`)
contains both the **conceptual signal flow** description and an
**Implementation details** subsection. Implementation details grow
over time as concrete circuit choices are made, organized into
"Blocks" (Block 1, Block 2, …) corresponding to chunks of the
signal path that were finalized in a single conversation. The Block
template and editing rules live in `CLAUDE.md`.

---

## Project status snapshot

(Update this section briefly at the end of major conversations, so
future Claude can immediately see where things stand.)

- Mono channel: signal flow conceptually complete. Implementation:
  - Block 1 (input stage → CHANNEL SOURCE switch + DIRECT OUT
    SELECT relay on input PCB) FINALIZED — CHANNEL SOURCE
    relay-driven (AGQ210A03 latching) with momentary pushbutton +
    firmware bipolar pulse; primary motivation: enables global
    master A/B flip across all 24 channels. Channel Output SELECT
    relay added: makes the Channel Output TRS jack switchable between
    Input A full-differential and channel PRE/POST impedance-
    balanced signal (see Block 8).
  - Block 2 (HPF + Insert Send/Return + jack PCB) FINALIZED.
  - Block 3 (meter buffer concept + PFL switch + MUTE) FINALIZED —
    MUTE relay (AGQ210A03) now configured **series + shunt** (contact
    set 1 = series switch, contact set 2 = shunt PRE-FADER to AGND
    when muted), replacing the earlier paralleled-contacts approach.
    Meter buffer implementation (NE5532 + DG419) finalized in Block 10.
  - Block 4 (pre-fader node + fader PCB + post-fader amp +10 dB +
    AUX/CUE pre & post sends) FINALIZED.
  - Block 5 (active pan, Self-style) FINALIZED — Self canonical
    values (R_FB = 2.2 kΩ, center dip ≈ 4.5 dB).
  - Block 6 (post-pan 2-gang × 4-pos routing rotary) topology
    defined; bus summing resistor values TBD together with §04 / §07.
  - Block 7 (AFL switch) FINALIZED — tap at post-pan opamp output;
    NC to AGND (constant termination, click-free); NO to AFL bus.
    Summing resistor values deferred to §08.
  - Block 8 (Output PRE-POST switch + switchable Channel Output)
    IN-PROGRESS — 75R taps from pre-MUTE and POST-FADER nodes;
    mechanical PRE-POST selector; 2-pin connector to input PCB;
    Channel Output SELECT relay on input PCB. Front-panel control
    resolved: latching DPDT pushbutton ("DIRECT/PROCESSED output"),
    sec. 1 → MCU GPIO, sec. 2 → indicator LED. Cold dummy network and
    LED colors TBD (see `10-open-tbd.md`).
  - Block 10 (dual meter buffers) FINALIZED — CHANNEL SOURCE relay
    contact set 2 (wired complementary to set 1) delivers the
    inactive receiver directly to the meter buffer, no additional
    electronic switch; TL072 dual JFET opamp buffers active
    (pre-MUTE) and inactive meter signals; 3-pin connector to meter
    bridge PCB. CHANNEL SOURCE LEDs now MCU GPIO driven (+2 GPIO
    output per channel vs. relay-contact).
- AUX returns: 4 channels, all identical, conceptual stage. ACTIVE
  and AFL switching elements fixed as AGQ210A03 (1 relay
  per function, handling L+R on its two contact sets).
- Groups: same — ACTIVE and AFL fixed as AGQ210A03.
- Master monitor: source-select (§8.1) and Solo summer AFL/PFL
  alternation (§8.2) fixed as AGQ210A03. Other stages still
  conceptual.
- All other sections (AUX/CUE/Main masters): still conceptual.
- Global conventions: established (power, grounds, packages, default
  audio opamp **NE5532**, **standard signal relay
  AGQ210A03** (Panasonic, 1-coil latching, 3 V coil), EMI filter,
  sleeve termination, impedance-balanced output topology,
  front-panel switches with LEDs, per-PCB local bulk decoupling on
  power rails). Selective upgrade to OPA1642 / OPA1656 in specific
  positions is identified as a budget-review item — see
  `10-open-tbd.md`.
- Relay infrastructure: coil supply rail settled (+3.3 V direct);
  driver partitioning settled (per-PCB stackable two-board pattern
  per `00-conventions.md` — audio PCB carries only the relay coil
  via a 3-pin connector, lower stackable PCB carries all logic).
  Total relay count: **112** (24 channels × 4 + 4 AUX returns × 2 +
  3 groups × 2 + 2 master monitor). Coil bipolar driver IC and
  master-output hard-mute for firmware boot init still tracked as
  design-phase open issues in `10-open-tbd.md`.
- Fader PCB partitioning: settled at **2 channels per fader PCB**
  (12 fader PCBs total). Input PCB partitioning still TBD.
- Front-panel channel buttons (§2.11): defined — 4 pushbuttons per
  channel: CHANNEL SOURCE (Pattern B, relay-contact LED), plus
  ACTIVE/MUTE (orange), SOLO (red), REC ARM (red) all Pattern C
  (integrated LED, MCU GPIO in + out). SOLO drives AFL relay + SIP
  option; REC ARM MIDI-only, no audio relay. Per-channel digital
  I/O: 4 GPIO inputs (buttons) + 3 GPIO outputs (LEDs) +
  relay coil drivers via dedicated ICs.
