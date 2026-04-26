# 07 — Main Mix Master + Parallel Compression

Distinctive feature: **parallel SSL-style bus compression** with two
independent faders (dry and compressed) mixed together at the output.

---

## 7.1 Signal path

1. **Stereo summer** — L and R, sums all Main contributions
   (channels via 4-pos rotary, AUX returns via 4-pos rotary, groups
   via Main on/off switch).
2. **Stereo Insert** — switchable.
   - Send: impedance-balanced TRS (L + R).
   - Return: balanced TRS receiver (L + R).
   - Mechanical switch, as per other inserts.
3. After the insert the stereo signal **splits into two parallel
   paths**:
   - **Path A (DRY)** → **Main stereo fader** → output.
   - **Path B (WET)** → **SSL-style stereo compressor** (black box
     for now — think G-series VCA bus compressor) → **second stereo
     fader** → output.
4. **Two final summers** (one for L, one for R) mix the dry and wet
   outputs into the final Main Mix L and Main Mix R.

This is **bus parallel compression** (New York-style), with
independent control of the uncompressed and compressed levels.

---

## 7.2 Distribution of Main Mix L/R

The final Main Mix L/R is sent to:

- **Main Out A** — 2 × TRS impedance-balanced (L + R).
- **Main Out B** — 2 × TRS impedance-balanced (L + R). Second pair
  for redundancy / split feeds.
- **LED meter buffers** — drive the main master meter.
- **Master monitor section** (see `08-monitor.md`).
- **"In front balance" pot** inside the master monitor — an
  attenuator tapping Main Mix into the AFL summer (see
  `08-monitor.md` §8.3).

---

## 7.3 Implementation details

Status: **conceptual**. No topologies, parts, or values chosen yet.

*(SSL-style bus compressor is the largest sub-project here — it's a
complete VCA compressor design on its own. Treat the compressor as a
separate module; this section records only how it integrates with
the main summing + parallel mix structure.)*
