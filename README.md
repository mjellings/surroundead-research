# SurrounDead Research

Unofficial community reverse-engineering notes for **SurrounDead**, currently focused mainly on the **0.8 / Unreal Engine 5.6** era.

This repository collects practical findings from save-file reverse engineering and live UE4SS investigation. The aim is to build a useful technical reference for understanding SurrounDead's save data, inventory/container structures, weapon stats, Unreal serialization and exposed runtime behaviour.

> [!WARNING]
> This project is not affiliated with or endorsed by the developer of SurrounDead. The game is under active development, so structures and behaviour may change between builds. Back up your saves before experimenting.

## What's here?

- **Save File Format** — save locations, EMSH, Oodle blocks, Unreal property serialization, `Player.sav`, `PlayerInfo.sav`, `Slot.sav`, inventory and container structures.
- **UE4SS / Runtime Research** — reflection, hooks, inventory objects, weapon stats, GameplayTags and persistence observations.
- **Research Notes** — confirmed discoveries, probable findings, failed approaches, open questions and a dated discovery log.

## Confidence markers

| Marker | Meaning |
| --- | --- |
| 🟢 **Confirmed** | Reproduced in-game, in saves, or across reloads. |
| 🟡 **Probable** | Strong evidence exists, but verification is incomplete. |
| 🔵 **Research** | Active or unresolved research target. |
| 🔴 **Failed / obsolete** | Tested approach that did not work or has been superseded. |

Failed experiments are intentionally retained. Knowing what *doesn't* work can be valuable when reverse engineering.

## Start here

- [Save file research](docs/save-format/README.md)
- [UE4SS and runtime research](docs/modding/README.md)
- [Current research status](docs/research/README.md)
- [Discovery log](docs/research/discovery-log.md)
- [Runtime weapon-stat research](docs/research/runtime-weapon-stats.md)

## Third-party mods

Some runtime discoveries were made while investigating compatibility problems with existing third-party SurrounDead mods. Those mods are **not authored or owned by this repository's maintainer**.

This repository documents observations about SurrounDead itself. It does not claim authorship of third-party mods and does not intentionally redistribute their source code, assets or original implementation. Where a third-party mod is relevant to the history of a discovery, it should be treated only as research context unless its licence explicitly permits otherwise.

## Current scope

Most findings currently relate to **SurrounDead 0.8 / Unreal Engine 5.6**. Older GVAS saves and intermediate builds may differ from newer 0.8-era property streams.

The project is documentation-first. Findings should distinguish observed facts from interpretation, include the game/build context where known, and preserve earlier conclusions when new evidence changes our understanding.

## Contributions and corrections

If you reproduce a finding, discover a new structure, or find something here that is wrong on another build, issues and pull requests are welcome. Please include the SurrounDead build/version and enough detail to reproduce the observation where possible.

---

**Research principle:** when new evidence changes our understanding, record the new evidence rather than silently rewriting the history of the discovery.
