# 10 — Open / TBD

Items deliberately unresolved at this stage. As they get resolved,
remove them from here and record the decision in the relevant
section's Implementation details.

---

## Block-specific open issues

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

- **Meter buffer opamp** — part TBD. Candidates: another OPA1642, or
  an audio-grade dual where the second half can be reused for a
  future stage in Block 4.
- **Channels per input PCB** (likely 4 or 6) — TBD. This decision
  drives the partitioning of the digital control connector for the
  ADG419 IN signals: one wide connector per PCB carrying all
  per-channel mute lines, vs one narrow connector per channel.
- **Other PFL sources feeding PFL DETECT**: AUX masters (`05`),
  AUX returns (`03`), and groups (`04`) all carry PFL controls. Do
  they all need to feed the PFL DETECT logical bus to drive the
  firmware source-select? — TBD. Probably yes for coherent
  monitor-switching semantics, but to be confirmed when those
  sections are detailed.
- **PFL signal diode** — 1N4148-class, exact part TBD.
- **PFL LED current-limit resistor value** — depends on chosen LED
  Vf and brightness target.

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
- **DPDT mechanical switch** part selection for CHANNEL SOURCE / HPF
  / INSERT.
- **Ferrite chip part selection** — 330 Ω @ 100 MHz, many
  equivalents; choose by current rating and package.

---

## Conceptual / design-phase open issues

- **Monitor-section output destination** — control room? headphones?
  Both? — path ends at the mono/check buttons currently.
- **AFL vs PFL priority** when both are simultaneously active —
  firmware decision.
- **SOLO vs ACTIVE exact semantics** — the electronic switches exist
  in hardware; whether SOLO-in-place mutes non-solo channels, whether
  AFL is latching or destructive to the main mix, etc., are firmware
  choices.
- **Fader law / taper** — not specified.
- **Op-amp choices for stages downstream of INSERT** — TBD (meter
  buffer, ACTIVE switch, fader amp, post-fader +10 dB, active pan,
  AUX/CUE/PFL sends, all summers).
- **PSU design** — TBD.
- **Channel physical layout** (which knob sits where) — not
  specified.
- **System-wide grounding scheme** — AGND / DGND / Chassis kept
  separate per-PCB. Joining topology (star point at PSU, ferrite
  bead, direct bond, etc.) not yet decided.
