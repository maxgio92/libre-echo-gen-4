# Echo 4 (`laser`) audio path

Document-phase draft for the `audio-path-discovery` change (tasks 2.1, 2.3).
No hardware probed yet. Every Echo 4 value here is `unverified` until the probe
phase (gated on `laser-board-port` boot bring-up). Confidence and provenance tags
use the definitions in [`../roadmap.md`](../roadmap.md) ("Claim vocabularies").

This draft predates the FCC hardware inventory (`fcc-exhibit-extraction`). Part
numbers below are hypotheses from public teardowns, not from the FCC parts list.

## Gen-2 contract (the template)

Source: `LibreEcho-Linux-6.1` README, "Production audio contract"
(hardware-verified 2026-08-07), and `arch/arm/boot/dts/libreecho-radar-puffin.dts`.
Provenance `document`, confidence `confirmed` for Gen-2 (does not transfer to Echo 4).

| Field | Gen-2 (Radar-Puffin) value |
|-------|----------------------------|
| Producer input | stereo `S16_LE`, 48 kHz |
| Programme processing | mix/downmix/duck/limit once to mono |
| Hardware output | duplicated mono, `S16_LE`, 48 kHz, 2 channels |
| Hardware PCM | PCM 23 / `hw:0,23` |
| Board channel config | `Stereo` |
| Codec | MediaTek SoC internal codec (MT6351-class); `DACSETUP=0x14` |
| Routes | HP/HPL/HPR enabled; LO/LOL/LOR line-out disabled; `Audio_DacMux_Setting=Off` |
| Amp | external analog amp with gain/DAC-mux select pins (`audexamp*` pinctrl); not I2C smart amp |
| Clocking | PRB_P2 resource class 12, IFACE3 `DACMOD2BCLK`; MCLK from SENINF |
| Crossover | calibrated, in-hardware |
| Channel map | left slot → left DAC → HPL/tweeter; right slot → right DAC → HPR/woofer |

Key point: Gen-2 drives the speakers through the **SoC internal codec plus a
gain-pin external amp**. It does **not** use an I2C-controlled smart amp.

## Echo 4 (`laser`) draft contract

All rows `unverified`. Confidence is the ceiling the document evidence allows.

| Field | Echo 4 draft value | Confidence | Provenance |
|-------|--------------------|------------|------------|
| SoC audio controller | MT8512 AFE, I2S/TDM ports | likely | document |
| Digital link | I2S (TDM if >2 slots) | unknown | unverified |
| Codec | unknown; may be MT8512 internal codec or external | unknown | unverified |
| Amp | TI TAS5805M Class-D smart amp (I2S in, I2C control, enable GPIO) | likely | document |
| Amp driver | `sound/soc/codecs/tas5805m.c` present in the LibreEcho tree | confirmed (driver exists) | document |
| DSP in playback path | assume none (SoC AFE → I2S → amp); MT8512 voice DSP + AZ1 sit on capture/wake path | likely | unverified |
| Sample rate / format | 48 kHz, `S16_LE` (Gen-2 shape, target) | unknown | unverified |
| Speakers | 3" woofer + 2x 0.8" tweeters | confirmed | document |
| Channel map | woofer + two tweeters; count/route unknown | unknown | unverified |
| Amp-enable / reset / mute GPIO | unknown | unknown | unverified |
| I2C bus/address for amp/codec | unknown | unknown | unverified |

## Gen-2 → Echo 4 divergences (what does NOT carry over)

1. **Amp topology likely changes.** Gen-2 uses a gain-pin analog amp off the SoC
   codec. Echo 4 is hypothesized to use the TAS5805M, an I2C-controlled I2S smart
   amp. Different driver, different control path (I2C + enable GPIO vs gain pins),
   different DTS nodes. The Gen-2 `audexamp*` pinctrl does not apply.
2. **Speaker count differs.** Gen-2 is 2-way (one tweeter path, one woofer path).
   Echo 4 is 3 drivers (woofer + two tweeters), so the channel map and possibly
   amp count differ. Much published 4th-gen audio data is from the Echo **Dot** 4
   (single speaker); do not promote a Dot part to `confirmed` for `laser`.
3. **Codec may be internal, not external.** Unknown whether the MT8512 drives the
   amp from its own AFE directly (no separate codec) or through an external codec.
   The probe phase (I2C scan) resolves this.
4. **Architecture.** Gen-2 audio config is ARM32; the MT8512 port is arm64, so the
   defconfig audio symbols and DTS bindings are the arm64 equivalents, not copies.

## Target audio contract (draft pass condition)

Reuse the Gen-2 field shape so the DTS author can diff the two. The Echo 4 must,
once the probe phase finalizes values:

- play a known test tone and a music file at 48 kHz `S16_LE` through ALSA;
- produce clean output from the 3" woofer and **both** 0.8" tweeters;
- with no distortion and no channel swap (each physical speaker verified by tone).

Per-speaker verification is the gate. A route that plays silence or swaps drivers
is a failure, not a finding. The pass/fail gate itself is `laser-board-port` task 4.4.

## Open questions routed to the probe phase / teardown

- Exact amp part and count (one TAS5805M for all three drivers, or more?). I2C scan.
- Amp-enable/reset/mute GPIO. Toggle-and-observe.
- I2S vs TDM and slot layout for three drivers. Scope/logic-analyzer on I2S lines.
- Whether a codec sits between the AFE and the amp. I2C scan.
- Whether the amp needs a proprietary init sequence or firmware blob. Capture stock
  I2C traffic during teardown.
