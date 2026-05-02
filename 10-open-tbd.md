# 10 — Open / TBD

Items deliberately unresolved at this stage. As they get resolved,
remove them from here and record the decision in the relevant
section's Implementation details.

---

## Block-specific open issues

### From mono channel Block 1 (input + receiver)

- **Phase-lead compensation on balanced receivers** — Jung/Kester
  Fig. 6-26 describes an optional R-C trim network to improve HF CMR
  by ≈ 30 dB at 10 kHz. Not implemented in first rev. Footprints will
  be present on PCB; populate in rev 2 if measured CMR at 10 kHz is
  insufficient.
- **Receiver matched-resistor part selection** — 0.1 % thin-film
  recommended; exact part TBD (candidates: Susumu RR0816P, Vishay
  MCT0603, etc.).
- **Bipolar electrolytic brand / series** for signal-path coupling
  (candidates: Nichicon Muse ES, Panasonic SU, Elna Silmic bipolar).
- **DPDT mechanical switch** part selection for HPF / INSERT.
  (CHANNEL SOURCE is no longer a mechanical DPDT — it is a
  momentary pushbutton + logic latch + FTR-B3 relay per
  `00-conventions.md`; the front-panel pushbutton part is itself
  TBD.)
- **Ferrite chip part selection** — 330 Ω @ 100 MHz, many
  equivalents; choose by current rating and package.

### From mono channel Block 2 (HPF + Insert)

- **HPF voicing** — topology copied from another mixer produces
  3rd-order rolloff with a +5 dB resonant peak around 80 Hz (Q ≈ 3.46
  on the Sallen-Key stage). To revisit: confirm as intentional voicing
  or rebalance toward Butterworth (Q ≈ 0.7, R24 → ~31 kΩ). See
  `02-mono-channel.md` §2.10 Block 2.
- **Send cold dummy network — low-frequency CMR**: the 47 µF dummy
  cap introduces a tip vs ring impedance mismatch below ~100 Hz,
  degrading CMR exactly where mains hum lives. Default pattern in
  `00-conventions.md` reflects current choice; revise (larger dummy
  cap or alternative topology) if measured hum rejection on Insert is
  insufficient.
- **HPF film cap dielectric** (PPS / PET / PP / NPO depending on
  physical size at 220 nF) — TBD.
- **HPF switch and INSERT switch part selection** (DPDT, front-panel,
  current rating adequate for LED section).

### From mono channel Block 3 (meter buffer + PFL + MUTE)

- **Channels per input PCB** (likely 4 or 6) — TBD. This decision
  drives the partitioning of the digital control connector for the
  MUTE relay coil drives: one wide connector per PCB carrying all
  per-channel mute lines, vs one narrow connector per channel.
  Note: the **fader PCB** partitioning is settled (2 channels per
  fader PCB — see Block 4); the input PCB partitioning is
  independent.
- **Other PFL sources feeding PFL DETECT**: AUX masters (`05`),
  AUX returns (`03`), and groups (`04`) all carry PFL controls. Do
  they all need to feed the PFL DETECT logical bus to drive the
  firmware source-select? — TBD. Probably yes for coherent
  monitor-switching semantics, but to be confirmed when those
  sections are detailed.
- **PFL signal diode** — 1N4148-class, exact part TBD.
- **PFL LED current-limit resistor value** — depends on chosen LED
  Vf and brightness target.

### From mono channel Block 4 (pre-fader node + fader PCB + post-fader amp + AUX/CUE sends)

- **C_bp** (bipolar DC block at the fader wiper, on the fader PCB):
  value TBD.
- **R_bias** (wiper to AGND after C_bp): value TBD. Together with
  C_bp these set the LF rolloff into the post-fader amp's +input
  and define the wiper bias condition at startup. With NE5532
  (I_B ~ 200 nA) the order-of-magnitude target is 47 kΩ → ~9 mV
  DC at the +input, absorbed by the 220 µF output DC block.
- **Polarity of the 220 µF output DC block** at the fader PCB
  output: depends on DC at the next stage (AUX-post gang tops and
  pan top on the channel PCB). To be set when measured on rev 1.
- **AUX dual-gang pot value** (× 4) and **AUX bus summing resistor
  values** (8 total per channel: 4 pre + 4 post). To be calculated
  together with the AUX master summing topology (`05`).
- **CUE dual-gang pot value** (× 1) and **CUE bus summing resistor
  values** (× 2: L + R per channel). To be calculated together with
  the CUE master summing topology (`06`).
- **Fader part / taper / law** — not specified.
- **3-pin connector type / cable** between channel PCB and fader
  PCB (twisted pair shielded vs flat ribbon): TBD.

### From mono channel Block 5 (active pan)

- **Pan pot exact part** — dual-gang 10 kΩ LIN center-detent
  preferred; model TBD.

### From mono channel Block 6 (post-pan routing rotary)

- **Bus summing resistor values** (8 per channel × 24 channels =
  192 resistors from this stage alone, plus contributions from AUX
  returns and groups via their own routing). To be solved together
  with `04` (group summers) and `07` (Main summer) — they form one
  coherent system.
- **Rotary switch part selection** — 2-gang, 4-position, mechanical,
  front-panel: TBD.
- **Make-before-break vs break-before-make** — probably
  break-before-make to avoid momentary cross-routing during a
  position change.

### Mono channel — AFL switch (post-pan, stereo) — partly decided

`02-mono-channel.md` §2.9 (b). The conceptual signal flow is fixed
(parallel tap from post-pan L/R to AFL bus L/R, controlled by an
electronic switch under digital logic). The switching element is
fixed: **one FTR-B3GA4.5Z-B10 per channel** (DPDT 2 form C, both
contact sets ganged on a single coil — contact 1 = L, contact 2 = R)
per `00-conventions.md`. Still to be decided:

- Where on the post-pan signal the tap sits — at the opamp output
  pin directly (Block 5), or further down the chain.
- Topology around the relay: NO contacts to the AFL bus only when
  active, NC contacts left floating (simplest); or NC tied to
  AGND for shunt to-ground when off (better off-isolation, but
  ≥ 80 dB open-contact isolation is already ample at audio so
  unnecessary).
- Bus summing resistor toward the AFL bus L/R (these are the
  channel's contribution to the AFL summer in §08).

### Mono channel — Post-Fader Output (mono, jack out) — not yet designed

`02-mono-channel.md` §2.9 (e). The conceptual signal flow is fixed
(tap from POST-FADER node, before pan, to a rear-panel TRS
impedance-balanced jack). Still to be decided:

- Direct tap vs buffered tap. The POST-FADER node is already loaded
  by 4 AUX-post gangs and the pan pot; adding a TRS jack with
  EMI filter and dummy network adds further loading and any cable
  /connection changes downstream of the jack would slightly alter
  the pan stage's source impedance. A dedicated buffer isolates
  the post-fader amp and keeps the jack independent of internal
  loading variations.
- Output stage topology: default impedance-balanced per `00-
  conventions.md` (75 Ω tip + 75 Ω + 47 µF ‖ 47 kΩ cold dummy),
  or ground-cancelling alla Soundcraft (compensates for GND
  difference at the receiving device, gives most of the benefit
  of a balanced link into an unbalanced output)?
- EMI filter: standard T-balanced double-pi per `00-conventions.md`.

---

## Conceptual / design-phase open issues

- **Standard signal relay — driver IC, partitioning, power budget,
  protection.** The standard electronic switch in CONSOLE is the
  **FTR-B3GA4.5Z-B10** (per `00-conventions.md`). Approximately
  88 relays total across the console (24 mono channels × 3
  positions — CHANNEL SOURCE, MUTE, AFL — + 4 AUX returns × 2 +
  3 groups × 2 + 2 in master monitor). Open items:
  - **Coil driver IC**: ULN2803-class octal Darlington (with
    integrated flyback diodes) vs discrete BJT/MOSFET per coil,
    vs a dedicated relay-driver IC.
  - **Driver partitioning**: centralized (one driver bank near
    the firmware) vs per-PCB (driver lives next to the relays it
    powers). Coupled to the channels-per-input-PCB decision in
    Block 3.
  - **+5 V coil rail PSU sizing**: worst-case all-energized
    ≈ 12 W on the +5 V rail (88 × 140 mW). Average load much
    lower but the rail must be designed for the worst case
    headroom.
  - **Optional 22 Ω series resistor** per coil to drop the +5 V
    rail to nominal 4.5 V — population decided at layout time
    after measuring coil temperature rise on rev 1.
  - **Coil flyback diode** (1N4148-class): external when the
    driver doesn't integrate one; not needed with ULN2803.
  - **Relay coil RF / audible click radiation**: PCB layout to
    keep coil currents tightly looped, away from audio traces.
- **Selective opamp upgrade — budget-review stage.** Default audio
  opamp is NE5532 per `00-conventions.md`. At the prototype /
  measurement stage, identify positions where NE5532 limitations
  (~200 nA bipolar input bias current, ~5 nV/√Hz noise, BJT
  CM-distortion at high source impedance) measurably impact the
  result. Strongest candidates within the mono channel:
  - **Block 5 active pan** — +input sees a high-Z, position-dependent
    source (wiper of 10 kΩ pot in parallel with R_LAW = 3.3 kΩ from
    the output). Both bias-current offset stability and CM-distortion
    benefit from a JFET / CMOS input.
  - **Block 2 HPF Sallen-Key** — +input swings the full audio signal
    at high Z (62 kΩ to AGND); CM-distortion is the most visible
    penalty of NE5532 here.
  - Less critical but worth review: Block 4 post-fader amp, meter
    buffer.
  Upgrade options: **OPA1642** (JFET, low CM-distortion, drop-in,
  ~negligible bias current) or **OPA1656** (CMOS, lowest noise of the
  three at 1.6 nV/√Hz, drop-in). Topology and all passive values
  (R_FB / R_G / R_LAW / C_FB / etc.) remain unchanged across any of
  these substitutions — only the part marking on the SOIC changes.
- **Monitor-section output destination** — control room? headphones?
  Both? — path ends at the mono/check buttons currently.
- **AFL vs PFL priority** when both are simultaneously active —
  firmware decision.
- **SOLO vs ACTIVE exact semantics** — the electronic switches exist
  in hardware; whether SOLO-in-place mutes non-solo channels, whether
  AFL is latching or destructive to the main mix, etc., are firmware
  choices.
- **Op-amp choices for stages still conceptual**: AUX/CUE/PFL summer
  opamps (master sections `05` `06`), Main summer opamp (`07`),
  AFL/PFL summer opamps (`08`), monitor-section switches, meter-bridge
  active stages. Default per `00-conventions.md` is NE5532 unless a
  specific Block argues otherwise.
- **PSU design** — TBD.
- **Channel physical layout** (which knob sits where) — not
  specified.
- **System-wide grounding scheme** — AGND / DGND / Chassis kept
  separate per-PCB. Joining topology (star point at PSU, ferrite
  bead, direct bond, etc.) not yet decided.
