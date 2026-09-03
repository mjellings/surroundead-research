# `Slot.sav`

`Slot.sav` contains metadata useful for presenting and identifying save folders.

🟢 **Observed fields / regions:**

- `Name`
- `Level` — map/level asset path
- `Timestamp` — serialized `DateTime`
- a `Players` region from which player-name strings can be recovered in examined saves

This file can be useful for a save-management tool because it provides human-readable metadata without needing to fully parse the much larger inventory representation in `PlayerInfo.sav`.

🔵 The complete schema remains unmapped.
