# Save File Format

> **Scope:** primarily SurrounDead 0.8 / Unreal Engine 5.6. These are observed reverse-engineering notes, not an official specification.

## Save location

🟢 **Confirmed** — Windows saves are stored beneath:

```text
%LOCALAPPDATA%\SurrounDead\Saved\SaveGames\
```

Each save is represented by a folder. Observed files include:

| File | Observed purpose |
| --- | --- |
| `PlayerInfo.sav` | Inventory, containers and item instances |
| `Player.sav` | Level, XP, vitals, skills and skill points |
| `Slot.sav` | Save display metadata, map/level, timestamp and player-name hints |
| `General.sav` | Present; not yet deeply mapped |
| `Level.sav` | Present; not yet deeply mapped |
| `Difficulties.sav` | Present; not yet deeply mapped |

Other quest or skill-tree saves may also appear. Folder naming can provide clues for Quick Save, Auto Save and Manual Save classification.

## High-level file pipeline

🟢 **Confirmed** for examined 0.8-era saves:

```text
.sav bytes
   ↓
optional EMSH prefix
   ↓
Oodle compressed wrapper block(s)
   ↓
raw payload
   ↓
GVAS or newer property-stream representation
```

Newer files may begin with ASCII `EMSH`. A practical decoder can scan the first ~256 bytes for the known Oodle wrapper magic and treat preceding bytes as a prefix.

### Oodle wrapper

Observed magic:

```text
C1 83 2A 9E 22 22 22 22
```

Observed block header:

| Offset | Size | Field |
| ---: | ---: | --- |
| 0 | 8 | Magic |
| 8 | 8 | BlockSize (`uint64`) |
| 16 | 1 | CompressionMethod |
| 17 | 8 | TotalCompressedSize |
| 25 | 8 | TotalUncompressedSize |
| 33 | 8 | ChunkCompressedSize |
| 41 | 8 | ChunkUncompressedSize |
| 49 | variable | Compressed chunk bytes |

Blocks can repeat.

🟢 **Confirmed** — decompression can use `oo2core_8_win64.dll` shipped with the game under its `Binaries\Win64` directory.

## Payload forms

🟢 Older/legacy payloads may begin with `GVAS`, or an `int32` length followed by GVAS data.

🟡 0.8-era saves can instead expose a newer property stream sometimes referred to in this research as **TJBO-style**. The complete outer header/schema is not yet mapped, and the useful property stream does not necessarily begin at byte zero.

See [Unreal serialization notes](unreal-serialization.md) and the individual save-file pages for more detail.

## Safety

Writing saves is substantially riskier than reading them. Preserve wrapper/header fields exactly unless their meaning and update requirements have been verified. Always create a backup before modifying a save.
