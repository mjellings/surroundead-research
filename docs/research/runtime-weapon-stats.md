# Runtime Weapon-Stat Research

> This page documents **SurrounDead runtime behaviour** discovered while investigating compatibility of an existing third-party mod with SurrounDead 0.8. The mod itself is not authored by this repository's maintainer and its source/implementation is not reproduced here.

## Resolving a live inventory item

🟢 **Confirmed** — during an inventory-add event, SurrounDead exposes enough information to obtain an item UID. Passing that UID through the game's inventory lookup path can populate a result whose `Found` member is a live `JSI_Slot_C` UObject.

An important implementation observation is that a function's Lua return value is not necessarily the useful result: the supplied output structure may be populated even when the direct return is `nil`.

Freshly added items can also require a short retry window before the live slot becomes resolvable.

## `ItemStats`

🟢 **Confirmed** — the live slot exposes an `ItemStats` TArray whose values correspond to the weapon values displayed by the game.

The serialized/property names encountered at runtime include UE-generated/hash-suffixed forms such as a `STAT_NAME` field and a `MinValue` field. This is especially interesting because the save-file research independently identifies `S_ItemStat`, `STAT_NAME`, `MinValue` and related data in `PlayerInfo.sav`.

This provides a useful bridge between the **serialized item representation** and the **live UObject representation**.

## GameplayTags

🟢 **Confirmed** — stat identity can be resolved through GameplayTag/FName data rather than relying only on opaque generated property names.

Tags observed during firearm research include:

```text
Jig.Stat.FirearmDamage
Jig.Stat.CriticalHitMultiplier
Jig.Stat.CriticalHitChance
Jig.Stat.FirearmRPM
Jig.Stat.DamageFallOff
```

## Updating stats

🟢 **Confirmed** — SurrounDead exposes a UID-based stat-update path capable of changing live weapon values. Controlled tests changed multiple firearm statistics and those changes were reflected by the game.

The important research finding is the existence and behaviour of the game's update path; third-party mod implementation code is intentionally not included here.

## Persistence

🟢 **Confirmed** — weapon stat modifications made through the live game path can survive a save/restart cycle. SurrounDead itself serializes the resulting `ItemStats` values into the save.

🟢 **Confirmed in tested saves** — item UIDs were sufficiently stable across reloads to associate separate metadata with the same weapon instance.

This leads to a useful architectural lesson for future original mods/tools: if the game already persists the modified item stats, external persistence should avoid blindly reapplying multiplicative changes after reload, otherwise values can compound.

## Open research

🔵 Determine the most reliable way to identify the currently equipped weapon UID.

🔵 Identify reliable enemy damage/death events and establish weapon attribution for kills.

🔵 Map additional GameplayTags and the full `S_ItemStat` structure.

🔵 Strengthen the mapping between live `JSI_Slot_C` objects and their exact serialized `ItemInfo` records.
