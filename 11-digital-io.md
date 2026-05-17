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
| Mono channel | SOURCE (A/B select) | 1 | 24 | 24 | ✓ |
| Mono channel | MUTE (ACTIVE/SOLO) | 1 | 24 | 24 | ✓ |
| Mono channel | AFL | 1 | 24 | 24 | ✓ |
| Mono channel | Channel Output SELECT | 1 | 24 | 24 | ✓ |
| Mono channel | FX-FOLLOW | 1 | 24 | 24 | ✓ |
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

> ACTIVE (orange), SOLO (red), REC (red) LEDs are **integrated
> in their pushbutton bodies** (located on the fader PCB). They are MCU
> GPIO driven but are **not listed here** as separate indicators —
> they ship with the button.

| LED name | Color | Drive |
|---|---|---|
| LED-SOURCE-A | red | MCU GPIO |
| LED-SOURCE-B | green | MCU GPIO |
| LED-FX-FOLLOW-FOLLOW-PATH | orange | MCU GPIO |
| LED-FX-FOLLOW-FOLLOW-A | orange | MCU GPIO |
| LED-FX-FOLLOW-FOLLOW-B | orange | MCU GPIO |
| LED-HPF | orange | HPF button sec. 2 |
| LED-INSERT | blue | INSERT button sec. 2 |
| LED-PFL | red | PFL button sec. 2 |
| LED-PRE | orange | OUT-PRE/POST sec. 2, COM gated by DIR/PROC sec. 2 NO |
| LED-POST | orange | OUT-PRE/POST sec. 2, COM gated by DIR/PROC sec. 2 NO |
| LED-DIRECT | orange | DIR/PROC sec. 2 NC |
| **Per channel total** | **11** | |

**Mono channels subtotal: 11 × 24 = 264** ✓

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

**Console total indicator LEDs: ~402–420** (round estimate: ~410)

> Meter bridge LEDs (signal-level meters) are driven by dedicated
> LED driver ICs on the meter PCBs — counted separately when the
> meter bridge is designed.

---

## Front-panel pushbuttons

### Per mono channel (× 24) — 10 per channel

**Momentary with integrated LED — on fader PCB (3 per channel):**

| Button | LED color | Relay / action | LED drive | Status |
|---|---|---|---|---|
| ACTIVE | orange (in button) | MUTE relay | MCU GPIO | ✓ |
| SOLO | red (in button) | AFL relay + others' MUTE (SIP opt.) | MCU GPIO | ✓ |
| REC | red (in button) | none — MIDI to DAW | MCU GPIO | ✓ |

Connection from fader PCB to digital logic PCB: TBD.

**Momentary without integrated LED (2 per channel):**

| Button | Separate LEDs | Relay / action | LED drive | Status |
|---|---|---|---|---|
| SOURCE | LED-SOURCE-A (red) + LED-SOURCE-B (green) | SOURCE relay | MCU GPIO | ✓ |
| FX-FOLLOW | LED-FX-FOLLOW-FOLLOW-PATH / -A / -B (3× orange) | FX-FOLLOW relay (3 modes) | MCU GPIO | ✓ |

**Latching DPDT (5 per channel):**

| Button | LED sec. 2 | Signal / relay | MCU GPIO in? | Status |
|---|---|---|---|---|
| HPF | orange | audio path direct | no | ✓ |
| INSERT | blue | audio path direct | no | ✓ |
| PFL | red | audio path direct | no | ✓ |
| OUT-PRE/POST | TBD × 2 | audio path direct | no | ✓ |
| DIR/PROC | 1 LED (DIRECT, TBD color) via sec. 2 NC; sec. 2 NO → PRE-POST sec. 2 COM (gates PRE/POST LEDs) | sec. 1 → MCU GPIO in; MCU fires Channel Output SELECT relay | yes | ~ |

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
| GPIO input — buttons | 6 | 144 | SOURCE, ACTIVE, SOLO, REC, FX-FOLLOW, DIR/PROC (latching button sec. 1) |
| GPIO output — LEDs | 8 | 192 | ACTIVE, SOLO, REC (integrated in button on fader PCB), LED-FX-FOLLOW-FOLLOW-PATH / -A / -B, LED-SOURCE-A + LED-SOURCE-B |
| Relay coil control lines | 10 (5 relays × 2) | 240 | via dedicated driver ICs, not direct GPIO |

LED-SOURCE-A and LED-SOURCE-B: MCU GPIO driven — updated on each SOURCE toggle.
ACTIVE, SOLO, REC: MCU GPIO driven — integrated in button body (fader PCB); GPIO wire routes fader PCB → digital logic PCB (TBD).
LED-FX-FOLLOW-FOLLOW-PATH, LED-FX-FOLLOW-FOLLOW-A, LED-FX-FOLLOW-FOLLOW-B: MCU GPIO driven — separate from the button.
LED-HPF, LED-INSERT, LED-PFL: latching button sec. 2 driven directly — no MCU GPIO output needed.
LED-DIRECT, LED-PRE, LED-POST: chained — DIR/PROC sec. 2 NC drives LED-DIRECT; sec. 2 NO feeds OUT-PRE/POST sec. 2 COM which drives LED-PRE or LED-POST depending on OUT-PRE/POST switch position. No MCU GPIO output needed for any of the three LEDs.
