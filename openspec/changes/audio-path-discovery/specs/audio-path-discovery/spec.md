## Purpose

Give the Echo 4 port a documented, cited audio chain and a target audio contract, so the DTS, codec/amp drivers, and ALSA config in `laser-board-port` are built against known hardware instead of trial and error.

Confidence levels (confirmed, likely, unknown) and provenance tags (document, probed, unverified) use the definitions in `docs/roadmap.md` ("Claim vocabularies").

## ADDED Requirements

### Requirement: Identified audio chain
The project SHALL document the Echo 4 audio chain from SoC to speaker, naming each stage and its part number where known, each with a confidence level and a source.

#### Scenario: Chain stages are named
- **WHEN** the audio findings document is read
- **THEN** it names, in order, the MT8512 audio controller, the digital audio link (I2S or TDM), the codec, any DSP, the amplifier(s), and the speaker set (3" woofer plus two tweeters)
- **AND** each named IC has a part number or an explicit "unknown", a confidence level, and a source

#### Scenario: Amp and codec control paths are captured
- **WHEN** the codec and amplifier are identified
- **THEN** the document records their I2C bus and address (or SPI, if used) and any amp-enable, reset, or mute GPIO they require

### Requirement: Digital audio format and clocking
The project SHALL determine the digital audio format between the MT8512 and the codec: sample rate, sample format, channel count, TDM slot layout if used, and the master/slave clock roles.

#### Scenario: Format is specified
- **WHEN** the audio findings document is read
- **THEN** it states the I2S/TDM format, bit clock and frame clock configuration, and which side is clock master
- **AND** any value not yet verified on hardware is marked "unverified"

### Requirement: Target audio contract
The project SHALL define a target audio contract for the Echo 4, in the same terms as the Gen-2 contract, that later port work must satisfy.

#### Scenario: Contract is complete and testable
- **WHEN** the target contract is read
- **THEN** it states sample rate, sample format, transport channel count, channel mapping to the physical speakers (including any mono-to-stereo duplication), the PCM/route names, clock configuration, and crossover handling
- **AND** it states the pass condition: clean playback of a known test tone and a music file through the woofer and both tweeters without distortion or channel swap

### Requirement: Evidence provenance
The project SHALL record, per finding, whether it came from a document, from live hardware probing, or remains unverified until the OS boots.

#### Scenario: Provenance is explicit
- **WHEN** any audio finding is read
- **THEN** it is tagged document, probed, or unverified
- **AND** findings that depend on a running kernel are flagged as gated on `laser-board-port` boot bring-up
