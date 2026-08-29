## Purpose

Define the Echo 4 `laser` board target so a free OS boots on MT8512 hardware and plays local audio, added on top of the existing LibreEcho base without changing the Gen-2 target.

## ADDED Requirements

### Requirement: Board target is additive
The port SHALL add the Echo 4 as a `laser` board target without modifying the generic LibreEcho base or the existing Gen-2 (`Radar/Puffin`) target.

#### Scenario: Gen-2 still builds
- **WHEN** the `laser` target is added and the Gen-2 target is built
- **THEN** the Gen-2 build still succeeds unchanged
- **AND** Echo 4 specifics live under a `laser/` path, not in shared base files

### Requirement: Proven owner boot path
The port SHALL prove that the Echo 4 accepts an owner-supplied kernel before committing to full kernel and driver work.

#### Scenario: Custom payload runs
- **WHEN** an owner-built minimal payload is loaded through the chosen MT8512 boot path
- **THEN** the device executes owner code (observable over UART or an LED signal)
- **AND** if no such path is found, the change records that result and stops before deep kernel work rather than assuming one exists

### Requirement: MT8512 kernel boots
The port SHALL provide an MT8512 kernel, based on the LibreEcho Linux 6.1.178 tree, that boots to a userspace shell on the Echo 4.

#### Scenario: Boot to shell
- **WHEN** the packaged kernel, DTB, and initramfs are loaded on the device
- **THEN** the kernel boots to an interactive shell over UART
- **AND** CPU, RAM, and storage (eMMC) are detected with correct sizes

### Requirement: Echo 4 device tree
The port SHALL provide `laser.dts` describing the Echo 4 hardware the OS uses.

#### Scenario: Core peripherals described
- **WHEN** `laser.dts` is compiled and booted
- **THEN** it declares pinctrl, eMMC, the I2C buses with their devices (codec, PMIC), the I2S/TDM link and amplifier, SDIO Wi-Fi, UART Bluetooth, and the LED ring
- **AND** it omits or disables Alexa-specific, wake-word, and speech-capture hardware paths

### Requirement: Working local audio
The port SHALL play local audio through the Echo 4 speakers per the contract from `audio-path-discovery`.

#### Scenario: Audio plays correctly
- **WHEN** a known test tone and a music file are played through ALSA
- **THEN** sound comes from the 3" woofer and both tweeters with no distortion or channel swap
- **AND** the sample rate, format, channel map, and routes match the target audio contract

### Requirement: Networking and control I/O
The port SHALL bring up Wi-Fi and the LED ring so the device is usable headless.

#### Scenario: Wi-Fi connects
- **WHEN** the OS is configured with credentials
- **THEN** the device associates to a 2.4/5 GHz network and gets an IP
- **AND** the LED ring can be driven from userspace to signal state

### Requirement: Echo 4 boot packaging
The port SHALL package boot artifacts the Echo 4 boot chain accepts.

#### Scenario: Reproducible image
- **WHEN** the board build runs
- **THEN** it produces a kernel image, DTB, and initramfs in the boot envelope format the MT8512 chain accepts
- **AND** the build is reproducible from the repo without manual steps

### Requirement: Amazon stack removed
The port SHALL exclude Alexa, wake word, speech capture, Amazon services, and (initially) Zigbee and Sidewalk from the running OS.

#### Scenario: No Amazon services run
- **WHEN** the OS is booted and running
- **THEN** no Alexa, wake-word, speech-capture, or Amazon service process is present
- **AND** Zigbee and Sidewalk radios are left uninitialized in the first release

### Requirement: Microphone left disabled
The port SHALL leave the microphone hardware disabled in the first release, so the free OS does not capture audio.

#### Scenario: Mic hardware is not brought up
- **WHEN** the OS is booted and running
- **THEN** the microphone array is left uninitialized (no capture path in `laser.dts` and no recording service running)
- **AND** the hardware mic-disable mechanism identified in `fcc-hardware-recon` is documented so a later change can decide whether the OS must honor it
