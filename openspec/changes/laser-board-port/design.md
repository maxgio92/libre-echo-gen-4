## Context

LibreEcho splits into a kernel repo (Linux 6.1.178-derived, board drivers + DT/DTS + config, currently MT8163) and a platform repo (initramfs, recovery, boot envelope, OTA, owner firmware import). The MT8512 is a close MediaTek relative of the MT8163: same Cortex-A53 family, similar AFE and peripheral IP, much of it already mainline. So the port is mostly device tree plus config plus a few board drivers, not new SoC enablement from scratch. The gating risk is boot: no owner-code path on the Echo 4 is demonstrated. The realistic vector on a locked MediaTek appliance is the BROM / Download-Agent layer (mtk_uartboot, mtkclient-style DA exploits), not Fenrir. Fenrir's `bl2_ext` verification bypass needs an already-unlocked bootloader (`fastboot flashing unlock`), which the Echo 4 likely will not expose, so it is a secondary path only. See proposal.md for motivation.

## Goals / Non-Goals

**Goals:**
- Boot mainline-based Linux on the Echo 4 and play local audio.
- Keep all Echo 4 code under `laser/` so the base and Gen-2 target stay untouched.
- Order work so the boot blocker is tested before expensive driver work.

**Non-Goals:**
- Alexa, wake word, speech capture, cloud services. Removed by design.
- Zigbee, Sidewalk, and 900-MHz radio in the first release. Deferred.
- Microphone capture. Deferred with the audio capture path.
- OTA and recovery hardening. Reuse the platform repo's machinery; do not redesign it here.

## Decisions

- **Reuse the MT8163 board layout as the template for `laser/`.** The Gen-2 target already proves the folder shape (kernel config fragment, DTS, firmware handling, audio, boot packaging). Copy that shape for MT8512. Alternative (new layout) rejected: it would diverge two boards that should stay parallel.
- **Base the kernel on the LibreEcho 6.1.178 tree, not fresh mainline.** LibreEcho already carries the Echo-specific patches and build glue. Rebasing onto newer mainline is a separate, later effort. Alternative (latest mainline) rejected for the first port: more unknowns, no payoff yet.
- **Prove boot with a minimal payload before kernel work.** The first milestone loads a tiny UART/LED payload through the candidate boot path. This tests the Fenrir/`bl2_ext` question cheaply. Alternative (build the full kernel first, then try to boot) rejected: it risks weeks of work against a locked bootloader.
- **Device tree first, drivers only where mainline lacks them.** Most MT8512 peripherals have mainline drivers; the work is describing them in `laser.dts` with correct pinctrl and I2C/I2S wiring. Write a new driver only when the codec, amp, or LED controller has none.
- **Gate audio bring-up on `audio-path-discovery`.** The DTS sound nodes and ALSA UCM come straight from that change's contract. Do not guess routes here.

## Risks / Trade-offs

- No owner-controlled boot path exists on Echo 4. Mitigation: milestone 0 tests this first; if it fails, the port stops at a documented result instead of sinking effort into a kernel that cannot load.
- MediaTek BROM authentication (SBC / signed Download Agent) may be enabled on `laser`, so even download mode would reject an owner DA and need its own BROM exploit. This is the true dead-end risk. Mitigation: M0 checks BROM-auth state explicitly before assuming download mode gives code execution.
- MT8512 config or a peripheral IP block differs from MT8163 in a way the DTS cannot cover. Mitigation: bring up peripherals one at a time over UART, starting from a minimal DTS.
- Wi-Fi firmware blob is Amazon-specific or missing. Mitigation: identify the Wi-Fi part in `fcc-hardware-recon`, then source the matching upstream/vendor firmware; treat as its own task.
- Bricking the device during boot experiments. Mitigation: confirm a recovery/download-mode path (MediaTek BROM) before writing anything persistent.

## Migration Plan

Milestones, in order. Each gates the next.

- M0 Boot proof: reach a BROM/DA code-exec path (check BROM-auth state first), then load a minimal payload; confirm owner code runs. Blocker gate.
- M1 Board scaffold: create `laser/` (kernel config fragment, empty `laser.dts`, boot packaging stub); Gen-2 still builds.
- M2 Kernel to shell: minimal DTS (CPU, RAM, eMMC, UART); boot to a shell.
- M3 Storage and Wi-Fi: eMMC rootfs, SDIO Wi-Fi up, network association.
- M4 Audio: sound card, codec, amp per the contract; test tone then music.
- M5 LED and headless polish: LED ring from userspace; strip Amazon stack; reproducible image.
- M6 Packaging and OTA: fold into the platform repo's boot envelope, recovery, and OTA.

Rollback: all experiments use BROM download mode as the recovery path; nothing persistent is written until M2 boots cleanly.

## Open Questions

- Which exact MT8512 boot path is viable (Fenrir-style `bl2_ext` bypass, an unlockable bootloader, or another route)? This is M0's job to answer; it does not change the milestone order.
