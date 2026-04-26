# 12 — Mental model summary (one paragraph)

CONSOLE is a 24+4+3 analog desk (24 mono channels, 4 stereo AUX
returns, 3 stereo sub-groups) feeding a Main stereo bus with parallel
SSL-style compression. Channel strips follow a conventional flow
(balanced in → gain → HPF → insert → meter and PFL taps → MUTE
electronic switch → fader → +10 dB post-amp → pan → routing) with
two distinctive design choices: (1) dual balanced inputs A/B with a
full-differential Direct Out from Input A for ADC tracking; (2) every
channel has 4 AUX sends implemented as dual-gang pots that
simultaneously drive a pre-fader
and a post-fader mono bus, giving 8 AUX buses total. The 4 stereo
AUX returns are routing-only (no Post-Fader Output to the outside,
no Insert, no AUX send) — they contribute only to internal buses
(Main / Groups / AFL / CUE / PFL stereo). Groups offer per-AUX
split-or-mono mode switches so a stereo group can feed stereo AUX
pairs either as L/R split or as mono premix. The master section's
distinguishing features are the parallel compression on the main bus
(dry fader + SSL-style compressor + wet fader → summed output) and
an "in front balance" pot that mixes an attenuated main mix behind
the AFL so the engineer always hears the context. Digital control is
limited to hard-muting / enabling electronic switches for ACTIVE,
SOLO, and AFL — no VCAs, no recall, no automation. Talkback, tone
oscillators, and master master meters are deliberately absent.
