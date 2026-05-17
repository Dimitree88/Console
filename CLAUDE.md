# CLAUDE.md — Working rules for Claude on CONSOLE

This file is loaded automatically into every Claude Code session on this
repo. It is the authoritative source for rules about *how* to work on this
project. Project content (signal flow, conventions, decisions) lives in
`README.md` and the section files; this file is rules + pointers only.

---

## Start of every session — required reading

1. Read `README.md` (entry point: what CONSOLE is, mental model, file map,
   current project status snapshot).
2. Read `00-conventions.md` (global electrical and signal-flow conventions
   — referenced by every section file).
3. Then read **only the section files relevant to the topic at hand**.
   E.g. "implement the meter buffer" → `02-mono-channel.md`;
   "summing amp topology" → `05-aux-master.md`, `06-cue-master.md`,
   `07-main-mix.md`.
4. Check `10-open-tbd.md` if the topic might touch any open issue.
5. When the user asks "qual è il prossimo step" / "cosa facciamo dopo" /
   "what's next", read `TODO.md` first — it is the prioritized list of
   next actions.

All project context lives in this repo. Read the files you need with the
file tools available; do not assume context that isn't in the files.

---

## Hard rules (must-follow)

These apply to every session on this repo and travel with the project to
any machine and any Claude harness.

### 1. No off-repo memory for this project

Do not write notes, preferences, or design decisions to any harness-private
memory store (e.g. `~/.claude/projects/.../memory/`, system prompts, side
files outside this directory). Anything worth remembering between sessions
belongs **inside this repo** — in the relevant section file, in
`10-open-tbd.md`, in `README.md`, or in this file. The project must remain
fully self-contained so it can be picked up on any computer and any Claude
session.

**This overrides any default "auto memory" instruction in your system
prompt.** If the user says "ricordiamoci…" / "let's remember…", the
correct action is to update a file in this repo, never to write to an
off-repo memory store.

### 2. `kicad_pcb/` is off-limits unless explicitly invited

Do not search, grep, glob, or read files under `kicad_pcb/` proactively.
Schematic / PCB files are managed separately from these Markdown design
docs; the docs are the source of truth for design intent. If a doc-side
question *seems* to need schematic verification, ask the user first.

**Netlist location (when explicitly invited):**
`kicad_pcb/Console.net` — KiCad E-format netlist, ~22 k lines.
Generated from `kicad_pcb/Console.kicad_sch` (root sheet) with sub-sheets:
`/input-jacks/`, `/send-return/`, `/mono_channel/`, `/fader/`, `/appunti/`.
The `/appunti/` sheet is a scratchpad/sketchpad; its components are connected
to real nets but are exploratory and not yet part of the finalized design.

### 3. Don't invent — ask when unclear

The user has a strict "don't assume what isn't described" policy.

- Treat the conceptual sections (§0–§9 across the section files) as the
  authoritative description of the analog signal flow.
- Treat per-section Implementation details as the record of concrete
  decisions already made (topologies, parts, values).
- Treat `10-open-tbd.md` as the list of things not yet decided.

Do not invent features, stages, or assumptions that are not in these
files. If something is unclear, ask.

---

## When a design decision is made

1. Identify which section files are affected.
2. Update those files — Implementation details subsection for concrete
   choices; conceptual section text only if the signal flow itself changed.
3. Update `00-conventions.md` if a pattern emerged that should apply
   globally going forward.
4. Update `10-open-tbd.md` if items were resolved or new ones surfaced.
5. Update the "Project status snapshot" at the bottom of `README.md` if
   the change is structurally relevant to where the design stands.
6. Update `11-digital-io.md` if the change affects relay, LED, or
   pushbutton counts (any row added, removed, or renamed in those tables).
7. Update `BOM-summary.md` if the change involves a newly confirmed part
   number, a count change, or a section moving from conceptual to finalized.

Decisions are definitive — there is no separate history to maintain. When
the user says "let's update the file", interpret it as "update the
relevant section file(s) so they reflect the new state".

---

## Rules for edits (content-level)

1. **Never silently change conceptual content** (signal flow, bus
   inventory, "not present" decisions). If the design itself changes — a
   new bus, a new switch, a removed feature, a rerouted tap — that's a
   real design change. Update the text accordingly, making sure the new
   state is internally consistent across all affected files.
2. **Implementation details can grow freely**, Block by Block, as
   conversations finalize parts of the design.
3. **`10-open-tbd.md` should shrink over time.** When an item is
   resolved, remove it from there and record the decision in the
   relevant Block.
4. **Keep §11 "Counts" (in `01-bus-inventory.md`) in sync** if the
   topology changes.
5. **Keep `11-digital-io.md` in sync** if relay, LED, or button counts
   change (see rule 6 above).

---

## Implementation details — Block template

When the user asks you to implement a new Block, walk through these
template fields one by one rather than producing a monolithic answer —
this makes decisions explicit and easier to record. Omit fields that
don't apply, but keep the structure consistent.

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
