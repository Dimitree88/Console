# CONSOLE — Reference Specification

> Analog mixing console design — full signal-flow reference and
> implementation record. This is a multi-file documentation project.
> If you (Claude) are reading this in a new conversation, treat this
> README as the entry point.

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
with a full-differential Direct Out from Input A for ADC tracking;
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
| `README.md` | this file — entry point, file map, working rules |
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

(Sections §11 "Counts" lives at the end of `01-bus-inventory.md`.
The one-paragraph mental model lives at the top of this README.)

---

## How to use this documentation

### When starting a new conversation about CONSOLE

1. Always read `README.md` and `00-conventions.md` first. These two
   together give the global context.
2. Then read **only the section files relevant to the topic at hand**.
   E.g. for "implement the meter buffer" you need `02-mono-channel.md`;
   for "discuss summing amp topology" you need `05-aux-master.md` and
   probably `06-cue-master.md` and `07-main-mix.md`.
3. Look at `10-open-tbd.md` if the topic might touch any open issue.

All project context lives in this repo. Read the files you need with
the file tools available to you; do not assume context that isn't
in the files.

### When ending a conversation

If any design decision was made:

1. Identify which section files are affected.
2. Update those files (Implementation details subsection for concrete
   choices; conceptual section text only if the signal flow itself
   changed).
3. Update `00-conventions.md` if a pattern emerged that should apply
   globally going forward.
4. Update `10-open-tbd.md` if items were resolved or new ones surfaced.

The user will replace the modified files in Project Knowledge. Files
not touched do not need to be re-uploaded.

---

## Implementation details — format for each section file

Every main section file (`02-mono-channel.md` through `08-monitor.md`)
contains both the **conceptual signal flow** description and an
**Implementation details** subsection. The Implementation details
subsection grows over time as concrete circuit choices are made,
organized into "Blocks" (e.g. Block 1, Block 2, …) corresponding to
chunks of the signal path that were finalized in a single conversation.

Each Block uses this template (omit fields that don't apply, but keep
the structure consistent):

```
#### Block N — [name] (FINALIZED | IN-PROGRESS)

Status: [conceptual | in-progress | finalized]

Topology chosen:
- brief description per stage in the block
- schematic reference (Self fig X.Y, Jung p.NN, etc.) if any

Active devices:
- Opamps: part numbers + rationale
- Analog switches: part numbers (if applicable)
- Other ICs (VCA, matched pairs, etc.)

Key passive values:
- values that affect transfer function, noise, CMRR, or bandwidth
- tolerances where they matter (CMR-critical resistors, etc.)

Design targets / expected performance:
- gain, Zin, Zout, CMRR, THD+N, noise, bandwidth, headroom

Decisions and tradeoffs:
- why this topology over alternatives
- compromises accepted

Open issues for this Block:
- still-undecided items specific to this Block
```

---

## Rules for edits

1. **Never silently change conceptual content** (signal flow,
   bus inventory, "not present" decisions). If the design itself
   changes — a new bus, a new switch, a removed feature, a rerouted
   tap — that's a real design change. Update the text accordingly,
   making sure the new state is internally consistent across all
   affected files.
2. **Implementation details can grow freely**, Block by Block, as
   conversations finalize parts of the design.
3. **`10-open-tbd.md` should shrink over time.** When an item is
   resolved, remove it from there and record the decision in the
   relevant Block.
4. **Keep §11 "Counts" (in `01-bus-inventory.md`) in sync** if the
   topology changes.

---

## Instructions for future Claude

If you are reading this in a new conversation:

- Treat the conceptual sections (§0 – §9 across the section files) as
  the authoritative current description of the **analog signal flow**.
- Treat `10-open-tbd.md` as the list of things not yet decided.
- Treat per-section Implementation details as the record of concrete
  decisions already made (topologies, parts, values).
- **Do not invent features, stages, or assumptions that are not in
  these files.** The user has a strict "don't assume what isn't
  described" policy. If something is unclear, ask.
- If the user asks you to implement a new Block, walk through the
  Implementation details template fields one by one rather than
  producing a monolithic answer — this makes decisions explicit and
  easier to record.
- When the user says "let's update the file", interpret it as
  "update the relevant section file(s) so they reflect the new
  state". Decisions are definitive — there is no separate history
  to maintain.

### Project rules for Claude (must-follow)

These rules apply to every Claude session on this repo. They live
here, in the repo, on purpose — so they travel with the project to
any machine and any Claude harness.

1. **No hidden / off-repo memory for this project.** Do not write
   notes, preferences, or design decisions to any
   harness-private memory store
   (e.g. `~/.claude/projects/.../memory/`, system prompts,
   side files outside this directory). Anything worth remembering
   between sessions belongs **inside this repo** — in the relevant
   section file, in `10-open-tbd.md`, or in this README. The
   project must remain fully self-contained so it can be picked up
   on any computer and any Claude session.
2. **`kicad_pcb/` is off-limits unless the user explicitly says
   otherwise.** Do not search, grep, glob, or read files under
   `kicad_pcb/` proactively. Schematic / PCB files are managed
   separately from these Markdown design docs; the docs are the
   source of truth for design intent. If a doc-side question
   *seems* to need schematic verification, ask the user first.

---

## Project status snapshot

(Update this section briefly at the end of major conversations, so
future Claude can immediately see where things stand.)

- Mono channel: signal flow conceptually complete. Implementation:
  - Block 1 (input stage → CHANNEL SOURCE switch) FINALIZED —
    CHANNEL SOURCE now relay-driven (AGQ210A03 latching) with
    front-panel momentary pushbutton + firmware-driven bipolar
    pulse, replacing the earlier mechanical DPDT.
  - Block 2 (HPF + Insert Send/Return + jack PCB) FINALIZED.
  - Block 3 (meter buffer + PFL switch + MUTE) FINALIZED — MUTE
    now uses the AGQ210A03 latching signal relay (both form-C
    contacts paralleled), replacing the earlier ADG419.
  - Block 4 (pre-fader node + fader PCB + post-fader amp +10 dB +
    AUX/CUE pre & post sends) FINALIZED.
  - Block 5 (active pan, Self-style) FINALIZED — Self canonical
    values (R_FB = 2.2 kΩ, center dip ≈ 4.5 dB).
  - Block 6 (post-pan 2-gang × 4-pos routing rotary) topology
    defined; bus summing resistor values TBD together with §04 / §07.
  - AFL switch: switching element fixed (AGQ210A03 latching, one
    per channel handling L+R on its two form-C contact sets);
    tap location and AFL summing-resistor values still deferred
    to §08. Post-Fader Output stage: conceptual flow fixed,
    implementation deferred (see `10-open-tbd.md`).
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
- Relay coil bipolar driver IC, partitioning, coil-rail choice
  (3 V LDO vs +3.3 V direct), and master-output hard-mute for
  firmware boot init are tracked as design-phase open issues in
  `10-open-tbd.md`.
- Fader PCB partitioning: settled at **2 channels per fader PCB**
  (12 fader PCBs total). Input PCB partitioning still TBD.
