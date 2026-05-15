# 11 — Digital I/O Summary

Running count of every relay, front-panel LED, and digital pushbutton
in the console. **Keep this file in sync with the design**: whenever
a relay, LED, or button is added, removed, or changed in any section
file, update the relevant table here.

Status flags: ✓ = confirmed from finalized blocks or fixed decisions;
~ = estimated from sections still conceptual (numbers firm up as those
sections are implemented).

---

## Relays

All relays are **AGQ210A03** (DPDT 2 form C, 1-coil latching, 3 V coil,
per `00-conventions.md` "Standard signal relay").

| Section | Function | Per unit | Units | Subtotal | Status |
|---|---|---|---|---|---|
| Mono channel | CHANNEL SOURCE (A/B select) | 1 | 24 | 24 | ✓ |
| Mono channel | MUTE (ACTIVE/SOLO) | 1 | 24 | 24 | ✓ |
| Mono channel | AFL | 1 | 24 | 24 | ✓ |
| Mono channel | Channel Output SELECT | 1 | 24 | 24 | ✓ |
| Mono channel | HPF/INSERT BYPASS | 1 | 24 | 24 | ✓ |
| AUX return | ACTIVE | 1 | 4 | 4 | ✓ |
| AUX return | AFL | 1 | 4 | 4 | ✓ |
| Group | ACTIVE | 1 | 3 | 3 | ✓ |
| Group | AFL | 1 | 3 | 3 | ✓ |
| Monitor | Source-select (§8.1) | 1 | 1 | 1 | ✓ |
| Monitor | AFL/PFL alternation (§8.2) | 1 | 1 | 1 | ✓ |
| **Total** | | | | **136** | |

---

## Front-panel LEDs (indicators only — meter bridge LEDs not counted here)

### Per mono channel (× 24)

> ACTIVE/MUTE (orange), SOLO (red), REC ARM (red) LEDs are **integrated
> in their pushbutton bodies** (located on the fader PCB). They are MCU
> GPIO driven but are **not listed here** as separate indicators —
> they ship with the button.

| Switch / button | Color | Count | Drive |
|---|---|---|---|
| CHANNEL SOURCE | red (A) / green (B) | 2 | MCU GPIO |
| HPF/INSERT BYPASS | orange × 3 (FOLLOW PATH / A / B) | 3 | MCU GPIO |
| HPF | orange | 1 | latching button sec. 2 |
| INSERT | blue | 1 | latching button sec. 2 |
| PFL | red | 1 | latching button sec. 2 |
| Output PRE-POST | TBD × 2 (one per position) | 2 | latching button sec. 2 |
| **Per channel total** | | **10** | |

**Mono channels subtotal: 10 × 24 = 240** ✓

### Other sections (estimates — update as sections are finalized)

| Section | Estimated LEDs | Status |
|---|---|---|
| AUX return × 4 | ~5–6 each → ~20–24 | ~ |
| Group × 3 | ~10–12 each → ~30–36 | ~ |
| AUX master × 4 | ~2–3 each → ~8–12 | ~ |
| CUE master | ~2 | ~ |
| Main Mix master | ~3–5 | ~ |
| Monitor section | ~3–5 | ~ |

**Estimated non-channel subtotal: ~66–84**

**Console total indicator LEDs: ~378–396** (round estimate: ~386)

> Meter bridge LEDs (signal-level meters) are driven by dedicated
> LED driver ICs on the meter PCBs — counted separately when the
> meter bridge is designed.

---

## Front-panel pushbuttons

### Per mono channel (× 24) — 10 per channel

**Momentary with integrated LED — on fader PCB (3 per channel):**

| Button | LED color | Relay / action | LED drive | Status |
|---|---|---|---|---|
| ACTIVE/MUTE | orange (in button) | MUTE relay | MCU GPIO | ✓ |
| SOLO | red (in button) | AFL relay + others' MUTE (SIP opt.) | MCU GPIO | ✓ |
| REC ARM | red (in button) | none — MIDI to DAW | MCU GPIO | ✓ |

Connection from fader PCB to digital logic PCB: TBD.

**Momentary without integrated LED (2 per channel):**

| Button | Separate LEDs | Relay / action | LED drive | Status |
|---|---|---|---|---|
| CHANNEL SOURCE | red (A) + green (B), MCU GPIO | CHANNEL SOURCE relay | MCU GPIO | ✓ |
| HPF/INSERT BYPASS | 3× orange, MCU GPIO | HPF/INSERT BYPASS relay (3 modes) | MCU GPIO | ✓ |

**Latching DPDT (5 per channel):**

| Button | LED sec. 2 | Signal / relay | MCU GPIO in? | Status |
|---|---|---|---|---|
| HPF | orange | audio path direct | no | ✓ |
| INSERT | blue | audio path direct | no | ✓ |
| PFL | red | audio path direct | no | ✓ |
| Output PRE-POST | TBD × 2 | audio path direct | no | ✓ |
| DIRECT/PROCESSED output | TBD | sec. 1 → MCU GPIO in; MCU fires Channel Output SELECT relay | yes | ~ |

**Mono channel subtotal: 10 × 24 = 240**

### Other sections (estimates)

| Section | Estimated buttons | Status |
|---|---|---|
| AUX return × 4 | ~2 each (ACTIVE + AFL) → ~8 | ~ |
| Group × 3 | ~2 each (ACTIVE + AFL) → ~6 | ~ |
| AUX master × 4 | 1 each (PFL) → 4 | ~ |
| Monitor / master | ~2–4 (mono/check L+R + TBD) | ~ |

**Console total pushbuttons: ~258–262**

---

## Per-channel MCU I/O summary (confirmed)

| Signal type | Per channel | × 24 channels | Note |
|---|---|---|---|
| GPIO input — buttons | 6 | 144 | CHANNEL SOURCE, ACTIVE/MUTE, SOLO, REC ARM, HPF/INSERT BYPASS, DIRECT/PROCESSED output (latching button sec. 1) |
| GPIO output — LEDs | 8 | 192 | ACTIVE/MUTE, SOLO, REC ARM (integrated in button on fader PCB), HPF/INSERT BYPASS × 3 (separate), CHANNEL SOURCE A-LED + B-LED |
| Relay coil control lines | 10 (5 relays × 2) | 240 | via dedicated driver ICs, not direct GPIO |

CHANNEL SOURCE LEDs (red/green): MCU GPIO driven — updated on each CHANNEL SOURCE toggle.
ACTIVE/MUTE, SOLO, REC ARM LEDs: MCU GPIO driven — integrated in button body (fader PCB); GPIO wire routes fader PCB → digital logic PCB (TBD).
HPF/INSERT BYPASS mode LEDs (3× orange): MCU GPIO driven — separate from the button.
HPF, INSERT, PFL, PRE-POST, DIRECT/PROCESSED LEDs: latching button sec. 2 driven — no MCU GPIO output needed.
