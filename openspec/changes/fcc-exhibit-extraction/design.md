## Context

The FCC OET database publishes exhibits by FCC ID. For 2AVVJ-5273 the internal-photo set (19 pages) and the RF test reports are already indexed on public mirrors (fcc.report, fccid.io). Block diagram, schematics, operational description, and parts list are often filed under a short-term confidentiality request. Some become public after the request lapses; some stay withheld. This change extracts what is public and records what is withheld. See proposal.md for motivation.

## Goals / Non-Goals

**Goals:**
- One reproducible pull of all public exhibits into the repo.
- A hardware inventory later changes can read without re-opening raw PDFs.
- Honest confidence marking. Never promote a photo guess to "confirmed".

**Non-Goals:**
- Physical teardown. Deferred; only if FCC data leaves a blocking gap.
- Pin-level I2S/GPIO routing. That is `audio-path-discovery` and, if needed, teardown.
- Filing an FCC request to unseal confidential exhibits. Noted as an open question if it blocks progress.

## Decisions

- **Store exhibits in-repo, not just linked.** Mirrors disappear and confidentiality windows change. A local archive under `docs/hardware/fcc/2AVVJ-5273/` is the durable source of truth. Alternative (bookmark URLs only) rejected: it breaks the moment a mirror rotates. The exhibit set is large binary PDFs; track them with Git LFS so the working tree stays light.
- **Inventory as a single Markdown table with a confidence column.** It is readable, diffable, and greppable by later changes. Alternative (YAML/JSON) rejected for now: the audience writes a DTS by hand, so prose notes matter more than machine parsing.
- **Confidence vocabulary is fixed: confirmed, likely, unknown**, defined once in `docs/roadmap.md` ("Claim vocabularies"). This keeps the signal honest. "Confirmed" needs a schematic, parts list, or a legible chip marking in a photo. A plausible-looking package is at most "likely".
- **Chip identification from photos uses package markings plus cross-reference.** Read the top-line marking, then match against MediaTek reference designs and known Echo teardowns. Record the reasoning so a reviewer can challenge it.

## Risks / Trade-offs

- Schematics and parts list may stay confidential. Mitigation: lean on photos and external teardowns, mark those facts "likely", and route the gap to teardown in `laser-board-port`.
- Photo resolution too low to read chip markings. Mitigation: capture the highest-resolution source; if still unreadable, mark "unknown" rather than guess.
- Mirror content differs from the official FCC record. Mitigation: prefer the official FCC OET link when reachable, and record which source each file came from.

## Open Questions

- If the schematic and parts list stay sealed, is an FCC inspection request worth it, or is teardown faster? Deferrable: it does not change the recon tasks, only what happens after the inventory shows the gap.
- Regulatory: what is the anti-circumvention exposure (DMCA 1201 or equivalent) of publishing the boot bypass, and what compliance notes apply to shipping an FCC-certified device with its Zigbee/Sidewalk/900-MHz radios left uninitialized? Deferrable, but recorded here because the recon change is the natural home and later changes assume publication is fine.
