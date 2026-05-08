# 08 — Master Monitor Section

Handles SOLO logic (AFL / PFL), monitor source selection, and
mono/check.

---

## 8.1 Source-select electronic switch (stereo)

Main Mix L/R is the default source. A **stereo relay-driven
electronic switch** (one **AGQ210A03**, DPDT 2 form C, 1-coil
latching, per `00-conventions.md` "Standard signal relay") is
controlled by the SOLO logic:

- If **no solo is active** (RESET state, NC contacts) →
  pass-through Main Mix.
- If **any AFL or PFL is active** (SET state, NO contacts) →
  source switches to the **Solo summer** output.

Contact set 1 carries L, contact set 2 carries R, ganged on one
coil. The boot default is "Main Mix" — implemented by firmware
driving this relay to RESET at startup (the relay being latching,
state at boot is otherwise indeterminate; see "State at power-up
is indeterminate" in `00-conventions.md`).

(Control: firmware. Priority AFL vs PFL when both active is a
firmware decision — to be defined later.)

---

## 8.2 Solo summer

Another stereo relay-driven switch (one **AGQ210A03**, DPDT
2 form C, 1-coil latching) alternates between the **AFL summer**
and the **PFL summer** depending on which mode is active. Contact
set 1 carries L, contact set 2 carries R. The output of whichever
is selected feeds the main Solo summer output (which in turn feeds
the source-select switch above). Per `00-conventions.md` "Standard
signal relay".

---

## 8.3 AFL summer (stereo)

Sums all AFL contributions + an attenuated copy of the Main Mix L/R.

- **"In front balance" pot** — attenuator between Main Mix L/R and
  the AFL summer inputs. Lets the engineer hear the soloed signals
  "in front" of the main mix running in the background. This is NOT
  a standard solo feature — it's a design choice of CONSOLE.

AFL contributions come from all channels, AUX returns, and groups
(via their respective AFL electronic switches). Stereo (post-pan for
channels, post-balance for AUX returns and groups).

---

## 8.4 PFL summer (stereo)

Sums:

- **PFL mono bus** (contributions from 24 mono channels + 4 AUX
  master PFL pushes). Presumably routed to both L and R of the summer
  (center).
- **PFL stereo bus** (contributions from AUX return and groups). L
  to L, R to R.

No "in front balance" on PFL.

---

## 8.5 No dedicated AFL / PFL outputs

AFL and PFL are internal-only — they exist solely to drive the
master monitor source-select. They do not appear on any back-panel
jack.

---

## 8.6 Mono / Check buttons

After the source-select electronic switch, two push buttons (**L**
and **R**) act on the stereo signal going to the monitor output:

- Neither pressed → stereo (L on L line, R on R line).
- **L pressed only** → L on both lines (left-only mono check).
- **R pressed only** → R on both lines (right-only mono check).
- **Both pressed** → L + R mono on both lines (mono check).

---

## 8.7 Monitor output (TBD)

After the mono/check buttons, the signal is the final monitor feed.
Its destination (control-room out, headphone amp, etc.) has not been
decided yet. For now the path "ends" here.

---

## 8.8 Implementation details

Status: **conceptual**, with one part-level decision already
fixed: the source-select switch (§8.1) and the Solo summer
AFL/PFL alternation switch (§8.2) both use the
**AGQ210A03** latching signal relay per `00-conventions.md` — 2
relays from this section. No topology details for the AFL / PFL
summers, the "in front balance" pot, the mono/check network, or
the monitor output stage chosen yet.

*(Design topics expected here: AFL / PFL summer topology and
opamp choice — THD and crosstalk matter; network for the
mono/check buttons — simple summing resistors vs opamp-based
matrix.)*
