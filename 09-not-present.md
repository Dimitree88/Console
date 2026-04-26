# 09 — Explicitly NOT present

Things that a typical console might have, but which CONSOLE does NOT
include (per user's explicit decisions):

- No **talkback** mic / circuit.
- No **oscillator** / tone generator for bus calibration.
- No **master VU or PPM meter** (only LED meters per channel and on
  the main mix).
- No **AUX send on AUX returns** (AUX returns cannot be routed to
  AUX buses — avoids obvious feedback loops).
- No **Post-Fader Output (jack out) on AUX returns** — they
  contribute only to internal buses. External access to AUX return
  content can only happen via a bus that they feed (Main, Group,
  CUE).
- No **Insert** on AUX returns.
- No **HPF** on AUX returns or groups.
- No **Direct Out** on Input B of the mono channel (only on
  Input A).
- No **Direct Out** on AUX returns.
- No **dedicated outputs** for AFL or PFL (monitor-only).
- No **CUE PFL** on the CUE master.
