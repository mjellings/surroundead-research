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

🟢 Runtime stat changes made through SurrounDead's exposed update path survive a normal save/reload because the game serializes the modified item stats.

🟡 `MainJigContainers` appears to use a less obvious serialization layout than its declared ArrayProperty count initially suggests.

🔵 Equipment mapping, attachments, complete container hierarchy, enemy death events and equipped-weapon attribution remain open targets.

See the [Discovery Log](discovery-log.md) for dated findings and [Runtime Weapon Stats](runtime-weapon-stats.md) for the bridge between save-file and live-runtime research.
