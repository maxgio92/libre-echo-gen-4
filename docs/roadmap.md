# Echo 4 (`laser`) port roadmap

Goal: run a free OS on the Amazon Echo 4th gen (model L4S3RE, codename `laser`, MediaTek MT8512) for local audio, with Alexa and Amazon services removed and the microphone left disabled.

The plans are OpenSpec changes under `openspec/changes/`. Each has a proposal (why/what), specs (what must be true), design (how), and tasks (ordered checklist). View one with `openspec show <name>` or read its folder.

## The three plans, in dependency order

1. **[fcc-exhibit-extraction](../openspec/changes/fcc-exhibit-extraction/)** - build a cited hardware inventory from the public FCC filing (2AVVJ-5273) before touching the device. Feeds the other two. This change produces the capability named `fcc-hardware-recon`, which the other two plans reference by that name.
2. **[audio-path-discovery](../openspec/changes/audio-path-discovery/)** - identify the Echo 4 audio chain (codec, amp, DSP, I2S/TDM, GPIOs) and define the target audio contract. Its document phase runs now; its probe phase waits for a booting kernel.
3. **[laser-board-port](../openspec/changes/laser-board-port/)** - add the `laser` board target: MT8512 kernel, `laser.dts`, audio, LED/GPIO, and boot packaging. Milestone M0 proves an owner boot path before deep work; M4 connects audio to the contract from plan 2.

## Claim vocabularies

Two independent axes tag every hardware claim across all three plans. They are orthogonal: a fact can be "likely" and "document" at once.

- **Confidence** (how sure): `confirmed` needs a schematic, parts list, or legible chip marking; `likely` is a reasoned inference (package shape, teardown match); `unknown` means not yet determined.
- **Provenance** (how known): `document` from a filing/datasheet/teardown; `probed` measured on live hardware; `unverified` asserted but not yet checked on the device.

The order is not a clean 1-2-3: plan 2 and plan 3 interleave. Plan 2's probe phase needs a booting kernel from plan 3, and plan 3's audio milestone (M4) needs plan 2's probed contract. Actual sequence: plan 1 → plan 2 document phase → plan 3 M0-M3 (boot to networking) → plan 2 probe phase → plan 3 M4+ (audio and polish).

## Hard blockers

- **Boot chain unproven.** No owner-code path on the Echo 4 is demonstrated. The primary candidate is the MT8512 BROM / Download-Agent layer (mtk_uartboot, mtkclient-style DA exploits); the Fenrir `bl2_ext` bypass is a secondary path that needs an already-unlocked bootloader, which a locked appliance may never expose. MediaTek BROM authentication (signed-DA requirement) could also block download mode. Plan 3 milestone M0 tests this first; if it fails, the port stops at a documented result.
- **Audio path unknown.** Codec, amplifier, DSP, I2S/TDM routing, and GPIOs are unidentified. Plan 2 resolves this; plan 3 audio depends on it.

## Working the plans

- Start: `openspec show fcc-exhibit-extraction`
- Implement: `/opsx:apply <change-name>`
- Archive when done: `/opsx:archive <change-name>`
