# Research Status

This directory separates findings by confidence and keeps unresolved questions visible.

## Confidence system

- 🟢 **Confirmed** — reproduced in-game, in save data, or across reloads.
- 🟡 **Probable** — evidence is strong but verification is incomplete.
- 🔵 **Research** — unresolved or active investigation.
- 🔴 **Failed / obsolete** — tested approach that failed or has been superseded.

## Current highlights

🟢 Oodle wrapper/decompression has been reproduced on examined saves.

🟢 Core `Player.sav` progression values can be located structurally rather than with fixed offsets.

🟢 `PlayerInfo.sav` contains item records with item identity, stats and container-placement data.

🟢 Live inventory slots expose weapon stat data corresponding to in-game values.

🟢 Physical firearms can be tracked by GUID, resolved to live `JSI_Slot_C` objects and modified through `BP_JigComponent:UpdateStatByUID` when the live slot's native `ItemUniqueID` wrapper is used.

🟢 Runtime stat writes in the tested Weapon Progression path do **not** persist through a full SurrounDead restart by themselves. Mod-owned persistence and deterministic reconstruction from captured base stats are the confirmed production model.

🟢 More than one live `JSI_Slot_C` may carry the same physical weapon GUID. Zero-stat/stub matches exist, so GUID equality alone is not sufficient; populated `ItemStats` must be part of candidate selection.

🟢 SurrounDead's native firearm tooltip can be extended through its existing UMG widgets. Numeric rows are `BP_StatW_C`, text rows are `BP_StatTextW_C`, and additional native-looking Level / XP / Kills rows have been demonstrated in `TextStatsGrid`.

🟢 A complete standalone `UserWidget` can also be created directly from UE4SS Lua on SurrounDead 0.8 without a custom Blueprint or cooked `.pak`. `UserWidget`, `WidgetTree`, `CanvasPanel`, `Border`, `TextBlock` and `ProgressBar` have all been constructed and added to the viewport at runtime.

🟢 The tested standalone UI keeps presentation separate from authoritative mod state through a provider/state-table model and can be opened, refreshed and removed cleanly at runtime.

🟡 `MainJigContainers` appears to use a less obvious serialization layout than its declared ArrayProperty count initially suggests.

🔵 Remaining targets include stronger authority rules for duplicate populated live slots, exact vanilla XP-scaling provenance, more special-weapon layouts, equipment/attachment save mapping, broader tooltip/widget mapping, and interactive standalone UMG controls such as buttons, lists, images and focus/input handling.

See the [Discovery Log](discovery-log.md), [Runtime Weapon Stats](runtime-weapon-stats.md), [Runtime Tooltip / UI Research](runtime-tooltip-ui.md) and [Standalone Lua UMG Research](runtime-standalone-ui.md) for consolidated technical details.
