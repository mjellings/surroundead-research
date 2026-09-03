# UE4SS / Runtime Research

This section documents observations about SurrounDead's live Unreal objects and functions, primarily on **0.8 / Unreal Engine 5.6**.

It is deliberately about the **game's exposed runtime behaviour**, not redistribution of third-party mod code.

## Environment

🟢 SurrounDead 0.8 moved from an earlier Unreal Engine version to **UE 5.6**, which can affect UE4SS compatibility and assumptions made by older mods.

UE4SS provides useful reverse-engineering facilities including Lua scripting, UObject reflection, hooks, property inspection and SDK/header dumping.

## Current confirmed runtime findings

🟢 During live inventory/weapon research we have confirmed that:

- an inventory-add path exposes an item data asset plus a UID;
- an item UID can be resolved to a live inventory slot object;
- the resolved object can expose an `ItemStats` array;
- those stat entries correspond to values visible in-game;
- GameplayTag-backed stat identities can be decoded at runtime;
- the game's own stat-update function can change live weapon stats;
- modified weapon stats are subsequently serialized by SurrounDead into its save data;
- item UIDs observed during this work remain useful across save/reload for associating external metadata with the same item instance.

See [Runtime weapon-stat research](../research/runtime-weapon-stats.md) for the detailed evidence.

## Important distinction

Some of these discoveries arose while diagnosing an existing third-party mod that was incompatible with the current game/runtime environment. The third-party mod itself is not part of this repository. Only independently observed information about SurrounDead's runtime interfaces is documented here.

## Research direction

🔵 Useful next targets include reliable equipped-weapon identification, enemy damage/death events, kill attribution, equipment-slot semantics and the relationship between live inventory objects and their serialized `PlayerInfo.sav` counterparts.
