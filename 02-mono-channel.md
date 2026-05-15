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

**Channel Output** (TRS jack on input PCB): its default output is the
full-differential signal from the two Input A buffers (before the
unbalancing stage). A firmware-controlled relay (**Channel Output SELECT**,
on the input PCB) can switch this jack to instead output an
impedance-balanced signal from the channel PCB — either pre-MUTE or
post-fader, depending on the **Output PRE-POST** mechanical switch on
the channel strip (see §2.9 (e)).

Note: in the Input-A state (RESET / default), the Channel Output is fully
differential (both lines actively driven from Input A buffers) —
not merely impedance-balanced. In the Channel-signal state (SET), the
output is impedance-balanced (hot = channel PRE/POST through 75 Ω;
cold = 75 Ω dummy to AGND). See Block 1 and Block 8 for implementation
details.

---

## 2.2 CHANNEL SOURCE switch (A / B select)

A **relay-driven electronic switch** (DPDT 2 form C, **1-coil
latching**, standard signal relay per `00-conventions.md`) selects
which of the two unbalanced-and-gained signals continues down the
channel. Only one at a time. Named **CHANNEL SOURCE** to avoid
confusion with the master Monitor section (`08-monitor.md`).

The front-panel control is a **momentary pushbutton** (single
contact). Firmware reads each press and issues either a set or
reset pulse (≥ 10 ms) to the relay coil, toggling the relay's
mechanically latched state. The relay's two form-C contact sets
both carry audio signals:

- **Contact set 1** — switches the **active** audio signal (A or B
  onto the channel path). NC = Receiver A (RESET default); NO =
  Receiver B.
- **Contact set 2** — switches the **inactive** receiver output to
  the meter buffer (see §2.5.1 and Block 10). Wired complementary
  to set 1: NC2 = Receiver B; NO2 = Receiver A; COM2 = inactive
  meter buffer input. In RESET state (A active on channel), set 2
  delivers Receiver B to the meter; in SET state (B active on
  channel), set 2 delivers Receiver A.

Front-panel indicator LEDs (A-LED red, B-LED green) are driven by
**MCU GPIO outputs** — firmware knows the relay state because it
commands it, and updates the LEDs on each CHANNEL SOURCE toggle.
Boot default: A-LED on, B-LED off (A selected, RESET state).

---

## 2.3 HPF / INSERT chain

The HPF and INSERT stages form a switchable chain that can be bypassed
as a unit by the **HPF/INSERT BYPASS relay** (§2.3.1 below). The
individual HPF switch (§2.3.2) and INSERT switch (§2.3.3) remain and
operate independently within the chain.

### 2.3.1 HPF / INSERT bypass relay

A **DPDT relay** (standard AGQ210A03, firmware-controlled, per
`00-conventions.md`) is interposed between the CHANNEL SOURCE relay
output and the rest of the channel. It either routes the signal through
the HPF / INSERT chain (NO contacts) or bypasses it entirely
(NC contacts):

- **COM1** — CHANNEL SOURCE relay output.
- **NC1 ↔ NC2** — hard-wired together. In the RESET / NC state the
  signal passes COM1 → NC1 → NC2 → COM2, bypassing the chain.
- **COM2** — the pre-MUTE node: feeds the meter buffer, PFL tap, and
  MUTE relay COM1 (see §2.5 and §2.6). Also the PRE tap for the Output
  PRE-POST switch (see §2.9 (e)).
- **NO1** — input of the HPF switch (§2.3.2 Section 1). Chain entry
  point.
- **NO2** — output of the Insert Return receiver (post-C38, §2.3.3).
  Chain exit point.

In the SET / NO state the signal path is: COM1 → NO1 → [HPF switch →
HPF → INSERT switch → Insert Return] → NO2 → COM2. The individual
switches within the chain remain fully functional.

**Front-panel control:** a momentary **pushbutton** (MCU GPIO input)
cycles through three modes, each indicated by one of three separate
**orange LEDs** (MCU GPIO outputs):

| LED | Mode | Relay behaviour |
|---|---|---|
| FOLLOW PATH (center) | Chain always in path | Relay always SET (NO) |
| FOLLOW A | Chain follows source A | SET when CHANNEL SOURCE = A; RESET when B |
| FOLLOW B | Chain follows source B | SET when CHANNEL SOURCE = B; RESET when A |

Mode cycling logic and boot state are firmware decisions — TBD, see
`10-open-tbd.md`.

### 2.3.2 HPF

Switchable **fixed-frequency high-pass filter** with a characteristic
voicing (resonant peak before the rolloff, see §2.10 Block 2). The HPF
stage is always powered; a DPDT mechanical switch (front-panel) selects
between the unfiltered and filtered signal path. The DPDT's second
section drives an **orange LED** indicating HPF IN.

### 2.3.3 Insert

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

The pre-MUTE mono signal (output of the HPF/INSERT bypass relay, COM2)
feeds two parallel taps before reaching the MUTE switch.

### 2.5.1 Meter buffers (dual bargraph)

The channel feeds **two bargraph signals** to the meter bridge PCB:

- **Active meter** (full brightness): taps the pre-MUTE node
  (HPF/INSERT bypass relay COM2). Shows the signal currently in the
  channel path, regardless of CHANNEL SOURCE selection.
- **Inactive meter** (dimmed on the meter bridge): taps whichever
  receiver output (A or B) is NOT currently selected by CHANNEL
  SOURCE. A CMOS SPDT switch selects between Receiver A and Receiver B
  outputs dynamically.

Both signals are buffered by unity-gain opamps on the channel PCB
and sent via a 3-pin connector to the meter bridge PCB (rectifier,
log amp, LED driver ICs, and meter LEDs all on the meter bridge, not
on the channel PCB). Neither tap interrupts the main path. Dimming
of the inactive bargraph is a meter bridge design decision.

### 2.5.2 PFL switch

A pre-fader, **pre-mute** tap to the PFL mono bus. The signal
encounters a series resistance and then the pole of a DPDT
mechanical front-panel switch.

- **Section 1** (audio): alternates the resistor's far end between
  AGND (PFL OFF) and the PFL mono bus (PFL ON). The upstream node
  sees a fixed light load identical in OFF and ON, so no DC step on
  switching → click-free.
- **Section 2** (control): pole = +3.3 V (LED rail per §00). In
  PFL ON, drives a **red front-panel LED** AND contributes via a
  wired-OR diode to the **PFL DETECT logical bus** that firmware
  monitors to drive the master monitor source-select (§8.1).

PFL listen therefore continues to work even when the channel is
muted by the MUTE switch downstream — coherent live-sound semantics
("pre-fader listen always tells you what is at the channel
input").

---

## 2.6 ACTIVE / SOLO electronic switch (MUTE)

An **electronic switch** (standard latching signal relay per
`00-conventions.md`) controlled by digital logic interrupts the main
signal path. Configured as **series + shunt**: one contact set opens
the series path when muted; the other simultaneously shunts the
PRE-FADER node to AGND. Referred to as the **MUTE relay**.

This relay is commanded by three sources:

- **ACTIVE/MUTE button** on this channel (Pattern C, orange LED —
  see §2.11): toggles this channel between ACTIVE (relay RESET) and
  MUTED (relay SET).
- **SOLO logic** from any other channel: when another channel's
  SOLO is engaged in SIP (Solo In Place) mode, firmware drives
  this relay to SET (muted). Restored when the SOLO is cleared.
- **Global MUTE ALL** command from firmware (e.g. master button).

The output of this switch is the **PRE-FADER node** — source of all
post-mute pre-fader taps (CUE, AUX gang 1) and of the fader itself.

---

## 2.7 Fader + post-fader amp

- **Fader** — standard linear or log-audio mono fader.
- **Post-fader amp** — +10 dB fixed gain to recover headroom after
  the fader attenuates.

---

## 2.8 Active pan

**Active pan** circuit driven by a **dual-gang pot** (two gangs on
one shaft, reverse-wired with respect to each other so that opposite
ends of the two pot tracks see the input signal). The channel
**becomes stereo from this point on**: two lines (L and R) travel
together for the rest of the channel.

---

## 2.9 Post-pan distribution

From the post-pan stereo signal, several destinations tap in parallel:

### (a) 2-gang × 4-position rotary switch — main destination

Routes the post-pan stereo signal to exactly one of:

- Main Mix
- Group 1
- Group 2
- Group 3

Mechanical rotary, 2 gangs (one for L, one for R), 4 positions. The
inactive positions are open (not back-grounded — see Block 6 for
rationale). Only one destination at a time.

### (b) AFL electronic switch

Parallel tap, stereo. A relay-driven electronic switch (one
standard latching signal relay per channel per `00-conventions.md`
— DPDT 2 form C, 1-coil latching) connects the channel's stereo
post-pan signal to the AFL bus when the AFL logic calls for it.
Contact set 1 carries L, contact set 2 carries R; both ganged on
the same coil so L and R are perfectly synchronous.

*Tap location and reset-state direction are now fixed: tap is at the
post-pan opamp output (Block 5), through per-side bus summing resistors
(value TBD) to COM; NC contacts connect to AGND (constant termination
when AFL off — prevents impedance steps on switching, click-free); NO
contacts connect to AFL Left / AFL Right bus. Summing-resistor values
deferred to §08 — see `10-open-tbd.md`. See Block 7 for implementation
details.*

### (c) 4 × AUX dual-gang pots (mono sends)

Four knobs, each a dual-gang pot:

- **Gang 1** is fed by the **PRE-FADER node** (mono, after the
  ACTIVE/SOLO switch but before the fader). Its wiper feeds
  **AUX n PRE** bus via a bus summing resistor.
- **Gang 2** is fed by the **POST-FADER node** (mono, after the
  post-fader amp, before the pan). Its wiper feeds **AUX n POST**
  bus via a bus summing resistor.

Result: each AUX knob simultaneously drives a pre-fader and a
post-fader mono bus. 4 knobs × 2 gangs = 8 mono AUX buses.

**Note on channel-as-source vs. mono AUX bus**: the AUX sends are
taken before the pan stage. The pan would split L/R at constant
power, but since AUX buses are mono, splitting and re-summing would
just give the source back; tapping pre-pan saves the operation.

### (d) CUE dual-gang pot (stereo send)

Similar to an AUX send, but:

- Both gangs are tapped **pre-fader** (mono signal).
- The two gangs feed CUE L and CUE R of the stereo CUE bus.
- Because the source is mono, the same signal goes to both L and R
  (essentially a mono pan-center contribution into the stereo CUE
  bus).

### (e) PRE-POST output tap and Output PRE-POST switch

Two signal taps are taken from the channel PCB for routing to the input
PCB (to feed the switchable Channel Output — see §2.1):

- **PRE tap**: pre-MUTE signal (output of the HPF/INSERT bypass relay,
  COM2), through a **75 Ω** series resistor (branch tap; not in series
  with the main signal path).
- **POST tap**: POST-FADER node (after post-fader amp, before pan),
  through a **75 Ω** series resistor (branch tap).

A front-panel **DPDT mechanical switch** named **Output PRE-POST**
selects which of the two 75 Ω outputs continues to the input PCB:

- Section 1 (signal): common feeds the 2-pin connector (below);
  throws select PRE or POST 75 Ω tap output.
- Section 2 (indicator): drives two front-panel LEDs — one per
  position — from +3.3V / DGND per `00-conventions.md`. Colors TBD.

A **2-pin connector** carries the selected signal from the channel PCB
to the input PCB:

- Pin 1: the selected signal (hot, already through the 75 Ω build-out).
- Pin 2: a dedicated **75 Ω to AGND** cold dummy (provides the cold
  leg of an impedance-balanced link; full cold dummy network details
  per `00-conventions.md` standard — 47 µF ‖ 47 kΩ — TBD,
  see `10-open-tbd.md`).

On the input PCB, these lines feed the **Channel Output SELECT** relay
(Block 8), which alternates the Channel Output TRS jack between Input A
buffers and this channel signal.

*See Block 8 for implementation details.*

*(Note: the PFL switch was previously listed here as item (e). It
has been moved to §2.5.2 because the design now taps PFL pre-mute,
not post-mute.)*

---

## 2.10 Implementation details

Status: **in-progress** — Block 1 (input stage to CHANNEL SOURCE
switch + Channel Output SELECT relay on input PCB) FINALIZED. Block 2
(HPF + Insert Send/Return + jack PCB — now in the NO loop of Block 9),
Block 3 (meter buffer + PFL switch + MUTE, now series+shunt), Block 4
(pre-fader node + fader PCB + post-fader amp + AUX/CUE sends), and
Block 5 (active pan) finalized. Block 6 (post-pan routing rotary)
topology defined, values TBD. Block 7 (AFL switch) finalized — topology
fixed, summing resistor values deferred to §08. Block 8 (Output
PRE-POST switch + switchable Channel Output) in-progress — see
`10-open-tbd.md`. Block 9 (HPF/INSERT chain bypass relay) in-progress
— relay wiring established, logical modes and boot state TBD.
Block 10 (dual meter buffers: TL072 dual buffer, CHANNEL SOURCE relay
contact set 2 for inactive source selection) FINALIZED.

### Block 1 — Input stage, buffers, Channel Output, balanced receivers, CHANNEL SOURCE switch (FINALIZED)

**Status:** finalized

#### Topology chosen

*Input EMI filter* (× 2 inputs per channel: A and B, each with hot +
cold):

- T-balanced double-pi filter per global standard
  (see `00-conventions.md`).

*Input jack idle-noise scheme:*

- The TRS jacks for Input A and Input B are *stereo with switch*
  type (5 contacts: TIP, RING, SLEEVE, TIP-SW, RING-SW). TIP-SW and
  RING-SW are tied together but otherwise unconnected. When no plug
  is inserted, the jack's mechanical normalling closes TIP-SW onto
  TIP and RING-SW onto RING — therefore TIP↔RING are shorted at the
  jack itself in idle. The All-Inverting receiver downstream sees
  zero differential signal at idle, eliminating differential pickup
  on open inputs. SLEEVE remains permanently to chassis (Pin 1
  rule). With a plug inserted, the SW contacts open and balanced
  operation is fully restored. Common-mode pickup at idle is left to
  the receiver's CMR, deemed sufficient.

*Buffer stage:*

- One **OPA1679** (quad SOIC) per channel serves all 4 buffers (A-hot,
  A-cold, B-hot, B-cold), each wired as unity-gain voltage follower.
- Input DC block: 47 µF bipolar electrolytic in series, 100 kΩ to AGND.
- The 4 buffer outputs leave the input PCB via a 5-pin connector
  (4 signals + AGND centered).

*Channel Output* (Input A only, TRS rear-panel jack):

- Tap from the two buffer outputs of Input A (hot and cold), driven
  actively on both lines.
- **Build-out resistors: 75 Ω 0.1 % thin-film matched pair**, one
  per leg (hot and cold), in series between each buffer output and
  the EMI filter. The build-out is on the Channel Output branch only;
  the internal off-board path to the next PCB (toward the
  All-Inverting receiver) does NOT carry build-out, since the
  receiver wants Z_source ≈ 0 and the two opamps of the same quad
  are intrinsically matched.
- Same T-balanced EMI filter topology as the input
  (see `00-conventions.md`).
- **No DC block** between buffer output and the TRS jack. OPA1679
  V_os ≤ ±1.5 mV / section, differential offset ≤ ~3 mV — negligible
  for any downstream balanced receiver. A series cap on each leg
  would degrade CMR at low frequencies (electrolytic tolerance
  produces ΔX_c that translates to ΔZ_source) exactly where mains
  hum sits. Trade favors omitting the cap.
- Note: the Channel Output is "fully-balanced" only if the source driving
  the TRS input is itself balanced; if the source is
  impedance-balanced or single-ended, the Channel Output preserves that
  same character. The §2.1 wording "full-differential" should be read
  as "both lines are actively driven by a buffer", not as "a
  differential feedback forces hot = −cold".

*Channel Output SELECT relay (§2.1):*

- **AGQ210A03** (DPDT 2 form C, 1-coil latching signal relay, per
  `00-conventions.md`) on the input PCB, interposed between the
  75 Ω build-out resistors and the Channel Output EMI filter.
- **NC / RESET state (Input A — default):** contact set 1 COM →
  Input A hot 75 Ω build-out → EMI filter tip; contact set 2 COM →
  Input A cold 75 Ω build-out → EMI filter ring. Original
  full-differential behavior.
- **NO / SET state (Channel signal):** contact set 1 COM → pin 1
  of 2-pin connector (channel PRE/POST signal) → EMI filter tip;
  contact set 2 COM → pin 2 of 2-pin connector (cold dummy 75 Ω)
  → EMI filter ring. Impedance-balanced behavior.
- See Block 8 for full wiring and decisions.

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

- **AGQ210A03** (DPDT 2 form C, 1-coil latching signal relay,
  standard signal relay per `00-conventions.md`). See §2.2.
- Front-panel control: momentary pushbutton (single contact). Each
  press is read by firmware, which issues a set or reset pulse
  (≥ 10 ms) to the coil's bipolar driver, toggling the latched
  state.
- Contact set 1 (audio — active channel): COM = onward channel
  path; NC = Receiver A output (post-DC-block); NO = Receiver B
  output (post-DC-block). In RESET state, Receiver A is selected
  (firmware-initialized boot default).
- Contact set 2 (audio — inactive meter): COM = inactive meter
  buffer input (TL072 half 2 — see Block 10); NC = Receiver B
  output (post-DC-block); NO = Receiver A output (post-DC-block).
  Wired complementary to set 1: in RESET state (A active), COM2
  delivers Receiver B to the meter; in SET state (B active), COM2
  delivers Receiver A. No additional electronic switch needed.
- A-LED (red) and B-LED (green): MCU GPIO outputs, updated by
  firmware on each CHANNEL SOURCE toggle. Boot default: A-LED on,
  B-LED off.
- Coil drive: bipolar pulse from a 3 V coil supply per
  `00-conventions.md` "Standard signal relay". Driver IC and
  partitioning TBD — see `10-open-tbd.md`.

#### Active devices

**OPA1679 × 2 per channel:**

- 1× for buffers (4 opamps used).
- 1× for balanced receivers (4 opamps used).

**AGQ210A03 × 1 per channel** (DPDT 2 form C, 1-coil latching
signal relay): CHANNEL SOURCE A/B select. Per `00-conventions.md`
"Standard signal relay".

**AGQ210A03 × 1 per channel** (DPDT 2 form C, 1-coil latching
signal relay): Channel Output SELECT (Input A vs channel PRE/POST).
Per `00-conventions.md` "Standard signal relay". See Block 8.

#### Key passive values

- Receiver matched resistors: 1 kΩ × 4 per receiver × 2 receivers =
  **8 per channel — CMR-critical**. Target tolerance: **0.1 %
  thin-film**.
- **Channel Output build-out: 75 Ω × 2 per channel** (hot and cold of
  Input A only). **0.1 % thin-film matched pair**, same lot for
  thermal tracking.
- Gain pot: 10 kΩ log × 2 per channel (one per input, independent).
- Pot minimum series: 100 Ω × 2 per channel.
- A-LED (red) and B-LED (green) current-limit resistors on the
  CHANNEL SOURCE indicator (MCU GPIO driven): TBD (depends on
  chosen LED Vf and brightness target).
- Coil protection (flyback / freewheeling network for bipolar
  drive): exact topology TBD with the coil driver choice.

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
- **OPA1679 (CMOS quad) over the global NE5532 default.**
  Documented exception per `00-conventions.md`. Three reasons combine
  to justify the deviation here:
  (1) buffer count: 4 buffers + 4 receiver opamps per channel = 8
  opamps, naturally fitted by 2 quads.
  (2) the input-side buffer sits at very high source impedance
  (200 kΩ shunt to AGND); a bipolar opamp's bias current (~200 nA
  for NE5532) would drop ~40 mV across the input bias resistor and
  must be matched between hot and cold legs to preserve CMR — the
  CMOS ~10 pA bias current sidesteps this.
  (3) the Channel Output path runs without a DC block (see above);
  OPA1679's low Vos keeps the cumulative offset within budget for
  any downstream ADC differential input.
- **Channel Output build-out: 75 Ω 0.1 % matched pair**, on hot and cold,
  on the Channel Output branch only. Trades a small CMR floor (≈ 96 dB
  with 0.1 % matching, well below the receiver's CMR ceiling) for
  cable-capacitance isolation, short-circuit protection, and proper
  source impedance for the EMI filter — the build-out becomes the
  source impedance against which the π-stage attenuates; without it,
  the filter loses HF attenuation due to near-zero source Z.
  Internal off-board path to the All-Inverting receiver stays at
  Z_source ≈ 0.
- **No DC block on the Channel Output path.** OPA1679 V_os is small
  enough that a series cap is not needed and would actively hurt
  CMR at low frequencies (electrolytic tolerance ⇒ ΔX_c ⇒ ΔZ_source
  exactly where mains hum lives). See *Channel Output* in Topology
  chosen for the numbers.
- **TIP-SW = RING-SW shorting on Input A and B jacks.** Exploits
  the jack's normalling contacts to short TIP↔RING when no plug is
  inserted, so the receiver sees zero differential signal at idle.
  See *Input jack idle-noise scheme* in Topology chosen.
- **LED power on +3.3V / DGND**, galvanically separate from audio
  ±15V / AGND.
- **CHANNEL SOURCE as relay-driven electronic switch** (AGQ210A03
  latching signal relay, both contact sets for audio — set 1 for
  active channel, set 2 for inactive meter — ganged on one coil)
  instead of a direct mechanical DPDT. Primary reason: the relay
  approach is directly compatible with a **global master A/B flip**
  — firmware can toggle every channel's CHANNEL SOURCE relay
  simultaneously, routing all 24 channels from DAW (B) to
  instruments (A) or vice versa in one command. Contact set 2,
  wired complementary to set 1, delivers the inactive receiver to
  the meter buffer with no additional electronic switch. State is
  held mechanically (latch), contact resistance ≤ 100 mΩ initial.
  Boot default = A (RESET state, NC contacts); this is a
  firmware-init contract, not a hardware property. Per
  `00-conventions.md` "Standard signal relay".

---

### Block 2 — HPF + Insert Send/Return + jack PCB (FINALIZED)

**Status:** finalized — this block describes the circuitry within the
NO loop of the HPF/INSERT bypass relay (Block 9). The entry point is
NO1 of that relay; the exit point is NO2.

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
- Opamp: **NE5532 IC2B** (dual SOIC). The other half (IC2A) is used
  by the Insert Return receiver.
- HPF stage is **always powered**. The DPDT switch only selects which
  signal continues downstream (unfiltered or filtered).

*HPF switch:*

- DPDT mechanical, front-panel.
- Section 1: alternates between unfiltered (NO1 of the HPF/INSERT
  bypass relay, Block 9) and HPF output (post-C42).
- Section 2: drives **orange LED**, on when HPF IN, +3.3V / DGND.

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
- Opamp: **NE5532 IC2A** (the other half of the HPF opamp).

*INSERT switch:*

- DPDT mechanical, front-panel.
- Section 1: alternates onward signal between the pre-insert tap
  (post-HPF, branch upstream of the Send 75 Ω) and the Return
  receiver output (post-C38).
- Section 2: drives **blue LED**, on when INSERT IN, +3.3V / DGND.

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

**NE5532 × 1 per channel** (dual SOIC):

- IC2A: Insert Return receiver.
- IC2B: HPF (Sallen-Key + passive 1st-order).
- Rationale: shared between two functions, low part count. Per global
  default in `00-conventions.md`.

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
- HPF / INSERT LEDs: standard low-current parts on +3.3V / DGND
  (orange and blue). **Note**: blue LED Vf is typically
  3.0–3.4 V, leaving very little headroom across the current-limit
  resistor on a +3.3 V rail. Either use a low-Vf blue (~2.7 V) or
  re-evaluate the LED color choice — see `10-open-tbd.md`.

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
- **NE5532 dual shared between HPF and Insert Return receiver.**
  Single chip serves both functions, reducing part count. Per global
  default in `00-conventions.md`. NE5532's CM-distortion at the SK
  stage's high-Z +input is the most visible NE5532 limitation in
  this Block; identified as a candidate for selective upgrade to
  OPA1642 (JFET) at the budget-review stage — see `10-open-tbd.md`.
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

#### Topology chosen

*Meter buffers (§2.5.1):*

- Active meter buffer: unity-gain voltage follower. Input is the
  pre-MUTE node (HPF/INSERT bypass relay COM2). DC-free in both
  relay states: NC path — Block 1 post-receiver DC block; NO path —
  C42 or C38 (Block 2). No input DC block.
- Inactive meter buffer: unity-gain voltage follower. Input is the
  output of a CMOS SPDT switch (DG419) selecting whichever receiver
  output (A or B) is not currently on the channel path.
- Each buffer output: 75 Ω build-out → 3-pin connector to meter
  bridge PCB (pin 1 = active, pin 2 = inactive, pin 3 = AGND).
- Meter bridge PCB carries a full **AGND pour** (no TRS jacks →
  Muncy Pin 1 rule does not apply). Separate DGND pour for LEDs.
- **See Block 10 for full implementation** (DG419, NE5532,
  bypass caps, and connector details).

*PFL switch (§2.5.2):*

- 22 kΩ in series with the pre-MUTE node signal (bypass relay COM2)
  feeds the pole of a DPDT mechanical front-panel switch.
- Section 1 (audio): throws between AGND (PFL OFF) and the PFL mono
  bus (PFL ON). The 22 kΩ is the bus summing resistor for this
  channel's PFL contribution. Constant load on the upstream node in
  both positions → no DC step on switching → click-free.
- Section 2 (control): pole = +3.3 V (LED rail). In PFL ON, throws
  to a junction connecting (a) the anode of the **red front-panel
  PFL LED** (cathode to DGND via current-limit resistor) and
  (b) the anode of a small-signal diode whose cathode goes to the
  **PFL DETECT logical bus** (wired-OR HIGH ⇒ at least one PFL
  active anywhere on the console). The diode prevents back-feed of
  +3.3 V from one channel's Sec2 to another's LED via the bus.

*MUTE switch (§2.6):*

- **AGQ210A03** (DPDT 2 form C, 1-coil latching signal relay,
  standard signal relay per `00-conventions.md`).
- Audio path: **series + shunt configuration** — the two contact
  sets serve different roles (not wired in parallel):
  - **Contact set 1 (series):** COM1 = pre-MUTE node (HPF/INSERT
    bypass relay COM2 — taken upstream of the meter buffer / PFL
    tap node); NO1 = PRE-FADER node onward; NC1 = floating.
  - **Contact set 2 (shunt):** COM2 = PRE-FADER node; NC2 = AGND;
    NO2 = floating.
- **ACTIVE state (SET):** series contact closes (signal passes from
  post-INSERT to PRE-FADER); shunt contact opens (PRE-FADER not
  shunted to AGND). Normal signal flow.
- **MUTE state (RESET):** series contact opens (signal
  interrupted); shunt contact closes (PRE-FADER shunted to AGND).
  Both mechanisms act simultaneously on the single coil.
- Series contact R_on: ≈ 100 mΩ initial (single contact set).
- Coil drive: bipolar pulse from a 3 V coil supply, per
  `00-conventions.md` "Standard signal relay". Driver IC and
  partitioning TBD — see `10-open-tbd.md`.
- Logic semantics: **SET state ⇒ channel ACTIVE** (series contact
  makes, signal passes; shunt contact opens, PRE-FADER free);
  **RESET state ⇒ channel MUTED** (series contact opens, signal
  interrupted; shunt contact makes, PRE-FADER shunted to AGND).
  The relay holds its state mechanically; firmware initializes
  every MUTE relay to RESET at startup. Master-output hard-mute
  keeps the room silent until firmware completes init (master
  section, TBD).

#### Active devices

- **Meter buffer opamps**: **NE5532** (both halves used) — full
  implementation in Block 10.
- **AGQ210A03** × 1 per channel (DPDT 2 form C, 1-coil latching
  signal relay), used as the MUTE switch. Per `00-conventions.md`
  "Standard signal relay".
- **Small-signal diode** for the PFL DETECT wired-OR (1N4148-class):
  exact part TBD.
- **Coil protection** (flyback / freewheeling network for bipolar
  drive): exact topology TBD with the coil driver choice. Typically
  integrated into the chosen latching-relay driver IC.

#### Key passive values

- Meter buffer build-out: **75 Ω × 2** (0603 1 %) — see Block 10.
- PFL series / summing resistor: **22 kΩ** (0603 1 %).
- PFL LED current-limit resistor: TBD (depends on chosen LED Vf
  and desired brightness).
- Coil supply: directly from the +3.3 V logic rail per
  `00-conventions.md` "Standard signal relay" — no series
  dropping network needed.

#### Design targets / expected performance

- Meter buffer: unity gain DC–audio band, output capable of driving
  ≈100 pF cable capacitance via the 75 Ω build-out without ringing.
- MUTE switch contact resistance (series branch): ≤ 100 mΩ initial
  (single contact set). Effectively zero THD contribution (purely
  resistive, voltage-independent), no charge injection.
- MUTE off-isolation: open-contact isolation > 80 dB at 1 MHz per
  datasheet, effectively unbounded at audio. Far in excess of any
  practical channel mute requirement.
- MUTE operate / release time: ≤ 3 ms each per datasheet (no
  bounce specified beyond this figure). Adequate for hard-mute
  semantics; not used for fast gating.

#### Decisions and tradeoffs

- **Meter detection chain on a separate PCB**: pattern matches the
  jack PCB and input PCB partitioning. Allows the meter LED bridge
  to be a separate mechanical sub-assembly (the visual "meter
  bridge" at the top of a console). Two signals per channel (active
  + inactive) via a 3-pin connector.
- **Unbalanced send to meter PCB** rather than impedance-balanced
  or fully balanced: cable run is short and internal, destination
  is a rectifier (not a hi-fi amp), saves pins and parts.
- **Meter PCB AGND pour, not chassis-only**: no TRS jacks present
  on the meter PCB, so no Muncy "Pin 1" rationale to keep AGND
  isolated from chassis on that board.
- **PFL is pre-fader and pre-mute**, achieved by tapping upstream of
  the MUTE relay. PFL listen continues to work even when the
  channel is muted, matching live-sound semantics. Conceptually
  relocated out of §2.9 (post-pan distribution) into §2.5 (pre-mute
  taps).
- **22 kΩ + DPDT pattern for PFL** rather than a buffered tap:
  shared-bus summing where every active PFL contribution adds via
  its 22 kΩ at the PFL mono bus virtual-earth. The constant load on
  the upstream node (in both OFF and ON, the 22 kΩ sits between
  signal and 0 V) means no DC step on switching → click-free.
- **PFL DETECT as a separate logical bus**, fed via per-channel
  diode wired-OR from Sec2 of the DPDT. Avoids dragging analog PFL
  audio into firmware. The diode prevents inter-channel back-feed
  through the bus.
- **Latching relay (AGQ210A03) for MUTE instead of a CMOS analog
  switch.** Combined reasons: (1) signal-path quality — contact
  resistance ≤ 50 mΩ paralleled (vs ≈ 17 Ω for an ADG419-class
  CMOS), no charge injection, no CMOS distortion vs source-Z
  artefacts; (2) off-isolation effectively unbounded at audio
  through physical contact opening; (3) latching means coil power
  is consumed only during ~10 ms set/reset pulses — no continuous
  140 mW-per-energized-relay budget on the coil rail. Trade-off
  accepted: indeterminate state at power-up (firmware must
  initialize every MUTE relay to RESET / muted at boot, with a
  master-output hard-mute covering the init window — TBD, master
  section), ~ 4 ms switching time, and the need for a bipolar
  pulse driver. Per `00-conventions.md` "Standard signal relay".
- **Series + shunt configuration** for the MUTE relay, rather than
  paralleling both contact sets for lower R_on. Trade-off: series
  R_on rises from ≈ 50 mΩ (paralleled) to ≈ 100 mΩ (single contact
  set) — still negligible in the signal path. Gain: the shunt
  contact explicitly drives the PRE-FADER node to AGND when muted,
  so any residual signal leaking through the open series contact
  (contact capacitance, PCB parasitics at HF) is further attenuated
  by the shunt. Mute depth exceeds open-contact isolation alone.

#### Open issues for Block 3

- **Channels per input PCB** (4 or 6) and resulting **relay coil
  control connector partitioning** (one wide connector per PCB
  carrying SET + RESET lines per channel, vs narrower per-channel
  connectors) — TBD. See `10-open-tbd.md`.
- **Coil driver IC** for the MUTE relay (and, more broadly, for
  every AGQ210A03 in the console): bipolar / half-H-bridge per
  relay (TBD62783A + TBD62083A pair, octal latching-relay driver
  like MAX4820/4821, or per-relay discrete NPN+PNP); partitioning
  and exact part TBD. See `10-open-tbd.md`.
- **Coil supply**: from the +3.3 V logic rail (the 3 V coil
  tolerates +3.3 V — see `00-conventions.md`). Local bulk
  capacitance on the +3.3 V plane sized for the worst-case
  pulse-current transient on each PCB.
- **Other PFL sources feeding PFL DETECT**: AUX masters (§05),
  AUX returns (§03), and groups (§04) all carry PFL controls; do
  they all need to feed PFL DETECT to drive the firmware
  source-select? — TBD. See `10-open-tbd.md`.
- **PFL LED current-limit resistor value** — depends on chosen LED.
- **PFL signal diode** — 1N4148-class, exact part TBD.
- **Coil protection network** for bipolar drive — topology and
  parts TBD with the chosen latching-relay driver IC (often
  integrated). See `10-open-tbd.md`.

---

### Block 4 — Pre-fader node + fader PCB + post-fader amp + AUX/CUE sends (FINALIZED)

**Status:** finalized

#### Topology chosen

*Pre-fader node:*

The paralleled NO outputs of the MUTE relay (AGQ210A03) in Block 3
form the PRE-FADER node.
This node is loaded by three parallel paths:

- **4 × AUX dual-gang pots, pre-fader gangs (gang 1 of each).** Top
  of each pre gang to PRE-FADER, bottom to AGND, wiper drives a
  bus summing resistor toward AUX n PRE bus.
- **1 × CUE dual-gang pot, both gangs identically fed.** Top of
  both gangs to PRE-FADER, bottom of both gangs to AGND. The two
  wipers drive bus summing resistors toward CUE L bus and CUE R
  bus respectively. Topologically identical to an AUX-pre send,
  duplicated across L and R because the channel is mono and the
  CUE bus is stereo — produces a center-panned contribution on
  the stereo CUE bus, with one knob controlling level.
- **Pin 1 of a 3-pin connector** to the fader PCB (see below).

*3-pin connector — channel PCB ↔ fader PCB:*

| Pin | Direction | Signal |
|-----|-----------|--------|
| 1   | channel → fader | PRE-FADER |
| 2   | shared          | AGND (interposed as guard) |
| 3   | fader → channel | POST-FADER |

The AGND between Pin 1 and Pin 3 is intentional: Pin 1 carries the
pre-fader signal and Pin 3 carries the same signal attenuated by
the fader. Although correlated, interposing AGND prevents inter-PCB
ground potential differences from showing up at the wiper.

*Fader PCB:*

Hosts **2 channels per PCB** (12 fader PCBs total for the 24 mono
channels). Each channel uses one half of an NE5532 dual.

The fader PCB also hosts the **3 momentary pushbuttons with integrated
LEDs** for this channel: ACTIVE/MUTE (orange), SOLO (red), REC ARM
(red). Physical co-location with the fader is confirmed; electrical
connection to the digital logic PCB (GPIO routing) is TBD.

Per channel on the fader PCB:

1. **Fader** (mono, taper TBD): top lug = pin 1 (PRE-FADER), bottom
   lug = AGND (pin 2), wiper continues onward.
2. **Wiper DC block + bias**: wiper → C_bp (bipolar electrolytic,
   value TBD) in series → R_bias to AGND (value TBD) → +input of
   one half of an NE5532.
3. **Post-fader amp** (non-inverting, +10 dB nominal):
   - Inverting input: connected to a feedback node, plus R_G to
     AGND.
   - Feedback: from the FB node back to the inverting input through
     R_FB ‖ C_FB.
   - The opamp output pin drives a 680 Ω in series; the FB node is
     taken on the **far side** of the 680 Ω. R_FB ‖ C_FB closes
     back to the FB node.
4. **Output DC block**: 220 µF polarized electrolytic in series →
   pin 3 of the connector.

*Post-fader return on channel PCB:*

Pin 3 of the connector returns the POST-FADER signal to the channel
PCB. This signal feeds in parallel:

- **4 × AUX dual-gang pots, post-fader gangs (gang 2 of each).**
  Top of each post gang to POST-FADER, bottom to AGND, wiper drives
  bus summing resistor toward AUX n POST bus.
- **Top of the pan dual-gang pot** (both gangs — see Block 5).
- (later) Post-Fader Output tap — to be designed (see open issues).

#### Active devices

- **NE5532 × 1 per fader PCB** (dual SOIC; one half per channel,
  used as post-fader amp). Per global default in `00-conventions.md`.

#### Key passive values

- R_FB (post-fader amp): **4.7 kΩ** (0603 1 %).
- R_G (post-fader amp): **2.2 kΩ** (0603 1 %).
- **C_FB (HF pole on R_FB): 100 pF NPO/C0G** (0603). HF pole at
  ≈ 339 kHz.
- Series resistor inside loop: **680 Ω** (0603 1 %).
- Output DC block: **220 µF** polarized electrolytic.
- C_bp (wiper DC block, bipolar): TBD.
- R_bias (wiper to AGND after C_bp): TBD. With NE5532 (I_B ~ 200 nA),
  R_bias = 47 kΩ would produce ~9 mV DC at the +input → ~30 mV at
  the opamp output (post +10 dB), absorbed by the 220 µF output DC
  block. Final value to be set when C_bp is sized.
- 4 × AUX dual-gang pots: value TBD.
- 1 × CUE dual-gang pot: value TBD.
- 8 × AUX bus summing resistors per channel (4 pre + 4 post): TBD.
- 2 × CUE bus summing resistors per channel (L + R): TBD.

#### Design targets / expected performance

- Post-fader amp gain: **+9.93 dB** (1 + 4.7/2.2 = 3.136), nominal
  +10 dB.
- Post-fader amp HF pole: ≈ 339 kHz (R_FB ‖ C_FB). Audio-band
  response flat to within −0.015 dB at 20 kHz.
- Post-fader amp output impedance at audio: ≈ 0 Ω at the FB node
  (pulled there by feedback through R_FB; 680 Ω is inside the
  loop). At HF, where loop gain falls, the 680 Ω becomes effective
  output impedance and isolates the opamp from capacitive load /
  short-circuit conditions on the connector and 220 µF.
- Pre-fader node source impedance: ≈ 50 mΩ (parallel of two
  AGQ210A03 contacts, 100 mΩ each initial) + bias paths from the
  upstream stage (Block 3) — effectively dominated by upstream
  impedances.
- Post-fader amp drive: NE5532 is rated for 600 Ω loads at low
  distortion; the parallel load presented by 5 × 10 kΩ pot tops
  (4 AUX-post + pan) is ≈ 2 kΩ, well within capability. Verify when
  AUX/CUE/pan pot values are finalized (§05, §06).

#### Decisions and tradeoffs

- **Fader on a separate PCB hosting 2 channels.** Partitions the
  console into modular sub-assemblies aligned with the mechanical
  layout (fader bank in front of the channel strips). The 3-pin
  connector keeps the fader PCB minimal and standard across all
  channels.
- **3-pin connector with AGND interposed** between PRE-FADER and
  POST-FADER lines: prevents inter-PCB ground potential differences
  from appearing as offsets at the wiper.
- **NE5532 for the post-fader amp.** Per global default in
  `00-conventions.md`. The +input sees the wiper through C_bp:
  C_bp blocks DC, so the bipolar input bias current of NE5532
  (~200 nA) flows only through R_bias to AGND, producing a static
  DC offset at the +input independent of fader position. No
  position-dependent thump on fader movement. The 220 µF output
  DC block absorbs the resulting offset before it reaches
  downstream stages. Drive capability into the post-fader load
  (4 AUX-post gangs + pan + future post-fader output) is well
  within NE5532's specified 600 Ω rating.
- **680 Ω inside the loop** (Self's "zero-impedance output" pattern,
  c.f. Fig. 22.5c): R_FB closes back to the FB node, not to the
  opamp output pin. At audio frequencies, feedback pulls the FB-node
  impedance to near zero; at HF, where loop gain falls, the 680 Ω
  isolates the opamp output pin from capacitive loading on the
  connector / 220 µF and limits short-circuit current. Single
  resistor, two functions.
- **C_FB = 100 pF NPO/C0G** in parallel with R_FB: defines the HF
  pole at ≈ 339 kHz, ~one decade above audio. Three combined
  functions: (1) limits HF gain above audio to attenuate ultrasonic
  content; (2) provides preventive stability margin against parasitic
  capacitance at the −input and capacitive loading on the connector;
  (3) rejects RF residue. Coherent with C_FB = 220 pF on the active
  pan stage in Block 5 (same target HF pole frequency, scaled by
  R_FB).
- **Single dual-gang pot per AUX (one knob = one pre + one post
  send)** rather than 4+4 separate pots. Halves AUX knob count, at
  the cost of forcing the same level shape on pre and post — usually
  what one wants anyway.
- **CUE pot wired with both gangs receiving the pre-fader signal
  identically**: makes a one-knob send to a stereo CUE bus from a
  mono channel, coherent with the "send centered on the stereo
  bus" semantics. Topologically just a duplicated AUX-pre.

#### Open issues for Block 4

- **C_bp** (wiper DC block, bipolar): value TBD.
- **R_bias** (wiper to AGND after C_bp): value TBD. Together with
  C_bp these set the LF rolloff into the post-fader amp's +input
  and define the bias condition of the wiper at startup.
- **Polarity of the 220 µF output DC block**: depends on DC at the
  next stage on the channel PCB (the AUX-post gang tops and the
  pan top). To be set when measured on rev 1.
- **AUX dual-gang pot value** (× 4) and **AUX bus summing resistor
  values** (× 8: 4 pre + 4 post). To be calculated together with
  AUX master summing topology (§05).
- **CUE dual-gang pot value** (× 1) and **CUE bus summing resistor
  values** (× 2: L, R). To be calculated together with CUE master
  summing topology (§06).
- **Fader part / taper / law** — not specified.
- **3-pin connector type / cable** (twisted pair shielded vs flat):
  TBD.

---

### Block 5 — Active pan (FINALIZED)

**Status:** finalized

#### Topology chosen

*Pan pot:*

- **Dual-gang 10 kΩ LIN, center-detent preferred.** Reverse-wired
  between the two gangs so that at one extreme of rotation, one
  wiper sees the input signal while the other sees AGND.
- Top / bottom of each gang to POST-FADER signal / AGND
  (orientations opposite between L gang and R gang).
- Wiper of each gang feeds the +input of one half of an NE5532.

*Active pan stage* (per side, one half-NE5532 per side, both halves
of one NE5532 used for L and R):

Self's active panpot (Fig. 22.12, *Small Signal Audio Design*):
the law-bend resistor is driven from the opamp output rather than
from a fixed voltage, forming a negative-impedance-converter that
shapes the law toward sin/sin² compromise and improves offness
from ≈ 65 dB (passive) to ≈ 90 dB (active).

Per opamp half, three resistors and one capacitor:

- **R_LAW = 3.3 kΩ** — opamp output → non-inverting input. The wiper
  also connects to this same node (i.e. R_LAW and the wiper meet at
  the +input).
- **R_FB = 2.2 kΩ** — opamp output → inverting input. Self canonical.
- **R_G = 10 kΩ** — inverting input → AGND.
- **C_FB = 220 pF NPO/C0G** in parallel with R_FB. HF pole at
  ≈ 329 kHz.

The "non-inverting amp gain" alone (ignoring the law-bend loop) is
1 + R_FB/R_G = 1.22 → **+1.73 dB**.

*Becomes-stereo point:*

The two opamp output pins are the post-pan L and R signals. **The
channel becomes stereo at these two pins** — this is the canonical
"becomes stereo" point for the mono channel.

*Output:*

Post-pan L and R feed Block 6 (rotary routing) + AFL switch +
Post-Fader Output. No DC block at the pan output: NE5532 V_os ≤
0.5 mV typical, and the bias current through R_FB / R_G produces a
small additional DC offset that downstream summers / receivers
either absorb (virtual-earth nodes) or block (AC-coupled inputs).

#### Active devices

- **NE5532 × 1 per channel** (dual SOIC; both halves used, one for
  L, one for R). Per global default in `00-conventions.md`.

#### Key passive values

- R_LAW = 3.3 kΩ × 2 (one per side) = 2 per channel.
- **R_FB = 2.2 kΩ × 2** = 2 per channel. Self canonical.
- R_G = 10 kΩ × 2 = 2 per channel.
- **C_FB = 220 pF NPO/C0G × 2** per channel. HF pole at ≈ 329 kHz.
- Pan pot: dual-gang 10 kΩ LIN, center-detent preferred.

#### Design targets / expected performance

(With Self's canonical values: pot = 10 kΩ, R_LAW = 3.3 kΩ,
R_FB = 2.2 kΩ, R_G = 10 kΩ.)

- Hard-pan output level: **+1.73 dB** (relative to POST-FADER, on
  the active side).
- Center output level: **−2.77 dB** (relative to POST-FADER, each
  side).
- **Center dip: ≈ 4.5 dB** — Self's canonical sin/sin² compromise
  law. Psychoacoustically uniform perceived level across the pan
  travel.
- HF pole: ≈ 329 kHz. Audio-band response flat to within −0.02 dB
  at 20 kHz.
- Offness when panned hard one side: limited by pot end-of-track
  resistance, ≈ 90 dB on the inactive side (active law-bending
  improvement over the ≈ 65 dB of a passive pan).

#### Decisions and tradeoffs

- **Active pan over passive law-bending.** Self's pattern: cleaner
  pan law and significantly better offness at the cost of one dual
  opamp per channel.
- **Self canonical values (R_FB = 2.2 kΩ, R_G = 10 kΩ, R_LAW = 3.3 kΩ,
  pot = 10 kΩ LIN).** The four values are tuned together to land on
  the sin/sin² compromise law with ≈ −4.5 dB center dip — the
  psychoacoustic compromise between constant-amplitude (−6 dB at
  center, hole-in-the-middle) and constant-power (−3 dB at center,
  bump-in-the-middle). Increasing R_FB (e.g. to 4.7 kΩ) would
  flatten the dip toward ≈ 2 dB and produce a "soft center" voicing
  with a perceptible level jump from hard-panned to center; we
  follow Self.
- **C_FB = 220 pF NPO/C0G**: HF pole at ≈ 329 kHz. Three combined
  functions: (1) limits HF gain above audio; (2) gives preventive
  stability margin in the law-bend positive-feedback loop (R_LAW
  closes from the opamp output back to the +input — limiting HF
  forward gain caps the loop gain before parasitic phase shifts can
  cause trouble); (3) rejects RF residue at the wiper, which is a
  high-Z node with front-panel cabling. Coherent with C_FB = 100 pF
  on the post-fader amp in Block 4 (same target HF pole frequency,
  scaled by R_FB). NPO/C0G dielectric for low voltage coefficient
  and low loss.
- **NE5532**: per global default in `00-conventions.md`. The +input
  sees the wiper (variable Z, 0 to ~2.5 kΩ depending on pan position)
  in parallel with R_LAW = 3.3 kΩ from the output. With NE5532 I_B
  ~ 200 nA, position-dependent DC offset at the +input is at most
  ~500 µV across the pan travel — negligible and inaudible at any
  practical pan-knob speed. Block 5 is identified as a candidate
  for selective opamp upgrade (OPA1642 JFET, or OPA1656 for noise)
  at the budget-review stage; the topology and all passive values
  are unchanged across this upgrade — see `00-conventions.md` and
  `10-open-tbd.md`.
- **Center-detent pot preferred**: gives the engineer a tactile
  reference for "hard center" without looking. Optional.
- **No DC block at pan output**: NE5532 output offset is small, and
  downstream destinations are either virtual-earth bus summers
  (DC-tolerant) or AC-coupled inputs. Saves capacitors and avoids
  inserting LF poles.

#### Open issues for Block 5

- **Pan pot exact part** — dual-gang 10 kΩ LIN center-detent, model
  TBD.

---

### Block 6 — Post-pan routing rotary (PARTIAL — values TBD)

**Status:** in-progress — topology defined, bus summing resistor
values to be calculated together with §07 (Main) and §04 (Groups).

#### Topology chosen

*Rotary switch (§2.9 (a)):*

- **2-gang × 4-position rotary**, mechanical, front-panel.
- Centrals (poles) of gang L and gang R are fed by the post-pan L
  and R signals from Block 5.
- Each of the 4 positions on each gang connects through a bus
  summing resistor to one of the four destination buses
  (Main / G1 / G2 / G3), respectively L or R sub-bus.
- 4 positions × 2 gangs = **8 bus summing resistors per channel**
  for this stage.
- One destination active at a time per channel; the inactive
  positions leave the corresponding bus summing resistor floating
  on the channel side (no back-grounding).

#### Active devices

None on this stage.

#### Key passive values

- 8 × bus summing resistors per channel: TBD. To be calculated
  together with §07 (Main summer) and §04 (group summers).
- Rotary switch: 2-gang, 4-position, mechanical, front-panel; part
  TBD.

#### Decisions and tradeoffs

- **Rotary over individual mechanical push-switches per
  destination**: simpler front panel, only one position active by
  construction, saves panel space at the cost of being unable to
  send a single channel to multiple stereo destinations
  simultaneously. Acceptable given the modest group count (3) and
  the operator's ability to use AUX sends and groups together for
  multi-destination workflows.
- **Open inactive positions, not back-grounded**: with the post-pan
  L/R NE5532 outputs sourcing each destination through its own
  bus summing resistor, the load presented to the pan stage varies
  with rotary position by ≈ 0 (one resistor at a time, all
  approximately the same value). No need for back-grounding;
  saves 8 resistor positions per channel on the rotary.

#### Open issues for Block 6

- **Bus summing resistor values** (8 per channel × 24 channels =
  192 resistors total, plus contributions from AUX returns and
  groups). To be solved together with §04 and §07.
- **Rotary switch part selection** (2-gang, 4-position, mechanical,
  make-before-break vs break-before-make): TBD. Probably
  break-before-make to avoid momentary cross-routing during
  position change.

---

### Block 7 — AFL switch (post-pan stereo tap) (FINALIZED)

**Status:** finalized — topology and relay wiring fixed; AFL bus
summing resistor values deferred to §08.

#### Topology chosen

*AFL tap (§2.9 (b)):*

- Tap directly from each post-pan opamp output (Block 5): one tap
  for L, one for R.
- Each tap feeds a **bus summing resistor** (value TBD — see §08)
  in series, whose far end connects to COM of the respective
  contact set on the AFL relay.
- **Contact set 1** carries L; **contact set 2** carries R. Both
  are mechanically ganged on the same coil (one AGQ210A03, DPDT
  1-coil latching) — L and R switch perfectly synchronously.
- **NC contacts → AGND**: in the RESET state (AFL off), the far
  end of each summing resistor is terminated to AGND. The post-pan
  opamp output always sees a constant resistive load (the summing
  resistor alone) regardless of AFL state — no impedance step on
  switching, no click.
- **NO contacts → AFL bus**: in the SET state (AFL on), the far
  end of each summing resistor connects to the AFL Left / AFL Right
  bus respectively.

Boot-up default: RESET (AFL off, NC contacts on AGND). Per the
console-wide firmware boot-init contract.

#### Active devices

- **AGQ210A03 × 1 per channel** (DPDT 2 form C, 1-coil latching
  signal relay). Contact set 1 = L; contact set 2 = R. Per
  `00-conventions.md` "Standard signal relay".

#### Key passive values

- AFL bus summing resistors: × 2 per channel (L and R). Value TBD
  — to be calculated together with §08 AFL summer design.

#### Design targets / expected performance

- Constant resistive load on post-pan L/R opamp output in both AFL
  states: R_load = summing resistor value (terminated to AGND or
  feeding AFL bus — both look like a resistive load to the opamp).
- Off-isolation: open-contact isolation > 80 dB at 1 MHz per
  datasheet, effectively unbounded at audio.

#### Decisions and tradeoffs

- **NC → AGND (terminated, not floating)**: gives constant
  resistive loading on the post-pan output in both AFL on and off
  states. Avoids any thump or level shift when switching. The
  summing resistor dissipates a small signal power to AGND at all
  times in AFL-off state — negligible at audio signal levels.
- **Tap directly from the post-pan opamp output** (Block 5), no
  buffer. NE5532 V_os is small enough that no DC block is needed
  before the AFL bus summer. The summing resistor serves both as
  AFL bus contribution weight and as switch source impedance.
- **Summing resistor value deferred to §08**: AFL bus summing
  resistors must be designed together with the AFL summer /
  monitor section (§08) as a coherent system (source count,
  individual resistor values, and summer opamp gain interact).

#### Open issues for Block 7

- **AFL bus summing resistor value** (× 2 per channel, L and R).
  TBD together with §08 AFL summer design.

---

### Block 8 — Output PRE-POST switch + switchable Channel Output (IN-PROGRESS)

**Status:** in-progress — signal flow and relay topology fixed;
cold dummy network details and LED color assignments TBD.

#### Topology chosen

*Pre-mute signal tap (on channel PCB):*

- A **75 Ω** resistor taps the pre-MUTE node — the HPF/INSERT bypass
  relay COM2 output — which is the same node that feeds the meter
  buffer (Block 3) and the MUTE relay COM1. This is a branch tap, not
  in series with the main path. The far end of this 75 Ω is one input
  to the Output PRE-POST switch.

*Post-fader signal tap (on channel PCB):*

- A **75 Ω** resistor taps the POST-FADER node (after the
  post-fader amp, before pan). Branch tap. Far end = other input
  to the Output PRE-POST switch.

*Output PRE-POST switch (on channel PCB, front-panel):*

- **DPDT mechanical switch, front-panel.** Pattern A per
  `00-conventions.md`.
- Section 1 (signal): common feeds pin 1 of the 2-pin connector
  (below). Throws select between the pre-mute 75 Ω output (PRE)
  and the post-fader 75 Ω output (POST).
- Section 2 (indicator): drives two front-panel LEDs (one per
  position) from +3.3V / DGND. Colors TBD —
  see `10-open-tbd.md`.

*2-pin connector — channel PCB → input PCB:*

- **Pin 1 (hot)**: the signal selected by the Output PRE-POST
  switch (already through the chosen 75 Ω build-out).
- **Pin 2 (cold dummy)**: a dedicated **75 Ω to AGND** only — no
  47 µF ‖ 47 kΩ completion. The two source paths have incompatible
  upstream DC-block conditions (PRE path: 47 µF blocks with 47 kΩ
  pull-down; POST path: 220 µF output block from fader PCB with no
  pull-down at the tap point), so a single matched cold dummy
  network cannot suit both positions. A simple 75 Ω to AGND gives
  frequency-independent impedance balance over this short internal
  run and is the correct compromise.

*Channel Output SELECT relay (on input PCB):*

- **AGQ210A03** (DPDT 2 form C, 1-coil latching signal relay, per
  `00-conventions.md`). See also Block 1 (Channel Output section) for
  the PCB context.
- **Contact set 1 (hot line):** COM → Channel Output EMI filter tip
  (→ TRS tip). NC = 75 Ω build-out on Input A hot buffer (existing
  path). NO = pin 1 of 2-pin connector (channel PRE/POST signal).
- **Contact set 2 (cold line):** COM → Channel Output EMI filter ring
  (→ TRS ring). NC = 75 Ω build-out on Input A cold buffer
  (existing path). NO = pin 2 of 2-pin connector (cold dummy from
  channel PCB).
- **RESET state (Input A — default):** Channel Output = full-
  differential from Input A buffers through their matched 75 Ω
  build-out resistors. Original behavior, unchanged.
- **SET state (Channel signal):** Channel Output = impedance-balanced;
  hot = selected PRE or POST signal (already through 75 Ω); cold =
  75 Ω-to-AGND cold dummy from channel PCB.
- In both states, signal passes through the standard T-balanced
  double-pi EMI filter on the Channel Output TRS jack.

#### Active devices

- **AGQ210A03 × 1 per channel** (DPDT 2 form C, 1-coil latching
  signal relay): Channel Output SELECT. Per `00-conventions.md`
  "Standard signal relay". Sits on the input PCB.

#### Key passive values

- Pre-mute 75 Ω tap: **75 Ω** (0603 1 %), on channel PCB.
- Post-fader 75 Ω tap: **75 Ω** (0603 1 %), on channel PCB.
- Cold dummy 75 Ω: **75 Ω** (0603 1 %), on channel PCB.
- Cold dummy bypass / load network (47 µF ‖ 47 kΩ): TBD.
- Output PRE-POST switch LED current-limit resistors: TBD.

#### Design targets / expected performance

- Channel Output in Input-A state: full-differential, same
  specification as Block 1 Channel Output.
- Channel Output in Channel-signal state: impedance-balanced output.
  Source impedance looking into tip = 75 Ω (matched to cold dummy
  75 Ω to AGND). Downstream CMR limited by impedance balance,
  not by differential drive.

#### Decisions and tradeoffs

- **Channel Output jack as multifunction output**: combines the
  original full-differential Input A tracking output with a
  channel-signal output (pre-mute or post-fader). Eliminates the
  need for a separate rear-panel "Post-Fader Output" jack
  (original §2.9 (e) concept), reducing jack and connector count.
- **Output PRE-POST as a mechanical switch (not firmware-
  controlled)**: set directly by the operator, appropriate for
  a per-session configuration choice (recording pre vs post fader).
  No firmware needed for this selection.
- **Channel Output SELECT as a relay (AGQ210A03, firmware-controlled)**:
  allows remote switching of the Channel Output source (e.g. for a
  global "engage all channel sends" command, or DAW integration),
  consistent with the console-wide relay-for-firmware-control
  approach.
- **75 Ω build-outs on both PRE and POST tap paths**: provides the
  correct source impedance for the Channel Output EMI filter in the
  Channel-signal state (same rationale as Input A build-out —
  see Block 1). Both taps use 75 Ω to stay consistent with the
  standard impedance-balanced output convention.
- **Cold dummy as plain 75 Ω to AGND (no 47 µF ‖ 47 kΩ completion)**:
  the two source paths have incompatible upstream conditions. The
  PRE path has 47 µF DC blocks with 47 kΩ pull-down upstream; the
  POST path exits the fader PCB through a 220 µF output block with
  no pull-down at the tap point. A single completion network cannot
  properly match both positions. Plain 75 Ω to AGND gives frequency-
  independent impedance balance over this short internal PCB-to-PCB
  run — the correct practical compromise.

#### Open issues for Block 8

- **Output PRE-POST LED color assignment**: two LEDs, one per
  position (PRE / POST). Colors TBD.
- **Channel Output SELECT front-panel control**: latching DPDT
  pushbutton, named "DIRECT / PROCESSED output". Section 1
  connects to MCU GPIO input (MCU reads state change and fires
  the relay set/reset pulse accordingly). Section 2 drives an
  indicator LED (TBD color). The button's mechanical latch means
  the MCU always reads the desired state directly — no toggle
  logic needed, no sync ambiguity at boot.
- **Channel Output SELECT relay boot state**: RESET (Input A /
  direct default) is the expected choice; to be confirmed and
  recorded in the firmware boot-init spec.

---

### Block 9 — HPF / INSERT chain bypass relay (IN-PROGRESS)

**Status:** in-progress — relay wiring and front-panel hardware
established; logical mode cycling and boot state TBD.

#### Topology chosen

Standard **AGQ210A03** DPDT 1-coil latching signal relay per
`00-conventions.md`. Contact wiring per §2.3.1:

| Contact | Connection |
|---|---|
| COM1 | CHANNEL SOURCE relay output |
| NC1 | Hard-wired to NC2 |
| NC2 | Hard-wired to NC1 |
| COM2 | Pre-MUTE node (meter buffer + PFL tap + MUTE relay COM1) |
| NO1 | HPF switch Section 1 input (chain entry) |
| NO2 | Insert Return receiver output, post-C38 (chain exit) |

- **RESET (NC active):** COM1 → NC1 ↔ NC2 → COM2. HPF / INSERT
  chain bypassed.
- **SET (NO active):** COM1 → NO1 → [HPF + INSERT chain] → NO2 →
  COM2. Chain in signal path.

Front-panel control:

- **Momentary pushbutton**, single contact, MCU GPIO input.
- **Three separate orange LEDs**, each driven by an individual MCU
  GPIO output: FOLLOW PATH (center), FOLLOW A, FOLLOW B — per the
  mode table in §2.3.1.

#### Active devices

- **AGQ210A03 × 1 per channel** (DPDT 2 form C, 1-coil latching
  signal relay). Per `00-conventions.md` "Standard signal relay".

#### Key passive values

- HPF/INSERT BYPASS LED current-limit resistors (× 3 per channel):
  TBD (depends on chosen LED Vf and brightness target).
- Coil drive: bipolar pulse from +3.3 V per `00-conventions.md`
  "Standard signal relay". Driver IC TBD.

#### Open issues for Block 9

- **Boot state**: RESET (chain bypassed by default) or SET (chain
  in path by default) — TBD. See `10-open-tbd.md`.
- **Mode cycling logic and CHANNEL SOURCE synchronization**:
  firmware decision — TBD. See `10-open-tbd.md`.
- **Front-panel button part** (momentary, single contact, no
  integrated LED): TBD.

---

### Block 10 — Dual meter buffers (FINALIZED)

**Status:** finalized

#### Topology chosen

*Active meter buffer (TL072 half 1):*

- Input: pre-MUTE node (HPF/INSERT bypass relay COM2). DC-free in both
  relay states per Block 3. No DC block at the TL072 input.
- Bias return: 47 kΩ to AGND already present at the pre-MUTE node
  (from Block 2 R22 / Block 1 post-receiver DC-block network, depending
  on bypass relay state). TL072 JFET input bias current ≈ pA —
  DC offset at input ≈ 0. No additional bias resistor needed.
- Configuration: unity-gain voltage follower (inverting input
  connected directly to output).
- Output: 75 Ω build-out → pin 1 of 3-pin connector to meter bridge.

*Inactive meter buffer (TL072 half 2):*

- Input: CHANNEL SOURCE relay contact set 2, COM2 output. Set 2 is
  wired complementary to set 1 (see Block 1): NC2 = Receiver B
  (post-DC-block); NO2 = Receiver A (post-DC-block). In RESET state
  (A active), COM2 delivers Receiver B; in SET state (B active), COM2
  delivers Receiver A. Source selection is performed by the relay
  itself — no additional electronic switch.
- DC-free: 47 µF DC blocks in Block 1 are upstream of both receiver
  outputs. No DC block at the TL072 input.
- Bias return: 47 kΩ pull-down at each receiver output (Block 1
  post-DC-block networks). TL072 JFET input bias current ≈ pA →
  DC offset ≈ 0. No additional bias resistor needed.
- Configuration: unity-gain voltage follower (inverting input
  connected directly to output).
- Output: 75 Ω build-out → pin 2 of 3-pin connector to meter bridge.

*3-pin connector — channel PCB → meter bridge PCB:*

- Pin 1: active meter signal (post-75 Ω). Full brightness.
- Pin 2: inactive meter signal (post-75 Ω). Dimmed on meter bridge.
- Pin 3: AGND.

The meter bridge PCB carries a full AGND pour (no TRS jacks — Muncy
Pin 1 rule does not apply). A separate DGND pour serves the LED driver
ICs. Dimming of the inactive bargraph is a meter bridge design decision
— the channel PCB delivers both signals at full amplitude.

#### Active devices

- **TL072** × 1 per channel (dual SOIC, JFET input): both halves used.
  Half 1 = active meter buffer (pre-MUTE tap); half 2 = inactive meter
  buffer (CHANNEL SOURCE relay contact set 2 output).

#### Key passive values

- Meter buffer build-out: **75 Ω × 2** (0603 1 %), one per TL072
  half output.

#### Design targets / expected performance

- Both buffers: unity gain, DC–audio band. Output drives ≈ 100 pF
  cable capacitance via 75 Ω build-out without ringing.
- DC offset at both TL072 inputs: ≈ 0 (JFET bias current in pA range).

#### Decisions and tradeoffs

- **CHANNEL SOURCE relay contact set 2 for inactive meter switching**
  (no additional electronic switch): contact set 2, wired with NC/NO
  complementary to set 1, delivers the inactive receiver at all times
  with relay-contact quality (Ron ≤ 100 mΩ, isolation > 80 dB at
  1 MHz). Eliminates one IC per channel and all associated passives.
- **TL072 over NE5532**: ~3× lower quiescent current (~2.5 mA/package
  vs ~8 mA) and JFET inputs (bias current ≈ pA vs ~200 nA). For a
  meter buffer — not in the critical audio chain, destination is a
  rectifier — TL072 is preferable: lower power and zero DC offset at
  the input. Higher voltage noise (18 nV/√Hz vs 5 nV/√Hz) is
  irrelevant for this application.
- **No bias resistors at TL072 inputs**: JFET inputs draw pA; upstream
  47 kΩ pull-downs provide the return path. DC offset ≈ 0.
- **3-pin connector (vs two separate 2-pin connectors)**: saves one
  connector per channel; shared AGND adequate for this short internal
  run to the meter bridge.
- **Dimming on meter bridge, not on channel PCB**: channel PCB delivers
  both signals at full amplitude; meter bridge applies different drive
  conditions per input. Keeps channel PCB simple.

#### Open issues for Block 10

- **Connector type** for 3-pin to meter bridge PCB: TBD.
- **Inactive bargraph dimming ratio**: LED drive threshold or current
  ratio on meter bridge PCB — TBD when meter bridge is designed.

---

## 2.11 Front-panel control buttons

Per mono channel there are **10 pushbuttons** in three categories:

**Momentary with integrated LED (3) — physically on the fader PCB:**
ACTIVE/MUTE (orange), SOLO (red), REC ARM (red). LED is inside the
button body; not listed separately in the LED inventory. Connection
from fader PCB to digital logic PCB: TBD.

**Momentary without integrated LED (2) — on channel/input PCB:**
CHANNEL SOURCE (§2.2) and HPF/INSERT BYPASS (§2.3.1).

**Latching DPDT (5) — on channel/input PCB:**
HPF (§2.3.2), INSERT (§2.3.3), PFL (§2.5.2), Output PRE/POST
(§2.9 (e)), DIRECT/PROCESSED output (§2.9 (e) / Block 8). For
each: section 1 carries the signal or GPIO-to-MCU control line;
section 2 drives an indicator LED.

### ACTIVE/MUTE (orange LED)

- Button press → firmware toggles the **MUTE relay** (AGQ210A03,
  §2.6) between ACTIVE (SET) and MUTED (RESET).
- Orange LED on = channel ACTIVE; off = MUTED. Driven by MCU GPIO.
- Boot default: LED off, relay RESET (channel muted).

### SOLO (red LED)

- Button press → firmware:
  1. Sets the **AFL relay** (AGQ210A03, §2.9 (b)) of this channel
     to SET — post-pan L/R joins the AFL bus.
  2. Optionally (SIP mode): drives the **MUTE relay** of every
     other channel to RESET (muted), leaving this channel's MUTE
     relay in its current ACTIVE state.
- Red LED on = this channel soloed. Driven by MCU GPIO.
- Un-pressing SOLO reverses both actions: AFL relay to RESET;
  other channels' MUTE relays restored to their pre-solo state.
- AFL-only vs. SIP mode and multi-SOLO priority: firmware
  decisions, TBD.

### REC ARM (red LED)

- Button press → firmware toggles the REC ARM state of this channel.
- No audio relay involved. Firmware signals armed/unarmed state
  toward the DAW via MIDI (exact protocol TBD).
- Red LED on = channel armed for recording. Driven by MCU GPIO.
- Boot default: LED off (unarmed).

### HPF / INSERT BYPASS (three separate orange LEDs)

- Button press → firmware cycles the HPF/INSERT bypass relay
  (AGQ210A03, §2.3.1) through three modes: FOLLOW PATH → FOLLOW A
  → FOLLOW B → FOLLOW PATH → …
- Three separate orange LEDs (not integrated in the button), each
  driven by a dedicated MCU GPIO output. Exactly one LED is on at
  all times, indicating the current mode.
- Boot default and mode cycling logic: TBD — see `10-open-tbd.md`
  and Block 9 open issues.
