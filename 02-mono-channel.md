# 02 — Mono Input Channel (× 24)

Signal flow, top-to-bottom, followed by Implementation details
(Block 1, Block 2, …).

---

## 2.1 Input stage (balanced, dual)

Two independent balanced inputs on TRS:

- **Input A** — intended for instruments (DI / mic-level-ish). Balanced
  TRS.
- **Input B** — intended for DAW return (DAC output). Balanced TRS.

Each input has:

1. **Differential buffer** — two high-Z buffers, one for hot, one for
   cold (forms the first half of the receiver).
2. **Unbalancing stage** — converts hot/cold to single-ended.
3. **Variable gain stage** — the unbalancing and gain are combined in a
   single balanced-receiver topology (see §2.10 Block 1). Each input
   (A and B) has its **own independent gain pot** — they are not
   ganged.

**Direct Out** (Input A only): tapped from the two differential buffers
of Input A, BEFORE the unbalancing stage. This gives a
**full-differential** direct out (both hot and cold are driven by the
buffer output), sent on a TRS jack. Intended use: feed an ADC / DAW
input for tracking.

Note: Input A's Direct Out is a fully differential output (not merely
impedance-balanced), preserving CMRR into a differential ADC. Other
impedance-balanced outs on the console are single-ended with a dummy
cold-side network (see `00-conventions.md`).

---

## 2.2 CHANNEL SOURCE switch (A / B select)

A **DPDT mechanical switch** (front-panel) selects which of the two
unbalanced-and-gained signals continues down the channel. Only one at
a time. Named **CHANNEL SOURCE** to avoid confusion with the master
Monitor section (`08-monitor.md`).

The DPDT's two sections serve different purposes:

- **Section 1** — switches the audio signal (A or B onto the channel
  path).
- **Section 2** — switches power to two indicator LEDs on the front
  panel:
  - **A selected** → red LED on.
  - **B selected** → green LED on.

The LED supply and its return are on +5V / DGND, not on the audio
±15V / AGND — see `00-conventions.md`.

---

## 2.3 HPF

Switchable **fixed-frequency high-pass filter** with a characteristic
voicing (resonant peak before the rolloff, see §2.10 Block 2). The HPF
stage is always powered; a DPDT mechanical switch (front-panel) selects
between the unfiltered and filtered signal path. The DPDT's second
section drives an **orange LED** indicating HPF IN.

---

## 2.4 Insert

Send/return insert point.

- **Send** — always active, impedance-balanced TRS jack. Taps the
  signal after HPF, before the INSERT switch.
- **Return** — switchable. TRS jack, balanced receiver (single-opamp
  4-resistor topology), unity gain.
- **Insert switch** — DPDT mechanical, front-panel. Section 1 alternates
  the onward signal between:
  - the send tap (i.e., return path bypassed), OR
  - the return input (i.e., external device in the insert loop).
  Section 2 drives a **blue LED** indicating INSERT IN.

The send and return TRS jacks live on a separate small PCB (the "jack
PCB"). The channel PCB and jack PCB are connected by a 5-pin
connector — see §2.10 Block 2 for pin assignment.

---

## 2.5 Pre-mute taps

The post-INSERT mono signal feeds two parallel taps before reaching
the MUTE switch.

### 2.5.1 Meter buffer

A high-Z buffer taps the signal and drives the LED meter of the
channel via a 2-pin connector to a separate **meter bridge PCB**
(rectifier, log amp, LED driver IC, and meter LEDs all live on the
meter bridge, not on the channel PCB). Does not interrupt the main
path.

### 2.5.2 PFL switch

A pre-fader, **pre-mute** tap to the PFL mono bus. The signal
encounters a series resistance and then the pole of a DPDT
mechanical front-panel switch.

- **Section 1** (audio): alternates the resistor's far end between
  AGND (PFL OFF) and the PFL mono bus (PFL ON). The upstream node
  sees a fixed light load identical in OFF and ON, so no DC step on
  switching → click-free.
- **Section 2** (control): pole = +5 V (LED rail per §00). In PFL
  ON, drives a **red front-panel LED** AND contributes via a
  wired-OR diode to the **PFL DETECT logical bus** that firmware
  monitors to drive the master monitor source-select (§8.1).

PFL listen therefore continues to work even when the channel is
muted by the MUTE switch downstream — coherent live-sound semantics
("pre-fader listen always tells you what is at the channel
input").

---

## 2.6 ACTIVE / SOLO electronic switch (MUTE)

An **electronic switch** controlled by digital logic interrupts the
main signal path. This is the on/off point used by the firmware to
implement SOLO and/or ACTIVE muting behavior — informally referred
to as the **MUTE** switch. (Exact behavior — mute-on-solo, AFL-only,
etc. — will be decided in firmware; the hardware just provides an
interruptible point.)

The output of this switch is the **PRE-FADER node** — source of all
post-mute pre-fader taps (CUE, AUX gang 1) and of the fader itself.

---

## 2.7 Fader + post-fader amp

- **Fader** — standard linear or log-audio mono fader.
- **Post-fader amp** — +10 dB fixed gain to recover headroom after
  the fader attenuates.

---

## 2.8 Active pan

**Active pan** circuit driven by a **stereo pot** (two ganged sections,
wired for constant-power pan). The channel **becomes stereo from this
point on**: two lines (L and R) travel together for the rest of the
channel.

---

## 2.9 Post-pan distribution

From the post-pan stereo signal, several destinations tap in parallel:

### (a) 4-position rotary switch — main destination

Routes the stereo signal to exactly one of:

- Main Mix
- Group 1
- Group 2
- Group 3

Only one destination at a time. Mechanical rotary.

### (b) AFL electronic switch

Parallel tap, stereo. An electronic switch (digital control) connects
the channel's stereo post-pan signal to the AFL bus when the AFL logic
calls for it.

### (c) 4 × AUX dual-gang pots (mono sends)

Four knobs, each a dual-gang pot:

- **Gang 1** is fed by the **pre-fader** tap (taken from before the
  fader, i.e. mono, after the ACTIVE/SOLO switch but before the fader
  and post-fader amp — effectively at the pre-fader point of the
  channel). Its wiper feeds **AUX n PRE** bus.
- **Gang 2** is fed by the **post-fader** tap (after post-fader amp,
  but whether before or after pan is not critical here — the AUX bus
  is mono anyway; if taken post-pan, one of L/R or their sum is used;
  by default treat as mono post-fader signal). Its wiper feeds
  **AUX n POST** bus.

Result: each AUX knob simultaneously drives a pre-fader and a
post-fader mono bus. 4 knobs × 2 gangs = 8 mono AUX buses.

**Important**: on the mono channel, both gangs receive a mono source.
AUX routing is always mono on this channel — the post-pan stereo
split is irrelevant to the AUX sends.

### (d) CUE dual-gang pot (stereo send)

Similar to an AUX send, but:

- Both gangs are tapped **pre-fader** (mono signal).
- The two gangs feed CUE L and CUE R of the stereo CUE bus.
- Because the source is mono, the same signal goes to both L and R
  (essentially a mono pan-center contribution into the stereo CUE
  bus).

### (e) Post-Fader Output

Dedicated jack. The post-fader mono signal (taken after the post-fader
amp, before pan — so still mono) is sent to an impedance-balanced TRS
jack at the rear.

*(Note: the PFL switch was previously listed here as item (e). It
has been moved to §2.5.2 because the design now taps PFL pre-mute,
not post-mute.)*

---

## 2.10 Implementation details

Status: **in-progress** — Block 1 (input stage to CHANNEL SOURCE
switch), Block 2 (HPF + Insert Send/Return + jack PCB), and Block 3
(meter buffer + PFL switch + MUTE) finalized. Remaining stages
(fader, post-fader amp, active pan, post-pan distribution) still
conceptual.

### Block 1 — Input stage, buffers, Direct Out, balanced receivers, CHANNEL SOURCE switch (FINALIZED)

**Status:** finalized
**Last updated:** 2026-04-24
**Conversation ref:** "mono channel — input stage to CHANNEL SOURCE switch"

#### Topology chosen

*Input EMI filter* (× 2 inputs per channel: A and B, each with hot +
cold):

- T-balanced double-pi filter per global standard
  (see `00-conventions.md`).

*Buffer stage:*

- One **OPA1679** (quad SOIC) per channel serves all 4 buffers (A-hot,
  A-cold, B-hot, B-cold), each wired as unity-gain voltage follower.
- Input DC block: 47 µF bipolar electrolytic in series, 100 kΩ to AGND.
- The 4 buffer outputs leave the input PCB via a 5-pin connector
  (4 signals + AGND centered).

*Direct Out* (Input A only, TRS rear-panel jack):

- Tap from the two buffer outputs of Input A (hot and cold), driven
  actively on both lines.
- Same T-balanced EMI filter topology as the input
  (see `00-conventions.md`).
- Note: the Direct Out is "fully-balanced" only if the source driving
  the TRS input is itself balanced; if the source is
  impedance-balanced or single-ended, the Direct Out preserves that
  same character. The §2.1 wording "full-differential" should be read
  as "both lines are actively driven by a buffer", not as "a
  differential feedback forces hot = −cold".

*Balanced receivers — "All Inverting" topology* (Jung/Kester Fig. 6-26,
Op Amp Applications Handbook):

- Two receivers per channel, one for Input A, one for Input B.
- Each receiver uses 2 opamps → 4 opamps per channel → one **OPA1679**
  (quad SOIC) hosts both receivers.
- **U1** (first opamp of the pair): inverting unity-gain inverter.
  - Input: hot line via 1 kΩ to inverting input; non-inv to AGND.
  - Feedback: 1 kΩ.
  - Output: −Vhot.
- **U2** (second opamp of the pair): inverting summing amplifier.
  - Inputs: −Vhot (from U1) via 1 kΩ, and cold line via 1 kΩ, both to
    inverting input; non-inv to AGND.
  - Feedback: 100 Ω (fixed minimum) in series with 10 kΩ log pot,
    used as a 3-terminal resistor (wiper + unused end strapped
    together for fail-safe behavior on wiper discontinuity).
  - Output: +(Vhot − Vcold) × Rf / 1 kΩ.
- Gain range per receiver: **G = 0.1 (−20 dB) to 10.1 (+20 dB)**,
  ≈ 40 dB total, pseudo-log taper.
- CMR depends only on matching of the four 1 kΩ resistors per
  receiver. The feedback Rf (pot) does **not** affect CMR — a defining
  property of this topology that justifies using a pot for gain
  control.

*Post-receiver DC block* (per receiver output, × 2 per channel):

- 47 µF bipolar electrolytic in series.
- 47 kΩ to AGND (bias path for the switch pole + discharge path for
  the unselected branch, avoiding click on A↔B switching).
- fc ≈ 0.072 Hz — effectively just a DC block in audio band.

*CHANNEL SOURCE switch:*

- DPDT mechanical, front-panel. See §2.2.
- Section 1 selects A or B audio signal onto the channel path.
- Section 2 switches +5V to A-LED (red) or B-LED (green) on DGND
  return.

#### Active devices

**OPA1679 × 2 per channel:**

- 1× for buffers (4 opamps used).
- 1× for balanced receivers (4 opamps used).

#### Key passive values

- Receiver matched resistors: 1 kΩ × 4 per receiver × 2 receivers =
  **8 per channel — CMR-critical**. Target tolerance: **0.1 %
  thin-film**.
- Gain pot: 10 kΩ log × 2 per channel (one per input, independent).
- Pot minimum series: 100 Ω × 2 per channel.

#### Design targets / expected performance

- Zin (balanced, differential): ≈ 200 kΩ.
- Gain range: −20 dB to +20 dB, pot-controlled, log taper.
- CMR: ≥ 60 dB target with 0.1 % matched 1 kΩ.

#### Decisions and tradeoffs

- **Two independent gain pots** (one per Input A, one per Input B)
  instead of a shared stereo pot. Reason: A and B typically have very
  different working levels.
- **All-Inverting receiver over other topologies.** Reason:
  single-resistor gain change with no CMR interaction.
- **No phase-lead compensation on the receivers (first revision).**
  Footprints provided but unpopulated; populate in rev 2 if measured
  10 kHz CMR is insufficient.
- **LED power on +5V / DGND**, galvanically separate from audio
  ±15V / AGND.

---

### Block 2 — HPF + Insert Send/Return + jack PCB (FINALIZED)

**Status:** finalized
**Last updated:** 2026-04-25
**Conversation ref:** "mono channel — HPF through INSERT switch"

#### Topology chosen

*HPF (§2.3):*

- Cascade of a passive 1st-order HPF and an active Sallen-Key
  2nd-order HPF, unity gain. Topology copied from another mixer
  (specific source not recorded; circuit accepted as-is).
- Passive 1st-order: C39 (220 nF) in series, R23 (5.1 kΩ) to AGND.
  fc ≈ 142 Hz.
- Active Sallen-Key 2nd-order: C40 (220 nF) in series with R24
  (1.3 kΩ) bootstrap to opamp output, C41 (220 nF) to opamp non-inv,
  R25 (62 kΩ) from non-inv to AGND. Opamp wired as unity-gain
  follower (inv pin directly connected to output).
  - f0 ≈ 80.6 Hz, **Q ≈ 3.46**.
- Output DC block: C42 (47 µF bipolar) in series, R26 (47 kΩ) to AGND.
- Opamp: **OPA1642 IC2B** (dual SOIC). The other half (IC2A) is used
  by the Insert Return receiver.
- HPF stage is **always powered**. The DPDT switch only selects which
  signal continues downstream (unfiltered or filtered).

*HPF switch:*

- DPDT mechanical, front-panel.
- Section 1: alternates between bypass (signal from CHANNEL SOURCE
  post-DC-block) and HPF output (post-C42).
- Section 2: drives **orange LED**, on when HPF IN, +5V / DGND.

*Insert Send (§2.4):*

- Branch tap taken at the HPF output side (post-C42, post DC-block),
  upstream of the INSERT DPDT switch. The Send is therefore always
  active regardless of INSERT switch position.
- The branch goes through 75 Ω in series → pin 4 of the 5-pin
  connector to the jack PCB.
- The cold dummy network (built on the channel PCB, on pin 5 of the
  connector): 75 Ω in series, followed by a node with 47 µF to AGND
  in parallel with 47 kΩ to AGND. Per the global default
  (see `00-conventions.md`).

*Insert Return receiver:*

- Classic single-opamp differential receiver, 4-resistor topology.
- Hot line: C34 (47 µF bipolar, DC block) → R18 (10 kΩ) → opamp inv
  pin.
- Cold line: C35 (47 µF bipolar, DC block) → R19 (10 kΩ) → opamp
  non-inv pin.
- Feedback: R20 (10 kΩ) from output to inv. R21 (10 kΩ) from non-inv
  to AGND.
- HF compensation: **C36 = C37 = 100 pF NPO ±5%**, in parallel with
  R20 (feedback) and R21 (non-inv to AGND) respectively. Symmetric
  placement preserves CMR at HF. fc of HF rolloff ≈ 159 kHz — RF
  filter only, zero impact in audio band.
- Output DC block: C38 (47 µF bipolar) in series, R22 (47 kΩ) to AGND.
- Gain: **unity, fixed**.
- Opamp: **OPA1642 IC2A** (the other half of the HPF opamp).

*INSERT switch:*

- DPDT mechanical, front-panel.
- Section 1: alternates onward signal between the pre-insert tap
  (post-HPF, branch upstream of the Send 75 Ω) and the Return
  receiver output (post-C38).
- Section 2: drives **blue LED**, on when INSERT IN, +5V / DGND.

*5-pin connector — channel PCB ↔ jack PCB:*

- Pin 1: Return hot (from jack PCB to channel PCB receiver).
- Pin 2: Return cold (from jack PCB to channel PCB receiver).
- Pin 3: AGND. Travels in the cable as a guard between Send and
  Return pin pairs to reduce intra-cable crosstalk. **On the jack
  PCB, pin 3 lands on an isolated test point** and does NOT propagate
  into any ground pour on the jack PCB (the jack PCB has only a
  chassis pour, no AGND pour).
- Pin 4: Send hot (from channel PCB, post 75 Ω, to jack PCB).
- Pin 5: Send cold (from channel PCB cold dummy network, to jack
  PCB).

*Jack PCB:*

- Two TRS jacks (Send and Return).
- Standard T-balanced double-pi EMI filter on tip and ring of each
  jack (see `00-conventions.md`).
- Sleeve of each TRS jack connected directly to the chassis pour
  (Muncy Pin 1 rule, see `00-conventions.md`).
- No AGND plane / pour on this PCB. Only chassis.

#### Active devices

**OPA1642 × 1 per channel** (dual SOIC):

- IC2A: Insert Return receiver.
- IC2B: HPF (Sallen-Key + passive 1st-order).
- Rationale: shared between two functions, low part count.

#### Key passive values

- HPF caps: C39 / C40 / C41 = 220 nF (film, low DA — final dielectric
  TBD based on physical size). C42 = 47 µF bipolar electrolytic.
- HPF resistors: R23 = 5.1 kΩ, R24 = 1.3 kΩ, R25 = 62 kΩ, R26 = 47 kΩ.
- Insert Return receiver matched resistors:
  R18 = R19 = R20 = R21 = 10 kΩ × 1 receiver per channel = **4 per
  channel — CMR-critical**. Target tolerance: **0.1 % thin-film**.
- Insert Return HF compensation: C36 = C37 = 100 pF NPO ±5%.
- Insert Return DC blocks: C34, C35, C38 = 47 µF bipolar electrolytic.
- Insert Return bias paths: R22 = 47 kΩ.
- Insert Send build-out: 75 Ω (tip series); cold dummy = 75 Ω + 47 µF
  to AGND + 47 kΩ to AGND.
- HPF / INSERT LEDs: standard low-current parts on +5V / DGND
  (orange and blue).

#### Design targets / expected performance

- HPF response: 3rd-order overall (1st passive + 2nd active SK), with
  a resonant peak of approximately +5 dB around 80 Hz before rolloff
  (due to Q ≈ 3.46 on the SK stage). Not Butterworth — see open
  issues.
- Insert Return receiver: unity gain, CMR limited by matching of the
  four 10 kΩ resistors. With 0.1 % thin-film, CMR ≈ 54 dB worst case.
- HF rolloff of receiver: fc ≈ 159 kHz — transparent in audio band,
  attenuates RF residue.

#### Decisions and tradeoffs

- **HPF topology copied from existing mixer**, accepted as-is. Q ≈
  3.46 on the SK stage produces a ~+5 dB peak around 80 Hz. Not yet
  decided whether this is intentional voicing or should be rebalanced
  toward Butterworth (see `10-open-tbd.md`).
- **OPA1642 dual shared between HPF and Insert Return receiver.**
  Single chip serves both functions, reducing part count.
- **Single-opamp differential receiver on the Insert Return** (rather
  than the All-Inverting topology used on the input). Reason: simpler,
  fixed unity gain (no need for the All-Inverting's CMR-vs-gain
  decoupling property), one less opamp, the Insert Return is a less
  critical input than the channel input.
- **Insert Return: unity gain fixed**, no trim. Coherent with §2.4
  "fixed gain" wording. If a trim is needed in future revisions, the
  topology supports adding it on the feedback path.
- **HF compensation caps C36 = C37 = 100 pF NPO ±5%**: chosen to
  filter only RF residue (fc ≈ 159 kHz), no impact on audio band.
  Symmetric placement preserves HF CMR.
- **Send tap upstream of INSERT switch**: Send is always active, even
  when INSERT switch is OUT. This matches §2.4 wording.
- **Send cold dummy network placed on the channel PCB**, not on the
  jack PCB. Reason: keeps the jack PCB purely passive (only TRS jacks
  + EMI filters), and consolidates the active-side electronics on one
  board.
- **5-pin connector with pin 3 as AGND guard trace** (option A,
  decided in conversation): AGND travels in the cable to reduce
  intra-cable crosstalk between Send and Return, but lands on an
  isolated test point on the jack PCB so the jack PCB remains
  chassis-only. No ground loop possible between the two PCBs through
  this connector.
- **No EMI filter on the channel-side end of the 5-pin connector** —
  EMI filtering is concentrated on the jack PCB at the TRS jacks
  themselves, which is where RFI actually couples in.

#### Open issues for Block 2

- **HPF voicing**: Q ≈ 3.46 is high (resonant peak ~+5 dB at 80 Hz).
  To revisit later: confirm intentional voicing or rebalance toward
  Butterworth (Q ≈ 0.7, R24 → ~31 kΩ). See `10-open-tbd.md`.
- **Send cold dummy network impedance balance at low frequencies**:
  the 47 µF in the dummy has Xc ~169 Ω at 20 Hz, so Z_tip ≠ Z_ring
  below ~100 Hz. CMR of the downstream receiver degrades at low
  frequencies — exactly where mains hum lives. Possible mitigation:
  larger dummy cap (220–470 µF) or alternative topology. Default
  pattern in `00-conventions.md` reflects the current choice; revise
  if measured hum rejection on Insert is insufficient. See
  `10-open-tbd.md`.
- HPF film cap dielectric (PPS / PET / PP / NPO depending on physical
  size at 220 nF) — TBD.
- HPF switch and INSERT switch part selection (DPDT, front-panel,
  current rating adequate for LED section).
- 75 Ω resistor part / package — non-critical, 0603 1 % standard.

---

### Block 3 — Meter buffer + PFL switch + MUTE (FINALIZED)

**Status:** finalized
**Last updated:** 2026-04-26
**Conversation ref:** "mono channel — meter buffer through PRE-FADER node"

#### Topology chosen

*Meter buffer (§2.5.1):*

- Single opamp wired as unity-gain follower. Input is the
  high-impedance tap directly on the post-INSERT node (signal is
  already DC-free thanks to C42 / C38 upstream, so no input
  DC-block on the buffer).
- Output: 75 Ω in series (build-out, for stability into cable
  capacitance) → 2-pin connector to the meter bridge PCB.
- Connector pinout: pin 1 = signal (post-build-out), pin 2 = AGND.
  Single-ended unbalanced.
- Meter bridge PCB carries a full **AGND pour** (it has no TRS jacks
  whose sleeve would otherwise inject chassis currents into AGND —
  the chassis-only pattern of the jack PCB does not apply here).
  A separate DGND pour on the meter bridge serves the LEDs per §00.

*PFL switch (§2.5.2):*

- 22 kΩ in series with the post-INSERT signal feeds the pole of a
  DPDT mechanical front-panel switch.
- Section 1 (audio): throws between AGND (PFL OFF) and the PFL mono
  bus (PFL ON). The 22 kΩ is the bus summing resistor for this
  channel's PFL contribution. Constant load on the upstream node in
  both positions → no DC step on switching → click-free.
- Section 2 (control): pole = +5 V (LED rail). In PFL ON, throws to
  a junction connecting (a) the anode of the **red front-panel PFL
  LED** (cathode to DGND via current-limit resistor) and (b) the
  anode of a small-signal diode whose cathode goes to the **PFL
  DETECT logical bus** (wired-OR HIGH ⇒ at least one PFL active
  anywhere on the console). The diode prevents back-feed of +5 V
  from one channel's Sec2 to another's LED via the bus.

*MUTE switch (§2.6):*

- **ADG419** (single SPDT analog switch, SOIC-8). Used as
  SPST-series: S1 receives the audio signal, D drives the
  PRE-FADER node, S2 left unconnected.
- Power: VDD = +15 V, VSS = −15 V, with 100 nF ceramic bypass on
  each rail per §00.
- VL = +5 V (LED logic rail).
- GND pin → **DGND** (referenced to the firmware-side ground that
  generates the IN control signal — keeps the IN threshold stable
  against any HF residue between AGND and DGND). This is a
  **block-local choice, not promoted to a global convention**; the
  GND-pin connection of other CMOS analog switches in the console
  will be decided case-by-case.
- Logic IN: comes from a connector. Partitioning between
  per-channel and per-PCB control connectors is deferred until
  channels-per-PCB is decided (likely 4 or 6 channels per input
  PCB). See `10-open-tbd.md`.
- Logic semantics: IN = 1 ⇒ S2 selected ⇒ D in high-Z ⇒ channel
  muted. The downstream PRE-FADER node is then pulled toward AGND
  by the ensemble of bias paths and virtual-earth returns of the
  CUE / AUX gang 1 / fader stages that load it (to be detailed in
  Block 4).

#### Active devices

- **Meter buffer opamp**: TBD. Candidates: another OPA1642, or an
  audio-grade dual where the second half can be reused for a future
  stage in Block 4.
- **ADG419** × 1 per channel, SOIC-8.
- **Small-signal diode** for the PFL DETECT wired-OR (1N4148-class):
  exact part TBD.

#### Key passive values

- Meter buffer build-out: **75 Ω** (0603 1 %).
- PFL series / summing resistor: **22 kΩ** (0603 1 %).
- ADG419 supply bypass: 100 nF × 2 (one per rail) per §00.
- PFL LED current-limit resistor: TBD (depends on chosen LED Vf
  and desired brightness).

#### Design targets / expected performance

- Meter buffer: unity gain DC–audio band, output capable of driving
  ≈100 pF cable capacitance via the 75 Ω build-out without ringing.
- ADG419 Ron: ≈17 Ω typical at ±15 V supplies. THD well below
  audible at audio levels. Charge injection ≈5 pC typical.
- Mute attenuation: limited by ADG419 off-isolation (≈80 dB at
  1 kHz) — adequate for any practical channel mute.

#### Decisions and tradeoffs

- **Meter detection chain on a separate PCB**: pattern matches the
  jack PCB and input PCB partitioning. Allows the meter LED bridge
  to be a separate mechanical sub-assembly (the visual "meter
  bridge" at the top of a console). One mono signal per channel via
  a 2-pin connector.
- **Unbalanced send to meter PCB** rather than impedance-balanced
  or fully balanced: cable run is short and internal, destination
  is a rectifier (not a hi-fi amp), saves pins and parts.
- **Meter PCB AGND pour, not chassis-only**: no TRS jacks present
  on the meter PCB, so no Muncy "Pin 1" rationale to keep AGND
  isolated from chassis on that board.
- **PFL is pre-fader and pre-mute**, achieved by tapping upstream of
  the ADG419. PFL listen continues to work even when the channel
  is muted, matching live-sound semantics. Conceptually relocated
  out of §2.9 (post-pan distribution) into §2.5 (pre-mute taps).
- **22 kΩ + DPDT pattern for PFL** rather than a buffered tap:
  shared-bus summing where every active PFL contribution adds via
  its 22 kΩ at the PFL mono bus virtual-earth. The constant load on
  the upstream node (in both OFF and ON, the 22 kΩ sits between
  signal and 0 V) means no DC step on switching → click-free.
- **PFL DETECT as a separate logical bus**, fed via per-channel
  diode wired-OR from Sec2 of the DPDT. Avoids dragging analog PFL
  audio into firmware. The diode prevents inter-channel back-feed
  through the bus.
- **ADG419 as SPST-series** (S2 NC), not series-shunt T or
  signal/AGND-alternating SPDT. Simplest acceptable topology;
  ≈80 dB off-isolation is sufficient for channel mute. Series-shunt
  T would require a second switch for marginal improvement.
- **ADG419 GND pin → DGND** (block-local, not a global convention).
  Rationale: the IN threshold is referenced to the GND pin; tying
  it to the same ground as the firmware that drives IN keeps the
  threshold stable against any HF residue between AGND and DGND.
  The audio path (S1/D) stays referenced to VDD/VSS, untouched by
  this choice. The user has explicitly elected to revisit this
  case-by-case for other CMOS switches rather than promote it to
  §00.

#### Open issues for Block 3

- **Meter buffer opamp** — part TBD.
- **Channels per input PCB** (4 or 6) and resulting **control
  connector partitioning** (one wide connector per PCB vs one
  narrow connector per channel) — TBD. See `10-open-tbd.md`.
- **Other PFL sources feeding PFL DETECT**: AUX masters (§05),
  AUX returns (§03), and groups (§04) all carry PFL controls; do
  they all need to feed PFL DETECT to drive the firmware
  source-select? — TBD. See `10-open-tbd.md`.
- **PFL LED current-limit resistor value** — depends on chosen LED.
- **PFL signal diode** — 1N4148-class, exact part TBD.
