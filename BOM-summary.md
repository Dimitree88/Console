# BOM Summary — Component Numerosity Reference

> Counts all major components except: resistors, capacitors, diodes, PCB-to-PCB connectors.
> Faders excluded from potentiometer and knob counts.
> **Status:** ✓ = confirmed from finalized blocks; ~ = estimated from sections still conceptual.

---

## 1. Relays

Single type throughout: **AGQ210A03** (Panasonic, DPDT 2 form C, 1-coil latching, 3 V coil)

| Function | Per unit | Units | Total | Status |
|---|---|---|---|---|
| CHANNEL SOURCE (Input A / B select) | 1 | 24 mono ch | 24 | ✓ |
| MUTE (series + shunt) | 1 | 24 mono ch | 24 | ✓ |
| AFL (post-pan stereo tap) | 1 | 24 mono ch | 24 | ✓ |
| DIRECT OUT SELECT (Input A / channel signal) | 1 | 24 mono ch | 24 | ✓ |
| HPF / INSERT BYPASS | 1 | 24 mono ch | 24 | ✓ |
| ACTIVE (stereo, both gangs on one coil) | 1 | 4 AUX return | 4 | ✓ |
| AFL (stereo, both gangs on one coil) | 1 | 4 AUX return | 4 | ✓ |
| ACTIVE (stereo, both gangs on one coil) | 1 | 3 groups | 3 | ✓ |
| AFL (stereo, both gangs on one coil) | 1 | 3 groups | 3 | ✓ |
| Monitor source-select (Main Mix / Solo summer) | 1 | monitor | 1 | ✓ |
| Monitor AFL / PFL alternation | 1 | monitor | 1 | ✓ |
| **TOTAL** | | | **136** | ✓ |

---

## 2. ICs / Opamps

### 2a. Mono channels (×24) — confirmed

| IC | Package | Function | Per ch | Total | Status |
|---|---|---|---|---|---|
| OPA1679 | Quad SOIC | Input buffers — A-hot, A-cold, B-hot, B-cold (unity-gain followers) | 1 | 24 | ✓ |
| OPA1679 | Quad SOIC | Balanced receivers — Input A + B, All-Inverting topology | 1 | 24 | ✓ |
| NE5532 | Dual SOIC | HPF (Sallen-Key 2nd order) + Insert Return receiver | 1 | 24 | ✓ |
| NE5532 | Dual SOIC | Post-fader amp +10 dB — 1 IC shared by 2 ch on fader PCB | 0.5 | 12 | ✓ |
| NE5532 | Dual SOIC | Active pan L + R (Self-style, both halves used) | 1 | 24 | ✓ |
| TL072 | Dual SOIC | Dual meter buffer — active (pre-MUTE) + inactive (CHANNEL SOURCE relay set 2) | 1 | 24 | ✓ |
| **Mono channel subtotal** | | | | **132** | ✓ |

**Per tipo IC — mono ch (per l'ordine; altre sezioni ancora TBD):**

| IC | Totale mono ch | Altre sezioni | Status |
|---|---|---|---|
| OPA1679 | **48** | TBD | ✓ / ~ |
| NE5532 | **60** | TBD | ✓ / ~ |
| TL072 | **24** | TBD | ✓ / ~ |

### 2b. Other sections — TBD (conceptual)

| Section | Notes | Status |
|---|---|---|
| AUX return ×4 | Summing amp, post-fader amps, MONO mixer, balance amp, meter buffers — IC TBD | ~ |
| Group ×3 | Summing amp, post-fader amps, MONO mixer, balance amp, AUX mode mix — IC TBD | ~ |
| AUX master ×4 | PRE + POST summing amps, variable gain stage — IC TBD | ~ |
| CUE master | Stereo summing amp — IC TBD | ~ |
| Main Mix master | Stereo summing amp + SSL-style parallel compressor (VCA / dedicated IC) — IC TBD | ~ |
| Monitor | AFL summer, PFL summer, mono/check network — IC TBD | ~ |

---

## 3. Potentiometers (faders excluded)

### 3a. Mono channels (×24)

| Function | Type | Value | Per ch | Total | Status |
|---|---|---|---|---|---|
| Gain — Input A + B (stesso part, All-Inverting Rf) | Single-gang | 10 kΩ LOG | 2 | 48 | ✓ |
| AUX 1–4 send — tutti stesso part, gang 1 pre-fader + gang 2 post-fader | Dual-gang | TBD | 4 | 96 | ~ |
| CUE send — both gangs pre-fader → CUE L / CUE R | Dual-gang | TBD | 1 | 24 | ~ |
| Pan — Self active panpot (L + R, center-detent preferred) | Dual-gang | 10 kΩ LIN | 1 | 24 | ✓ |
| **Mono channel subtotal** | | | **8 / ch** | **192** | |

### 3b. AUX returns (×4)

| Function | Type | Value | Per unit | Total | Status |
|---|---|---|---|---|---|
| Stereo gain (L + R ganged) | Dual-gang | TBD | 1 | 4 | ~ |
| Active balance (L / R) | Dual-gang | TBD | 1 | 4 | ~ |
| CUE send (stereo, pre-fader) | Dual-gang | TBD | 1 | 4 | ~ |
| **AUX return subtotal** | | | **3 / return** | **12** | |

### 3c. Groups (×3)

| Function | Type | Value | Per unit | Total | Status |
|---|---|---|---|---|---|
| AUX 1–4 send (tutti stesso part, pre + post gangs) | Dual-gang | TBD | 4 | 12 | ~ |
| Active balance (L / R) | Dual-gang | TBD | 1 | 3 | ~ |
| CUE send (stereo, pre-fader) | Dual-gang | TBD | 1 | 3 | ~ |
| **Group subtotal** | | | **6 / group** | **18** | |

### 3d. Master sections

| Function | Type | Value | Section | Total | Status |
|---|---|---|---|---|---|
| Variable gain (post PRE/POST select) | Single-gang | TBD | AUX master ×4 | 4 | ~ |
| Stereo gain (CUE bus) | Dual-gang | TBD | CUE master | 1 | ~ |
| "In front balance" (Main Mix level behind AFL) | TBD | TBD | Monitor | 1 | ~ |
| **Master subtotal** | | | | **6** | |

### Potentiometer total: 192 + 12 + 18 + 6 = **228**

---

## 4. Knobs

> 1 knob per potentiometer + 1 per rotary switch. Fader caps esclusi (§7).
> Colori disponibili (Tayda 15×13 mm plastic knob): **nero, grigio, blu, rosso** — da assegnare per funzione (TBD).

| Funzione | Pot/switch | Count | Colore | Status |
|---|---|---|---|---|
| Gain (Input A + B) — mono ch | Single-gang | 48 | TBD | ~ |
| AUX 1–4 send — mono ch | Dual-gang | 96 | TBD | ~ |
| CUE send — mono ch | Dual-gang | 24 | TBD | ~ |
| Pan — mono ch | Dual-gang | 24 | TBD | ~ |
| Stereo gain — AUX return | Dual-gang | 4 | TBD | ~ |
| Active balance — AUX return | Dual-gang | 4 | TBD | ~ |
| CUE send — AUX return | Dual-gang | 4 | TBD | ~ |
| AUX 1–4 send — groups | Dual-gang | 12 | TBD | ~ |
| Active balance — groups | Dual-gang | 3 | TBD | ~ |
| CUE send — groups | Dual-gang | 3 | TBD | ~ |
| Variable gain — AUX master | Single-gang | 4 | TBD | ~ |
| Stereo gain — CUE master | Dual-gang | 1 | TBD | ~ |
| "In front balance" — monitor | TBD | 1 | TBD | ~ |
| Routing — post-pan (mono ch) | Rotary 4-pos | 24 | TBD | ~ |
| Routing — post-balance (AUX return) | Rotary 4-pos | 4 | TBD | ~ |
| **TOTAL** | | **256** | | |

---

## 5. Rotary switches (routing)

| Function | Type | Per unit | Section | Total | Status |
|---|---|---|---|---|---|
| Post-pan routing → Main / G1 / G2 / G3 | 2-gang, 4-position, mechanical | 1 | 24 mono ch | 24 | ~ |
| Post-balance routing → Main / G1 / G2 / G3 | 2-gang, 4-position, mechanical | 1 | 4 AUX return | 4 | ~ |
| **TOTAL** | | | | **28** | |

---

## 6. Pushbuttons

All front-panel controls share the same physical pushbutton body. Three sub-categories:

- **Momentary with integrated LED** — the LED is inside the button body and is NOT listed separately in §8. Firmware drives both the LED and the relay.
- **Momentary without integrated LED** — spring-return; firmware manages relay and any separate LEDs. **Requires cap.**
- **Latching DPDT** — push-push self-locking; section 1 carries the audio or control signal, section 2 drives a separate LED (those LEDs ARE listed in §8). No firmware involved. **Requires cap.**

### 6a. Momentary — with integrated LED (on fader PCB)

> These 3 LEDs are integral to the button — **not** listed separately in §8.
> Location: fader PCB (same board as fader and post-fader amp). Connection to digital logic PCB: TBD.

| Function | LED color | Per ch | Total | Status |
|---|---|---|---|---|
| ACTIVE / MUTE | Orange | 1 | 24 | ✓ |
| SOLO | Red | 1 | 24 | ✓ |
| REC ARM (MIDI only — no audio relay) | Red | 1 | 24 | ✓ |
| **Subtotal momentary w/ LED** | | **3 / ch** | **72** | ✓ |

### 6b. Momentary — without integrated LED

| Function | Separate LEDs | Per unit | Section | Total | Status |
|---|---|---|---|---|---|
| CHANNEL SOURCE (A/B toggle) | 2 sep. (red A + green B), MCU GPIO | 1 | 24 mono ch | 24 | ✓ |
| HPF / INSERT BYPASS (cycles 3 modes) | 3 sep. orange, MCU GPIO | 1 | 24 mono ch | 24 | ✓ |
| ACTIVE | TBD | 1 | 4 AUX return | 4 | ~ |
| AFL | TBD | 1 | 4 AUX return | 4 | ~ |
| ACTIVE | TBD | 1 | 3 groups | 3 | ~ |
| AFL | TBD | 1 | 3 groups | 3 | ~ |
| PFL | TBD | 1 | 4 AUX master | 4 | ~ |
| Mono/check L | TBD | 1 | monitor | 1 | ~ |
| Mono/check R | TBD | 1 | monitor | 1 | ~ |
| **Subtotal momentary no LED** | | | | **~68** | |

### 6c. Latching DPDT (push-push self-locking — LED driven by button sec. 2, listed in §8)

#### Mono channels (×24)

| Function | LED sec. 2 | Per ch | Total | Status |
|---|---|---|---|---|
| HPF in / bypass | Orange | 1 | 24 | ✓ |
| INSERT in / bypass | Blue | 1 | 24 | ✓ |
| PFL on / off | Red | 1 | 24 | ✓ |
| Output PRE / POST select (2-position) | TBD ×2 | 1 | 24 | ✓ |
| DIRECT / PROCESSED output (controls DIRECT OUT SELECT relay on input PCB) | TBD | 1 | 24 | ~ |
| **Mono channel subtotal** | | **5 / ch** | **120** | |

#### Other sections

| Function | LED | Per unit | Section | Total | Status |
|---|---|---|---|---|---|
| PRE / POST send select | TBD | 1 | AUX master ×4 | 4 | ~ |
| Group → Main Mix on / off | TBD | 1 | Group ×3 | 3 | ~ |
| AUX mode — split / premix (one per AUX) | TBD | 4 | Group ×3 | 12 | ~ |
| **Other subtotal** | | | | **19** | |

### Subtotal latching: 120 + 19 = **139**

### Caps total (momentary no LED + latching): ~68 + 139 = **~207**

---

## 7. Faders

> Esclusi dai conteggi di pot e knob.

| Part | Tipo | Per unità | Sezione | Totale | Status |
|---|---|---|---|---|---|
| Alps RSA0N111900Q | Fader mono lineare | 1 | 24 mono ch | 24 | ✓ |
| Fader stereo gruppo | TBD | 1 | 3 groups | 3 | ~ |
| Fader master / altri | TBD | TBD | Main mix + altri | TBD | ~ |

---

## 8. Indicator LEDs (meter bridge bargraph LEDs excluded — counted separately)

### 8a. Mono channels (×24) — confirmed

> ACTIVE/MUTE, SOLO, REC ARM LEDs are integrated in their pushbutton bodies — **not listed here** (see §6a).

| Function | Color | Drive source | Per ch | Total | Status |
|---|---|---|---|---|---|
| CHANNEL SOURCE — Input A active | Red | MCU GPIO | 1 | 24 | ✓ |
| CHANNEL SOURCE — Input B active | Green | MCU GPIO | 1 | 24 | ✓ |
| HPF/INS BYPASS — FOLLOW PATH mode | Orange | MCU GPIO | 1 | 24 | ✓ |
| HPF/INS BYPASS — FOLLOW A mode | Orange | MCU GPIO | 1 | 24 | ✓ |
| HPF/INS BYPASS — FOLLOW B mode | Orange | MCU GPIO | 1 | 24 | ✓ |
| HPF in | Orange | Latching button sec. 2 | 1 | 24 | ✓ |
| INSERT in | Blue | Latching button sec. 2 | 1 | 24 | ✓ |
| PFL active | Red | Latching button sec. 2 | 1 | 24 | ✓ |
| Output PRE-POST — position 1 | TBD | Latching button sec. 2 | 1 | 24 | ~ |
| Output PRE-POST — position 2 | TBD | Latching button sec. 2 | 1 | 24 | ~ |
| **Mono channel subtotal** | | | **10 / ch** | **240** | ✓ |

**By color — mono channels only:**

| Color | Count | Notes |
|---|---|---|
| Red | 48 | CHANNEL SOURCE A (24) + PFL (24) |
| Green | 24 | CHANNEL SOURCE B |
| Orange | 96 | HPF/INSERT BYPASS ×3 (72) + HPF (24) |
| Blue | 24 | INSERT |
| TBD | 48 | Output PRE-POST ×2 per channel |

### 8b. Other sections — estimated

| Section | Estimated LEDs | Status |
|---|---|---|
| AUX return ×4 | ~5–6 each → **~20–24** | ~ |
| Group ×3 | ~10–12 each → **~30–36** | ~ |
| AUX master ×4 | ~2–3 each → **~8–12** | ~ |
| CUE master | **~2** | ~ |
| Main Mix master | **~3–5** | ~ |
| Monitor section | **~3–5** | ~ |
| **Other sections subtotal** | **~66–84** | ~ |

### Indicator LED total: **~306–324** (mid estimate ~315)

---

## 9. TRS Jacks

Single type throughout: **Rean RJ5VM** — TRS stereo-with-switch (5-contact). Switch contacts (TIP-SW / RING-SW) used for idle-noise termination — they short TIP↔TIP-SW and RING↔RING-SW when no plug is inserted — on: Input A, Input B, and Insert Return. Left unconnected on all output jacks and Insert Send.

| Function | Per unit | Section | SW contacts used | Total | Status |
|---|---|---|---|---|---|
| Input A | 1 | 24 mono ch | ✓ idle termination | 24 | ✓ |
| Input B | 1 | 24 mono ch | ✓ idle termination | 24 | ✓ |
| Channel Output (rear panel, switchable: direct / processed) | 1 | 24 mono ch | — | 24 | ✓ |
| Insert Send (impedance-balanced) | 1 | 24 mono ch | — | 24 | ✓ |
| Insert Return | 1 | 24 mono ch | ✓ idle termination | 24 | ✓ |
| Input L | 1 | 4 AUX return | TBD | 4 | ~ |
| Input R | 1 | 4 AUX return | TBD | 4 | ~ |
| Insert Send | 1 | 3 groups | — | 3 | ~ |
| Insert Return | 1 | 3 groups | TBD | 3 | ~ |
| Post-Fader Out L (impedance-balanced) | 1 | 3 groups | — | 3 | ~ |
| Post-Fader Out R (impedance-balanced) | 1 | 3 groups | — | 3 | ~ |
| AUX n OUT (impedance-balanced) | 1 | 4 AUX master | — | 4 | ~ |
| CUE OUT L (impedance-balanced) | 1 | CUE master | — | 1 | ~ |
| CUE OUT R (impedance-balanced) | 1 | CUE master | — | 1 | ~ |
| Main Out A L (impedance-balanced) | 1 | Main mix | — | 1 | ~ |
| Main Out A R (impedance-balanced) | 1 | Main mix | — | 1 | ~ |
| Main Out B L (impedance-balanced) | 1 | Main mix | — | 1 | ~ |
| Main Out B R (impedance-balanced) | 1 | Main mix | — | 1 | ~ |
| Main Insert Send L | 1 | Main mix | — | 1 | ~ |
| Main Insert Send R | 1 | Main mix | — | 1 | ~ |
| Main Insert Return L | 1 | Main mix | TBD | 1 | ~ |
| Main Insert Return R | 1 | Main mix | TBD | 1 | ~ |
| **TOTAL** | | | | **154** | |

> Monitor output jacks not included (TBD).

---

## Grand summary — per part number

> Questo è il reference per l'ordine. Items con part number TBD si sdoppieranno in più righe una volta definiti i valori.

| Part number / tipo | Descrizione | Qty | In possesso | Da comprare | Status |
|---|---|---|---|---|---|
| **AGQ210A03** | Relay DPDT latching 3 V (Panasonic) | 136 | 122 | 14 | ✓ |
| **OPA1679** | Quad opamp SOIC-14 | ≥48 | — | ≥48 | ✓ mono ch; altre sez. TBD |
| **NE5532** | Dual opamp SOIC-8 | ≥60 | — | ≥60 | ✓ mono ch; altre sez. TBD |
| **TL072** | Dual opamp SOIC-8 | ≥24 | — | ≥24 | ✓ mono ch; altre sez. TBD |
| **Alps RSA0N111900Q** | Fader mono lineare | 24 | 24 | 0 | ✓ |
| Fader stereo (part TBD) | Fader stereo — groups + master | TBD | — | TBD | ~ |
| **Rean RJ5VM** | Jack TRS 5-contact con switch | 154 | — | 154 | ✓/~ |
| Pot single-gang 10 kΩ LOG — [Tayda](https://www.taydaelectronics.com/10k-ohm-logarithmic-taper-potentiometer-round-shaft-pcb-9mm.html) | Gain (Input A + B, mono ch) | 48 | — | 48 | ✓ |
| Pot dual-gang 10 kΩ LIN — [Tayda](https://www.taydaelectronics.com/10k-ohm-linear-dual-taper-potentiometer-round-shaft-pcb-9mm.html) | Pan (mono ch) | 24 | — | 24 | ✓ |
| Pot dual-gang (value TBD) | AUX send (96 ch + 12 gr) + CUE send (24 ch + 4 ret + 3 gr) + balance (4 ret + 3 gr) + CUE master (1) = 147 | 147 | — | 147 | ~ |
| Pot single-gang (value TBD) | AUX master variable gain | 4 | — | 4 | ~ |
| Pot TBD (gang + value TBD) | Monitor "in front balance" | 1 | — | 1 | ~ |
| Rotary switch 2-gang 4-pos (part TBD) | Routing post-pan + post-balance | 28 | — | 28 | ~ |
| Pushbutton momentary w/ integrated LED (part TBD) | ACTIVE/SOLO/REC ARM — fader PCB | 72 | — | 72 | ✓ |
| Pushbutton momentary no LED (part TBD) | Channel source, HPF/INS bypass, AUX/group/master controls | ~68 | 300 | 0 | ✓/~ |
| Pushbutton latching DPDT (part TBD) | HPF, INSERT, PFL, PRE/POST, DIRECT/PROC, AUX mode | ~139 | 300 | 0 | ✓/~ |
| Button cap (part TBD) | Per momentary no LED + latching | ~207 | 300 | 0 | ~ |
| Knob (colore TBD) — [Tayda](https://www.taydaelectronics.com/catalogsearch/result/?q=15x13mm+plastic+knob) | 1 per pot + 1 per rotary | 256 | — | 256 | ~ |
| Indicator LED (color mix — see §8) | Separate LEDs, non integrati nei tasti | ~315 | — | ~315 | ✓/~ |
