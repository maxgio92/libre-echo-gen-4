## Scope and authorization

This is experimental right-to-repair work on a single, privately owned Echo 4. The boot-path work loads an owner-supplied free OS onto hardware the owner possesses. All experiments run on a device kept disconnected from any network. The goal is running local software the owner controls, not defeating protection on devices the owner does not own, extracting third-party secrets, or any fleet-scale action.

## Why

LibreEcho already runs on the Echo Gen 2 (MT8163, `Radar/Puffin`). Its kernel and platform repos cleanly separate the generic Linux base from board-specific code, so a new board is an additive port, not a rewrite. The Echo 4 (`laser`, MT8512) needs its own board folder: kernel support, device tree, audio, LED/GPIO, and boot packaging. This change lays out that folder and the port plan, so implementation is ordered by dependency and gated on the two hard blockers (unproven boot chain, unknown audio path).

## What Changes

- Add a `laser/` board target that carries all Echo 4 specifics without touching the generic base.
- Enable MT8512 kernel support: base the port on the LibreEcho Linux 6.1.178 tree, add or select the MT8512 SoC config, and reuse mainline drivers where they exist.
- Write the Echo 4 device tree (`laser.dts`): CPU/RAM/storage, pinctrl, I2C/I2S, SDIO Wi-Fi, UART BT, audio codec/amp nodes, and the LED ring.
- Wire the audio path per the `audio-path-discovery` contract: sound card, codec, amp, routes, ALSA UCM.
- Package boot artifacts for the Echo 4: kernel image, DTB, initramfs, and a boot envelope compatible with the MT8512 boot chain.
- Disable or omit Alexa, wake word, speech capture, Amazon services, and initially Zigbee/Sidewalk.
- Prove the boot chain first: validate that the Echo 4 will accept owner-supplied code before deep kernel work. The primary vector is the MT8512 BROM / Download-Agent layer (mtk_uartboot, mtkclient-style DA exploits); the Fenrir `bl2_ext` bypass is a secondary path that requires an already-unlocked bootloader, which a locked appliance may not expose. **BREAKING** for the port plan if this fails: no owner-code path means the port stops at research.

## Capabilities

### New Capabilities
- `laser-board-port`: the Echo 4 `laser` board target across kernel, device tree, audio, GPIO/LED, and boot packaging, with a milestone plan and explicit dependencies.

### Modified Capabilities
<!-- none: the generic LibreEcho base is unchanged; this is an additive board target -->

## Impact

- New `laser/` board tree in the kernel and platform layout; no change to the generic base or the existing Gen-2 target.
- Depends on `fcc-hardware-recon` (hardware inventory) and `audio-path-discovery` (audio contract).
- Boot bring-up here unblocks the probe phase of `audio-path-discovery`.
- Hard external dependency: a working owner-controlled boot path on MT8512. If unavailable, the port cannot load a custom kernel.
