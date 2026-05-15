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
| Mono channel | DIRECT OUT SELECT | 1 | 24 | 24 | ✓ |
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

| Switch | Color | Count | Drive | Pattern |
|---|---|---|---|---|
| CHANNEL SOURCE | red (A) / green (B) | 2 | MCU GPIO | — |
| ACTIVE/MUTE | orange | 1 | MCU GPIO | C |
| SOLO | red | 1 | MCU GPIO | C |
| REC ARM | red | 1 | MCU GPIO | C |
| HPF/INSERT BYPASS | orange × 3 (FOLLOW PATH / A / B) | 3 | MCU GPIO | — |
| HPF | orange | 1 | mech switch sec.2 | A |
| INSERT | blue | 1 | mech switch sec.2 | A |
| PFL | red | 1 | mech switch sec.2 | A |
| Output PRE-POST | TBD (one per position) | 2 | mech switch sec.2 | A |
| **Per channel total** | | **13** | | |

**Mono channels subtotal: 13 × 24 = 312** ✓

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

## Front-panel pushbuttons (MCU GPIO inputs)

### Per mono channel (× 24)

| Button | Pattern | Relay commanded | LED drive | Status |
|---|---|---|---|---|
| CHANNEL SOURCE | B | CHANNEL SOURCE relay | relay contact set 2 | ✓ |
| ACTIVE/MUTE | C | MUTE relay | MCU GPIO (orange) | ✓ |
| SOLO | C | AFL relay + others' MUTE (SIP opt.) | MCU GPIO (red) | ✓ |
| REC ARM | C | none — MIDI to DAW | MCU GPIO (red) | ✓ |
| HPF/INSERT BYPASS | — | HPF/INSERT BYPASS relay | MCU GPIO (3× orange, separate LEDs) | ✓ |
| **Per channel** | | | | **5** |

**Mono channels subtotal: 5 × 24 = 120** ✓

### Other sections (estimates)

| Section | Estimated buttons | Status |
|---|---|---|
| AUX return × 4 | ~2 each (ACTIVE + AFL) → ~8 | ~ |
| Group × 3 | ~2 each (ACTIVE + AFL) → ~6 | ~ |
| AUX master × 4 | 1 each (PFL) → 4 | ~ |
| Monitor / master | ~2–4 (mono/check L+R + TBD) | ~ |

**Console total pushbuttons: ~140–142**

---

## Per-channel MCU I/O summary (confirmed)

| Signal type | Per channel | × 24 channels | Note |
|---|---|---|---|
| GPIO input — buttons | 5 | 120 | CHANNEL SOURCE, ACTIVE/MUTE, SOLO, REC ARM, HPF/INSERT BYPASS |
| GPIO output — LEDs | 8 | 192 | ACTIVE/MUTE, SOLO, REC ARM (integrated), HPF/INSERT BYPASS × 3 (separate), CHANNEL SOURCE A-LED + B-LED |
| Relay coil control lines | 10 (5 relays × 2) | 240 | via dedicated driver ICs, not direct GPIO |

CHANNEL SOURCE LEDs (red/green) are MCU GPIO driven — firmware updates them on each CHANNEL SOURCE toggle.
HPF, INSERT, PFL, PRE-POST LEDs are mech-switch-driven — no GPIO output needed.
HPF/INSERT BYPASS mode LEDs (3× orange) are MCU GPIO driven — separate from the button.
