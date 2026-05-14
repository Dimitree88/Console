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
  momentary pushbutton + firmware-driven AGQ210A03 latching relay
  per `00-conventions.md`; the front-panel pushbutton part is
  itself TBD.)
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
- **INSERT blue LED Vf headroom on +3.3 V rail.** Standard blue
  LED Vf is 3.0–3.4 V, leaving very little margin across the
  current-limit resistor on a +3.3 V supply. Either source a
  low-Vf blue (≈ 2.7 V) at the chosen brightness, or revisit the
  LED color choice for the INSERT indicator. To resolve when
  selecting the actual LED parts.

### From mono channel Block 3 (meter buffer + PFL + MUTE)

- **Channels per input PCB** (likely 4 or 6) — TBD. Note: this is
  now decoupled from the relay-coil-drive routing question: per
  `00-conventions.md` ("PCB partitioning — stackable two-board
  pattern"), each relay's coil exits the audio PCB through its
  own 3-pin connector to a lower stackable digital PCB, so the
  number of channels per input PCB no longer dictates a
  per-channel-mute connector layout. The **fader PCB** partitioning
  is settled (2 channels per fader PCB — see Block 4); the input
  PCB partitioning remains independent and TBD.
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

### Mono channel — AFL switch (post-pan, stereo) — FINALIZED (summing values TBD)

`02-mono-channel.md` §2.9 (b) and Block 7. Topology now fully fixed:
tap at the post-pan opamp output (Block 5); per-side bus summing
resistor → COM of AGQ210A03; NC contacts to AGND (constant termination
when AFL off, click-free); NO contacts to AFL Left / AFL Right bus.

Still to be decided:

- **AFL bus summing resistor value** (× 2 per channel, L and R):
  TBD together with §08 AFL summer design (source count, individual
  resistors, and summer opamp gain form one coherent system).

### Mono channel — Block 8 (Output PRE-POST switch + switchable Direct Out) — in-progress

`02-mono-channel.md` §2.9 (e) and Block 8. The standalone
Post-Fader Output dedicated jack (original §2.9 (e)) is superseded:
channel signal is now routed to the input PCB via a 2-pin connector,
and the Direct Out TRS jack is made switchable via the DIRECT OUT
SELECT relay. Topology is fixed; remaining open items:

- **Output PRE-POST LED color assignment**: two LEDs, one per
  position (PRE / POST). Colors not yet assigned — to be decided
  together with the full front-panel LED color scheme.
- **DIRECT OUT SELECT front-panel control type**: relay-driven
  (AGQ210A03) regardless. Whether the front-panel element is a
  momentary pushbutton (Pattern B, always via MCU) or a mechanical
  toggle routed through MCU is TBD pending inter-PCB wiring
  decisions.
- **DIRECT OUT SELECT relay boot state**: RESET (Input A default)
  is the expected choice; to be confirmed with the firmware
  boot-init spec.

---

## Conceptual / design-phase open issues

- **Standard signal relay — driver IC, partitioning, supply,
  protection.** The standard electronic switch in CONSOLE is the
  **AGQ210A03** (1-coil latching, 3 V coil, per
  `00-conventions.md`). Approximately 112 relays total across the
  console (24 mono channels × 4 positions — CHANNEL SOURCE, MUTE,
  AFL, DIRECT OUT SELECT — + 4 AUX returns × 2 + 3 groups × 2 +
  2 in master monitor).
  Open items:
  - **Coil driver IC**: bipolar / half-H-bridge per relay
    (set-pulse and reset-pulse drive). Candidates: Toshiba
    TBD62783A high-side + TBD62083A low-side pair sharing 8
    channels, dedicated octal latching-relay drivers
    (e.g. MAX4820 / MAX4821), per-relay discrete NPN+PNP pair, or
    purpose-built relay-driver SoCs. Choice and protection
    network (freewheeling diodes / suppression for the bipolar
    drive) decided together.
  - **Coil supply rail**: settled — direct drive from the +3.3 V
    logic rail (the 3 V coil tolerates +3.3 V transiently, within
    150 % max allowable; the pulse-only duty cycle makes the small
    overdrive thermally irrelevant). Local bulk on the +3.3 V
    plane sized to absorb pulse-current transients per PCB.
  - **Coil rail bulk capacitance per PCB**: pulse current
    transients up to ~33 mA per relay × N relays simultaneously
    toggled. Firmware staging keeps the worst case manageable;
    bulk cap sized to limit ΔV during a pulse.
  - **Firmware boot-init protocol**: at every cold boot, every
    relay must be driven to its known reset state by an explicit
    pulse. Scheduling and progress reporting to the master-mute
    release logic (see "Master-output hard-mute" below) is part
    of the firmware spec.
  - **Relay coil RF / audible click radiation**: PCB layout to
    keep coil currents tightly looped, away from audio traces.
- **Master-output hard-mute during firmware boot init.** With the
  switch to a latching standard signal relay, the per-position
  fail-safe-by-construction at power-up is gone (a latching relay
  holds whatever state it was last commanded into; transport /
  installation impact may also disturb it). A separate hardware
  mute on the master output stage must hold the room silent until
  firmware finishes initializing all latching relays to their
  designated reset states. Topology of that master mute (relay,
  CMOS analog switch, depletion-mode FET shunt, etc.), its
  release semantics (firmware "ready" handshake), and its boot
  default direction are part of the master-section design, not
  yet started.
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
