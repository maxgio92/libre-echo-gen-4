## 1. Document phase: identify the ICs

- [ ] 1.1 Pull candidate codec and amplifier part numbers from `fcc-hardware-recon` inventory and photos, and from the published Echo 4 TechInsights teardown (L4S3RE, DDT-2011-802) and known 4th-gen parts (e.g. TI TAS5805M smart amp); verify each candidate has a source citation and flag any part sourced from the Echo Dot 4 as not yet confirmed for `laser`
- [ ] 1.2 Read the MT8512 AFE documentation and list its I2S/TDM ports and clock sources; verify the port list matches the SoC datasheet or reference DTS
- [ ] 1.3 Cross-reference each candidate codec/amp against mainline `sound/soc/codecs/` and `sound/soc/mediatek/`; verify whether a mainline driver exists per candidate
- [ ] 1.4 Record the codec and amp control interface (I2C bus/address or SPI) from parts list, photos, or datasheets; verify each address against the part datasheet

## 2. Document phase: draft the contract

- [ ] 2.1 Draft the target audio contract in the Gen-2 field shape (rate, format, channels, channel map, routes, clocking, crossover); verify every Gen-2 field has an Echo 4 value or an "unverified" marker
- [ ] 2.2 Define the per-speaker pass condition (test tone plus music through woofer and both tweeters, no distortion or swap); verify the condition names each physical speaker
- [ ] 2.3 Write `docs/hardware/audio-path.md` with the chain, control paths, format, and provenance tags; verify every finding is tagged document/probed/unverified

## 3. Probe phase (gated on laser-board-port boot bring-up)

Ownership note: this phase does not register the sound card. Sound-node bring-up (DTS, codec/amp drivers, ALSA UCM) is `laser-board-port` M4 work against the drafted contract. This phase measures on that running card and finalizes the contract. See design.md "Ownership boundary".

- [ ] 3.1 Over UART/JTAG or a booted kernel, scan the I2C bus and match device IDs to confirm codec and amp part numbers; verify scanned IDs match the datasheet device IDs
- [ ] 3.2 Identify the amp-enable/reset/mute GPIO by toggling and observing output; verify the correct GPIO gates audio on and off
- [ ] 3.3 On the M4 sound card, play a test tone through the drafted route; verify clean tone on at least one speaker (card bring-up is M4, not this task)
- [ ] 3.4 Confirm the I2S/TDM format and clock master by scope or logic analyzer on the I2S lines (if available); verify the captured format matches the drafted contract
- [ ] 3.5 Using the M4 card, play the full test set and check per-speaker output to finalize the contract's channel map; verify the contract's per-speaker pass condition (the pass/fail gate itself is `laser-board-port` task 4.4)

## 4. Finalize

- [ ] 4.1 Retag measured findings in `audio-path.md` from provenance "unverified" to "probed" (and raise confidence where the measurement warrants); verify no contract field is left "unverified" after a successful probe phase
- [ ] 4.2 File any unresolved unknowns (e.g. DSP init, proprietary amp sequence) as open questions for `laser-board-port`; verify each unknown names its follow-up
- [ ] 4.3 Run `openspec validate audio-path-discovery --strict` and confirm it passes
