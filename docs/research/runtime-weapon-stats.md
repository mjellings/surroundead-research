# Runtime Weapon-Stat Research

> This page documents **SurrounDead runtime behaviour** discovered while investigating and building original UE4SS tooling against SurrounDead 0.8 / Unreal Engine 5.6. Third-party mod source/implementation is not reproduced here.

## Physical weapon identity

🟢 **Confirmed** — `/Game/JigSInventory/Jigsaw/Components/BP_JigHelperComp.BP_JigHelperComp_C:GetEquipmentUID` exposes an `FGuid` for the currently used equipment slot. Immediately before the firearm damage path, this GUID identifies the specific physical weapon instance rather than merely the weapon type.

Different instances of the same weapon type have produced different GUIDs, while the same physical weapon retains its identity across the tested save/reload cycle.

This provides a reliable key for per-weapon progression metadata.

## Damage, death and vanilla XP attribution

🟢 **Confirmed** — the following runtime events can be combined to attribute a kill and the game's awarded XP to the physical firearm that caused it:

```text
BP_JigHelperComp:GetEquipmentUID
        ↓
physical weapon FGuid
        ↓
BP_FirearmPickup:SERVER_DamageEvent
        ↓
damaged enemy
        ↓
BP_MasterZombie:Death
        ↓
LevellingComponent:AddXP
```

The XP value observed at `AddXP` is the vanilla XP supplied by the game. This makes it possible for an external progression system to mirror the game's kill XP without changing player XP.

🟢 Per-physical-weapon XP, kill counts, levels, configurable thresholds, overflow and restart persistence have all been demonstrated in testing.

🟡 The upstream source of the game's XP scaling is not yet fully mapped. Separate environments have exposed `XpMultiplierCalc` ranges around 4–8 and 40–60, with correspondingly different vanilla `AddXP` values. Difficulty/world settings are a likely influence but are not yet confirmed as the exact source.

## Resolving a live inventory item

🟢 **Confirmed for freshly added inventory items** — an inventory-add event exposes an item data asset, UID and the player's `BP_JigMultiplayer` component. With a short delay, `FindItemByUID` can resolve that fresh UID to a healthy live `JSI_Slot_C` whose `ItemUniqueID` matches the supplied GUID and whose `ItemStats` are readable.

⚠️ **Important correction from later testing:** a non-nil `Found` result from `FindItemByUID` is not sufficient proof that the returned UObject is safe to dereference. When the stable equipment GUID of an already-equipped Crusher was passed through this route, UE4SS returned a non-nil UObject wrapper but its underlying object was not valid for normal member access. Even UObject identity/member calls could cause a native access violation, which `pcall` cannot catch.

Therefore do not treat `Found ~= nil` as equivalent to "valid live slot" for arbitrary/equipped items.

## Reliable live-slot bridge for an equipped weapon

🟢 **Confirmed** — the equipped physical weapon GUID can instead be mapped to the real live inventory widget by enumerating live `JSI_Slot_C` objects and comparing their proven-safe `ItemUniqueID` field.

The tested route is:

```text
GetEquipmentUID
        ↓
physical weapon FGuid
        ↓
FindAllOf("JSI_Slot_C")
        ↓
read ItemUniqueID on live slots
        ↓
exact GUID match
        ↓
real live weapon JSI_Slot_C
        ↓
ItemStats
```

This bypasses the unsafe/stale `FindItemByUID` wrapper observed for the already-equipped weapon.

🟢 The same Crusher slot was found repeatedly by this method before and during firing, so the result was not a one-off lookup coincidence.

## `ItemStats`

🟢 **Confirmed** — the matched live weapon slot exposes an `ItemStats` TArray whose values correspond to the weapon's current runtime statistics.

The serialized/property names encountered at runtime include UE-generated/hash-suffixed forms such as a `STAT_NAME` field and a `MinValue` field. This is especially interesting because the save-file research independently identifies `S_ItemStat`, `STAT_NAME`, `MinValue` and related data in `PlayerInfo.sav`.

This provides a useful bridge between the **serialized item representation** and the **live UObject representation**.

A controlled Crusher test exposed five readable entries:

```text
Jig.Stat.FirearmDamage
Jig.Stat.CriticalHitMultiplier
Jig.Stat.CriticalHitChance
Jig.Stat.FirearmRPM
Jig.Stat.DamageFallOff
```

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

## Updating stats — first independently verified controlled write

🟢 **Confirmed 2026-09-04** — an exact physical Crusher was located through the live-slot bridge above, its exact `Jig.Stat.FirearmDamage` entry was selected, and the game's `BP_JigComponent:UpdateStatByUID` path was called with the same physical weapon GUID and GameplayTag.

The controlled test changed only FirearmDamage by +1, once during the run:

```text
FirearmDamage: 93.0 → 94.0
```

A separate live-slot scan approximately 250 ms after the update independently read the value back as `94.0`. The other four observed firearm stats remained unchanged in that verification scan.

This confirms the full runtime bridge needed for per-physical-weapon stat progression:

```text
weapon fires
→ GetEquipmentUID
→ physical weapon GUID
→ match live JSI_Slot_C.ItemUniqueID
→ locate ItemStats entry by GameplayTag
→ UpdateStatByUID
→ re-read changed value from live slot
```

## Persistence

🟢 **Confirmed from earlier runtime/save testing** — weapon stat modifications made through the live game path can survive a save/restart cycle. SurrounDead itself serializes resulting `ItemStats` values into the save.

🟢 **Confirmed in tested saves** — item UIDs were sufficiently stable across reloads to associate separate metadata with the same weapon instance.

This leads to a useful architectural lesson for future original mods/tools: if the game already persists modified item stats, external persistence should avoid blindly reapplying changes after reload, otherwise values can compound.

## Inventory-add hook: current role

🟢 `JigTryAddItemSomewhere` was invaluable during reverse engineering because it supplied a known-good fresh slot/UID and exposed the player's `BP_JigMultiplayer` component.

🔵 It should not need to remain a user-facing prerequisite for weapon progression. The remaining architectural cleanup is to resolve the player's `BP_JigMultiplayer` directly, allowing normal progression/stat updates without requiring an inventory-add event first. The inventory-add hook can then remain as a diagnostic/fallback route.

## Open research

🔵 Resolve the player's `BP_JigMultiplayer` component directly without relying on an inventory-add event.

🔵 Turn the proven controlled stat mutation into a generic level-up upgrade system with configurable stat choices/increments/caps.

🔵 Determine the exact source of vanilla XP scaling/world-setting differences.

🔵 Map additional GameplayTags and the full `S_ItemStat` structure.

🔵 Strengthen the mapping between live `JSI_Slot_C` objects and their exact serialized `ItemInfo` records.
