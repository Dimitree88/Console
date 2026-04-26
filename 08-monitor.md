# 08 — Master Monitor Section

Handles SOLO logic (AFL / PFL), monitor source selection, and
mono/check.

---

## 8.1 Source-select electronic switch (stereo)

Main Mix L/R is the default source. A **stereo electronic switch** is
controlled by the SOLO logic:

- If **no solo is active** → pass-through Main Mix.
- If **any AFL or PFL is active** → source switches to the **Solo
  summer** output.

(Control: firmware. Priority AFL vs PFL when both active is a
firmware decision — to be defined later.)

---

## 8.2 Solo summer

Another (stereo) electronic switch alternates between the **AFL
summer** and the **PFL summer**, depending on which mode is active.
The output of whichever is selected feeds the main Solo summer output
(which in turn feeds the source-select switch above).

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

Status: **conceptual**. No topologies, parts, or values chosen yet.

*(Design topics expected here: choice of analog switch for the
source-select and AFL/PFL alternation — THD and crosstalk matter;
network for the mono/check buttons — simple summing resistors vs
opamp-based matrix.)*
