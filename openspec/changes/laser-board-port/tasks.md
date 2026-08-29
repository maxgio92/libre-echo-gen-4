## 0. Boot proof (blocker gate)

- [ ] 0.1 Confirm a BROM download/recovery path on the Echo 4 so experiments are reversible, and check the BROM-auth state (whether the SoC requires a signed Download Agent); verify the device enters download mode, is detectable by host tools, and record whether an owner DA is accepted
- [ ] 0.2 Assess MT8512 owner-code options against the device, primary first: BROM/Download-Agent code-exec (mtk_uartboot, mtkclient-style DA exploit), then the Fenrir-style `bl2_ext` bypass (only if the bootloader can be unlocked); verify each option is marked viable or ruled out with evidence
- [ ] 0.3 Load a minimal owner payload that signals over UART or the LED; verify owner code runs on the device
- [ ] 0.4 If no owner boot path exists, record the result and stop the port here; verify the outcome is documented before any kernel work starts

## 1. Board scaffold

- [ ] 1.1 Create the `laser/` board tree mirroring the MT8163 layout (kernel config fragment, `laser.dts` stub, boot packaging stub); verify the tree exists and mirrors the Gen-2 shape
- [ ] 1.2 Add the MT8512 SoC selection/config on top of the LibreEcho 6.1.178 tree; verify the kernel configures for MT8512
- [ ] 1.3 Build both targets; verify the Gen-2 target still builds unchanged and `laser` builds an image

## 2. Kernel to shell

- [ ] 2.1 Write a minimal `laser.dts` (CPU, RAM, timers, UART); verify it compiles to a DTB
- [ ] 2.2 Package kernel + DTB + minimal initramfs and boot on device; verify the kernel reaches an interactive UART shell
- [ ] 2.3 Confirm CPU cores, RAM size, and clocks; verify detected values reconcile against the `fcc-hardware-recon` inventory, logging any mismatch (inventory RAM/storage may be only "likely" from photos)

## 3. Storage and networking

- [ ] 3.1 Add eMMC and pinctrl to `laser.dts`; verify the kernel detects eMMC with correct size and mounts a rootfs
- [ ] 3.2 Add SDIO Wi-Fi node and load the matching firmware; verify the interface appears and scans networks
- [ ] 3.3 Associate to a 2.4/5 GHz network; verify the device gets an IP and passes a ping test
- [ ] 3.4 Add the UART Bluetooth node; verify the controller enumerates (HCI up)

## 4. Audio

- [ ] 4.1 Add I2C codec/PMIC and I2S/amp nodes to `laser.dts` from the `audio-path-discovery` contract (probe-phase fields finalized after M3); verify the sound card registers
- [ ] 4.2 Add or select the codec and amplifier drivers (write only if no mainline driver exists); verify the codec and amp probe without error
- [ ] 4.3 Add the ALSA UCM/routes for the contract's channel map; verify a test tone plays on the woofer and both tweeters
- [ ] 4.4 Play the full audio test set; verify the contract's per-speaker pass condition (no distortion or channel swap)

## 5. LED and headless polish

- [ ] 5.1 Add the LED ring controller to `laser.dts` and expose it to userspace; verify the ring can be driven from a userspace command
- [ ] 5.2 Remove Alexa, wake word, speech capture, and Amazon services from the image; verify none of those processes run on a booted system
- [ ] 5.3 Leave Zigbee, Sidewalk, and 900-MHz radio uninitialized; verify they are absent from the first-release config
- [ ] 5.4 Leave the microphone array uninitialized (no capture node in `laser.dts`, no recording service); verify no capture path exists and document the hardware mic-disable mechanism from `fcc-hardware-recon`
- [ ] 5.5 Make the board image build reproducible from the repo; verify a clean build produces the image with no manual steps

## 6. Packaging and OTA

- [ ] 6.1 Fold `laser` boot artifacts into the platform repo's boot envelope format; verify the envelope loads on device
- [ ] 6.2 Wire `laser` into recovery and OTA machinery, reusing the existing platform code; verify a recovery boot and an OTA update round-trip succeed
- [ ] 6.3 Run `openspec validate laser-board-port --strict`; verify it passes
