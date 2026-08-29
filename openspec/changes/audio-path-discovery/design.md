## Context

The MT8512 exposes standard MediaTek AFE (audio front end) blocks with I2S/TDM ports. On MediaTek reference designs the codec is usually an external I2C-controlled part and the speaker amp is a separate I2S-fed, I2C-controlled smart amp with an enable GPIO. The Gen-2 LibreEcho tree proves the method: pin the exact PCM index, routes, and clocking by validating on hardware, not from theory. This change repeats that method for the Echo 4. See proposal.md for motivation. Candidate part numbers come from `fcc-hardware-recon`.

Confidence and provenance vocabularies are defined once in `docs/roadmap.md` ("Claim vocabularies"). Use those definitions; do not redefine them here.

Strong public evidence already exists and should be treated as primary document-phase input, not a late cross-check. A TechInsights teardown of the Echo 4 (L4S3RE, report DDT-2011-802) is published, and the 4th-gen Echo family uses the TI TAS5805M Class-D smart amp (I2S in, I2C control, enable GPIO), which has a mainline ASoC driver at `sound/soc/codecs/tas5805m.c`. That is exactly the "prefer mainline-supported part" signal, and it lets much of the contract reach "likely" now. Caveat: much published 4th-gen audio-IC data is from the Echo Dot 4 (single speaker); the full Echo 4 (`laser`) has a 3" woofer plus two tweeters and may differ in amp count or topology. Do not promote a Dot part number to "confirmed" for `laser`; the per-speaker pass condition guards this.

## Goals / Non-Goals

**Goals:**
- Identify the codec and amplifier with the highest confidence the available evidence allows.
- Produce a target audio contract precise enough to write the DTS sound nodes and ALSA UCM.
- Separate what can be known from documents now from what needs a running kernel.

**Non-Goals:**
- Writing the codec/amp kernel drivers or the DTS. That is `laser-board-port`.
- Reaching bit-perfect DSP parity with Amazon firmware. Local playback is the target, not Alexa signal processing.
- Microphone capture path. Recording is out of scope for the first OS; only note the mic topology if a photo shows it.

## Decisions

- **Two phases: document phase, then probe phase.** The document phase (FCC photos, parts list, MediaTek AFE docs, existing mainline codec/amp drivers) needs no hardware and runs now. The probe phase needs UART/JTAG or a booting kernel and is gated on `laser-board-port`. This lets audio work start immediately without blocking on boot.
- **Prefer mainline-supported codecs/amps.** When a candidate part already has a mainline ASoC driver, that is strong evidence and cuts driver work. Cross-reference each candidate against `sound/soc/codecs/`.
- **Confirm routing by measurement, not assumption.** Once a kernel boots, confirm I2S format and amp-enable GPIO with `amixer`/`aplay` test tones and, if available, a logic-analyzer capture on the I2S lines. A guessed route that plays silence is a failure, not a finding.
- **Reuse the Gen-2 contract shape.** Write the Echo 4 contract in the same fields as Gen-2 so the two are directly comparable and the DTS author can diff them.
- **Ownership boundary between this change and `laser-board-port` M4.** This change's probe phase does not itself register a sound card. Sound-node bring-up (DTS nodes, codec/amp drivers, ALSA UCM) is M4 work in `laser-board-port`. The probe phase draft-first, then confirm: it drafts the contract from documents, M4 implements a first sound card against that draft, then the probe phase measures on that running card (I2C scan, GPIO toggle, scope/logic-analyzer capture, per-speaker tones) and finalizes the contract. So M4 partly precedes the probe phase's measurement steps; there is no strict "probe fully before M4" order.

## Risks / Trade-offs

- Codec/amp markings unreadable in photos and parts list sealed. Mitigation: fall back to probing over I2C once booted (scan bus, match device IDs); until then mark "unknown".
- Amp needs a proprietary init sequence or firmware blob. Mitigation: capture the I2C traffic from stock firmware over a logic analyzer during teardown; note as a probe-phase task.
- No boot yet, so the whole probe phase is blocked. Mitigation: land the full document phase first; the contract can be drafted from documents and tightened after boot.
- Wrong channel map ships silent or swapped speakers. Mitigation: the contract's pass condition requires per-speaker test-tone verification.

## Open Questions

- Does the Echo 4 route audio through a hardware DSP that must be initialized before the amp accepts data? Working assumption: no. The MT8512 voice DSP and the Amazon AZ1 Neural Edge core sit on the capture/wake-word path, so local playback likely goes SoC AFE → I2S → smart amp, bypassing the DSP. Treat this as the default and falsify it in the probe phase; it changes driver work in `laser-board-port`, not the discovery tasks here.
