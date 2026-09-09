# Research Status

This directory separates findings by confidence and keeps unresolved questions visible.

## Weapon documentation

| Page | Use it for |
| --- | --- |
| [Weapon Reference](weapons.md) | All weapon tables, shared defaults, and spread/recoil explanations. |
| [Weapon DataAsset Research](weapon-data-assets.md) | Extraction methods, the Crusher cross-check and unresolved stat mappings. |
| [Runtime Weapon Stats](runtime-weapon-stats.md) | Reading/modifying live weapon stats, physical UIDs and persistence. |
| [Runtime NPC Interaction](runtime-npc-interaction.md) | Runtime NPC spawning, QuestGiver/trader interaction hooks, custom UMG handoff and gunsmith research. |

## Confidence system

- 🟢 **Confirmed** — reproduced in-game, in save data, or across reloads.
- 🟡 **Probable** — evidence is strong but verification is incomplete.
- 🔵 **Research** — unresolved or active investigation.
- 🔴 **Failed / obsolete** — tested approach that failed or has been superseded.

## Current highlights

🟢 **Confirmed:** live handling tests now demonstrate HK416 recoil directions, independent horizontal-deviation aim movement, HK416 hip-fire spread and Benelli M4 shooting spread during hip-fire. See [test conditions and results](weapons.md#runtime-tests--2026-09-08).

🟡 **Probable:** intermediate parameter reductions will support useful progression bonuses; the amount of gameplay improvement is not yet measured.

🔵 **Research:** positive vertical recoil, vertical deviation, recovery controls, shotgun bounds, spread formulas and production handling persistence.

🔴 **Failed / obsolete:** the initial positive-horizontal-means-right assumption and an ADS-only interpretation of Benelli `ShootingSpread`; live tests superseded both.

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

🟢 Existing native NPC Blueprints can be spawned dynamically with UE4SS Lua. A tested supplies trader retained its normal native interaction/trading flow, while exact generic `BP_QuestGiver_C` and Settlement Officer actors also spawned successfully.

🟢 The generic QuestGiver presents SurrounDead's native `[F] ... Talk To` interaction prompt. Its Blueprint hook callback context is a `RemoteUnrealParam`; `ctx:get()` resolves the actual actor, allowing a specific runtime-spawned NPC to be identified reliably when wrapper identity itself is not stable.

🟢 Native NPC interaction can hand off into a custom Lua-created UMG service panel. The tested QuestGiver prototype opened the existing Weapon Progression UI module as a gunsmith panel after `OnExecuteInteract`, then cleanly removed only its own spawned NPC.

🔵 Writing `VendorName = Dave` succeeds on the spawned QuestGiver but does not change the native prompt, which still displays `Settlement Leader`. The true source of the QuestGiver interaction label remains unresolved.

🔵 The first gunsmith panel proves the interaction architecture, but its compact status-card layout is too narrow for longer service text. A dedicated service layout is the next UI step before implementing the progression-reroll transaction.

🟡 `MainJigContainers` appears to use a less obvious serialization layout than its declared ArrayProperty count initially suggests.

🔵 Remaining targets include stronger authority rules for duplicate populated live slots, exact vanilla XP-scaling provenance, more special-weapon layouts, equipment/attachment save mapping, broader tooltip/widget mapping, interactive standalone UMG controls such as buttons/lists/images/focus, QuestGiver display-name resolution, and a safe preview/accept weapon-progression reroll service.

See the [Discovery Log](discovery-log.md), [Runtime Weapon Stats](runtime-weapon-stats.md), [Runtime Tooltip / UI Research](runtime-tooltip-ui.md), [Standalone Lua UMG Research](runtime-standalone-ui.md) and [Runtime NPC Interaction](runtime-npc-interaction.md) for consolidated technical details.
