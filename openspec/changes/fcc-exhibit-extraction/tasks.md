## 1. Locate the filing

- [ ] 1.1 Confirm the FCC ID and grantee for the Echo 4 `laser` unit on the official FCC OET search and on fcc.report/fccid.io; verify by matching model L4S3RE across all three sources
- [ ] 1.2 List every exhibit filed under 2AVVJ-5273 (type, filename, date, public vs confidential); verify the list matches the FCC exhibit index page count

## 2. Build the archive

- [ ] 2.1 Create `docs/hardware/fcc/2AVVJ-5273/`, track PDFs with Git LFS, and download every public exhibit into it; verify each listed public exhibit has a local file
- [ ] 2.2 Record source URL, exhibit type, and retrieval date per file in an `INDEX.md` next to the exhibits; verify every stored file has an INDEX row
- [ ] 2.3 List each confidential or unpublished exhibit in the same INDEX with reason "confidential/withheld"; verify no exhibit from task 1.2 is missing from INDEX

## 3. Extract facts

- [ ] 3.1 Read the block diagram (or reconstruct one from photos if the diagram is sealed) and record the SoC-to-peripheral topology; verify the topology names every major block seen in the internal photos
- [ ] 3.2 Read the parts list (if public) and record part numbers for SoC, RAM, eMMC, Wi-Fi/BT, codec, amplifier, PMIC, LED controller; verify each part number resolves to a real datasheet
- [ ] 3.3 Read the internal photos at full resolution and transcribe legible chip top-line markings per board; verify each transcribed marking cites the photo page it came from
- [ ] 3.4 Record antenna layout and board-to-board connector positions from photos and RF test reports; verify the antenna count matches the RF test report
- [ ] 3.5 Record the microphone topology (array count and placement) and any hardware DSP presence from photos, block diagram, or operational description; verify each has an inventory value or an explicit "unknown"
- [ ] 3.6 Determine the Wi-Fi/BT part and whether matching upstream firmware exists (linux-firmware or vendor); verify the firmware availability is recorded as available, custom/unknown, or absent, so the risk surfaces before SDIO bring-up in `laser-board-port` M3
- [ ] 3.7 Record any boot-medium and partition-layout clues (eMMC topology, preloader/BL2 offsets) visible in the operational description or block diagram; verify each is captured or routed to a teardown/probe follow-up if the FCC set is silent

## 4. Produce the inventory

- [ ] 4.1 Write `docs/hardware/inventory.md` as a table: fact class, value, confidence, source exhibit; verify every fact class from the spec has a row
- [ ] 4.2 Set confidence per the design rule (confirmed needs schematic/parts-list/legible marking; else likely or unknown); verify no "confirmed" row lacks a citation
- [ ] 4.3 For every "unknown" row, add an open question naming the follow-up change; verify each unknown maps to `audio-path-discovery` or `laser-board-port`

## 5. Validate

- [ ] 5.1 Run `openspec validate fcc-exhibit-extraction --strict` and confirm it passes
- [ ] 5.2 Cross-check the inventory against one independent Echo 4 teardown and note any disagreement; verify each disagreement is logged as an open question
