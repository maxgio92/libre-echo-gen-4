## Why

The Echo 4 (`laser`, MT8512) port needs a hardware inventory before any kernel or DTS work starts. The FCC filing for L4S3RE (FCC ID 2AVVJ-5273) is public and unusually rich: internal photos (task 1.1 confirms the FCC ID, model, and exhibit count) plus block diagram, schematics, operational description, parts list, and a software security statement. Mining these first is cheaper and lower-risk than opening the device, and it drives every later change.

## What Changes

- Pull the full FCC exhibit set for FCC ID 2AVVJ-5273 into a version-controlled, offline archive under the repo: internal and external photos, block diagram, schematics, operational description, parts list, test reports, software security statement, and user manual.
- Extract hardware facts from each exhibit: part numbers, board topology, connector pinouts, antenna layout, and the block diagram.
- Produce a structured hardware inventory that marks each fact as confirmed, likely, or unknown, with the source exhibit cited per fact.
- Record the open questions the FCC set cannot answer, so `audio-path-discovery` and `laser-board-port` know what still needs teardown or reverse engineering.

## Capabilities

### New Capabilities
- `fcc-hardware-recon`: a maintained, cited hardware inventory for the Echo 4 `laser` board derived from FCC exhibits, with per-fact confidence and source.

### Modified Capabilities
<!-- none -->

## Impact

- New directory of downloaded FCC exhibits and a hardware inventory document. No source code.
- Feeds `audio-path-discovery` (audio ICs, I2S/GPIO clues) and `laser-board-port` (SoC, RAM, storage, radios, boot layout clues).
- External dependency: the FCC OET database (fcc.report / fccid.io mirrors) remains reachable; the archive removes that dependency going forward.
