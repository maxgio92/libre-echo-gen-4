## Purpose

Give the Echo 4 port a single, cited hardware inventory built from the public FCC filing, so later kernel, DTS, audio, and boot work start from documented facts instead of guesses.

## ADDED Requirements

### Requirement: Complete FCC exhibit archive
The project SHALL keep an offline copy of every public exhibit for FCC ID 2AVVJ-5273 (model L4S3RE) inside the repository, so recon does not depend on external sites staying up.

#### Scenario: Every exhibit is captured
- **WHEN** the archive is reviewed against the FCC exhibit list for 2AVVJ-5273
- **THEN** each listed exhibit (internal photos, external photos, block diagram, schematics, operational description, parts list, test reports, software security statement, user manual) has a stored local file
- **AND** any exhibit the FCC marks confidential or does not publish is recorded as missing with its exhibit name and reason

#### Scenario: Archive is traceable
- **WHEN** a stored exhibit file is opened
- **THEN** its source URL, FCC exhibit type, and retrieval date are recorded alongside it

### Requirement: Cited hardware inventory with confidence
The project SHALL maintain a hardware inventory that lists each hardware fact about the `laser` board with a confidence level (confirmed, likely, unknown) and the exhibit it came from.

#### Scenario: Facts cite a source
- **WHEN** any entry in the inventory is read
- **THEN** it names the source exhibit (or external research reference) and a confidence level
- **AND** no confirmed entry lacks a citation

#### Scenario: Required fact classes are present
- **WHEN** the inventory is checked for coverage
- **THEN** it has an entry (a value or an explicit "unknown") for each of: main SoC, RAM package, storage (eMMC), boot medium and partition-layout clues, Wi-Fi/BT part, Wi-Fi/BT firmware availability, audio codec, amplifier, DSP involvement, LED controller, PMIC, microphone topology, mic-disable mechanism, antenna layout, and board-to-board connectors

### Requirement: Recorded open questions
The project SHALL record the hardware questions the FCC exhibits cannot answer, so downstream changes know what still needs reverse engineering or teardown.

#### Scenario: Unknowns are actionable
- **WHEN** a fact class is marked unknown in the inventory
- **THEN** an open question captures what is missing and which later change (`audio-path-discovery` or `laser-board-port`) is expected to resolve it

### Requirement: Recorded regulatory questions
The project SHALL record the legal and regulatory questions the port raises, so they are surfaced before work that depends on their answers.

#### Scenario: Regulatory posture is noted
- **WHEN** the recon open questions are reviewed
- **THEN** they include the anti-circumvention exposure of the boot bypass (relevant if the work is published) and the compliance note for changing radio behavior (leaving Zigbee, Sidewalk, and 900-MHz radios uninitialized) on an FCC-certified device
