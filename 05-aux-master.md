# 05 — AUX Master (× 4)

Each of the four AUX has its own master stage. Per-AUX structure:

1. **Summer PRE** — summing amp that sums all contributions to AUX n
   PRE bus (from 24 channels + 1 AUX return + 3 groups).
2. **Summer POST** — same, but for AUX n POST bus.
3. **Global PRE/POST switch** — one mechanical switch per AUX master
   selects which of the two summer outputs continues downstream. So
   each AUX master outputs either the pre-fader or post-fader sum,
   not both. The unused summer's output is discarded.
4. **Variable gain** — pot controlling the level of the selected
   signal.
5. **Tap after gain** feeds two destinations:
   - **PFL push button** — momentary/latching push, sends the AUX
     master signal to the PFL mono bus when pressed.
   - **AUX n OUT jack** — TRS impedance-balanced. This is the
     external send output for AUX n (to outboard effect, headphone
     amp, etc.).

**Not present**: no return input back into this AUX bus. An AUX can
be sent externally but returns only via the stereo AUX return
channel, which does not feed back into the AUX bus.

---

## 5.6 Implementation details

Status: **conceptual**. No topologies, parts, or values chosen yet.

*(Main design topics expected here: summing amp topology — classic
virtual-earth vs alternatives; noise budget for summing N channels;
opamp choice driving the bus; gain structure so that the variable
gain pot sits in a sensible range.)*
