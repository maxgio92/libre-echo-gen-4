## Why

Audio is the whole point of the port: a free OS on the Echo 4 that plays sound. But the Echo 4 audio path is unknown: the codec, amplifier, DSP involvement, I2S/TDM routing, and amp-enable GPIOs are all unidentified. The Gen-2 LibreEcho project pinned an exact audio contract (48 kHz, S16_LE, 2-channel, mono duplicated to L/R, PCM 23, codec and speaker routes, clock config, crossover) through hardware validation. This change rediscovers the equivalent for the Echo 4 so the DTS and ALSA config in `laser-board-port` are grounded in facts.

## What Changes

- Map the Echo 4 audio chain end to end: MT8512 audio controller, I2S/TDM link, codec, DSP (if any), amplifier, and the 3" woofer plus two tweeters.
- Identify the audio ICs (codec, amplifier) by part number, with confidence, from FCC photos, parts list, and hardware probing.
- Determine the I2S/TDM format, clocking, I2C control addresses, and amp-enable/mute GPIOs the codec and amp need.
- Define the target audio contract for the Echo 4 (sample rate, format, channel map, routes) as the acceptance target for `laser-board-port` audio work.
- Record which facts came from documents versus live probing, and which remain unverified until the OS boots.

## Capabilities

### New Capabilities
- `audio-path-discovery`: an identified, cited Echo 4 audio chain and a target audio contract that the kernel/DTS/ALSA port must satisfy.

### Modified Capabilities
<!-- none -->

## Impact

- New audio findings document and a target audio contract. No production audio driver code in this change; probe scripts and captured logs are allowed as evidence.
- Depends on `fcc-hardware-recon` for candidate part numbers and board topology.
- Feeds `laser-board-port` audio tasks (DTS sound nodes, codec/amp drivers, ALSA UCM).
- Some steps need a running kernel or JTAG/UART on the device, which depends on the boot bring-up in `laser-board-port`. This change marks those steps as gated.
