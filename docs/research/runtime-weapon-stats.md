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

## Resolving the player's live Jig component

🟢 **Confirmed** — the player's live `BP_JigMultiplayer` component can be resolved directly from `BP_PlayerCharacter_C` without requiring an inventory-add event first.

Tested route:

```text
FindFirstOf("BP_PlayerCharacter_C")
        ↓
player.BP_JigMultiplayer
        ↓
BP_JigComponent_C
```

This component is valid for `UpdateStatByUID` when paired with the correct native UID wrapper described below.

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
physical weapon FGuid value
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

🟢 The same physical Crusher slot was found repeatedly by this method before and during firing.

## `ItemStats`

🟢 **Confirmed** — the matched live weapon slot exposes an `ItemStats` TArray whose values correspond to the weapon's current runtime statistics.

The serialized/property names encountered at runtime include UE-generated/hash-suffixed forms such as a `STAT_NAME` field and a `MinValue` field. This is especially interesting because the save-file research independently identifies `S_ItemStat`, `STAT_NAME`, `MinValue` and related data in `PlayerInfo.sav`.

This provides a useful bridge between the **serialized item representation** and the **live UObject representation**.

A conventional Crusher exposed five readable entries:

```text
Jig.Stat.FirearmDamage
Jig.Stat.CriticalHitMultiplier
Jig.Stat.CriticalHitChance
Jig.Stat.FirearmRPM
Jig.Stat.DamageFallOff
```

A Hunting Rifle exposed only four in testing and had no `FirearmRPM` entry. Progression code must therefore treat the supported stat set as variable per physical weapon rather than assuming all firearms have five entries.

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

## Updating stats — native UID wrapper requirement

🟢 **Confirmed 2026-09-05** — `BP_JigComponent:UpdateStatByUID` is sensitive not only to the GUID value but to the native `FGuid` wrapper supplied to it.

A controlled A/B test used the same Crusher GUID value through two different wrappers:

- `GetEquipmentUID` returned an `FGuid` wrapper that correctly identified the physical weapon but **did not mutate** its stat.
- The matching live `JSI_Slot_C.ItemUniqueID` wrapper successfully changed the stat and independent delayed readback confirmed the new value.

This yields the current known-good mutation route:

```text
weapon fires
→ GetEquipmentUID
→ physical weapon GUID value
→ direct player BP_JigMultiplayer
→ FindAllOf("JSI_Slot_C")
→ exact ItemUniqueID GUID-value match
→ retain that live slot.ItemUniqueID FGuid wrapper
→ locate ItemStats entry by GameplayTag
→ BP_JigComponent:UpdateStatByUID(slot.ItemUniqueID, tag, target)
→ independent delayed live readback
```

The practical rule is:

> `GetEquipmentUID` tells us **which weapon**. `JSI_Slot_C.ItemUniqueID` supplies the **native FGuid wrapper required to modify it**.

## Weapon Progression stat model

🟢 **Confirmed 2026-09-05** — the original Weapon Progression research build now supports permanent per-physical-weapon stat rewards driven by weapon level-ups.

Current tested reward set:

```text
Damage                 +2%
Critical Hit Chance    +1 point
Critical Hit Multiplier +2 points
RPM                    +2%
Damage Falloff         +2%
```

Percentage stats are reconstructed deterministically from their captured base value and upgrade count rather than by repeatedly multiplying the current live value. Point-based stats use base + points × count.

🟢 One eligible stat is awarded per weapon level, saved before runtime mutation, and the same stat is not selected on consecutive levels when alternatives exist.

🟢 Multiple level-ups, overflow XP, per-weapon kill counts and XP all use the same physical GUID record.

## Persistence — corrected model

🔴 **Correction to the earlier working assumption:** runtime stat writes made through `UpdateStatByUID` were later shown **not to survive a full SurrounDead restart by themselves** in the tested path. After restart, the same physical Crusher GUID returned with its original rolled/base values.

🟢 **Confirmed 2026-09-05** — Weapon Progression therefore uses **mod-owned persistence**. `data.db` is authoritative for the earned upgrade layer, while SurrounDead supplies the weapon's original rolled/base stats after a full restart.

The proven restart sequence is:

```text
original live weapon stats
→ capture base stats once
→ earn XP and levels
→ persist upgrade counters in data.db
→ mutate live weapon
→ full game exit
→ restart same save
→ same physical GUID resolves
→ game supplies original/base values
→ Weapon Progression reconstructs targets from base + saved upgrade counters
→ UpdateStatByUID using live slot.ItemUniqueID wrapper
→ delayed verification confirms reconstructed values
```

🟢 Full restart reconstruction has been independently verified for the same Crusher, including subsequent post-restart XP gain, another level-up and another permanent stat reward.

## Important `data.db` reset rule

⚠️ **Confirmed testing caveat — 2026-09-05:** deleting/resetting `data.db` while SurrounDead is still running does **not** revert stat mutations already present on live weapon objects in memory.

If the mod is then reloaded without restarting the game, a fresh database has no prior base-stat record and will correctly-but-unhelpfully capture the currently modified live values as the new base values. Any subsequent progression will therefore stack from those already-upgraded values.

Example from a deliberate test reset: the same Crusher's fresh database captured modified runtime values such as Damage `94.86`, Critical Multiplier `29`, RPM `977.67` and Damage Falloff `85.833`, rather than its earlier original values.

Therefore:

> **When deliberately resetting Weapon Progression's `data.db`, fully exit and restart SurrounDead before allowing the mod to recapture weapon base stats.**

This rule applies specifically to resetting persistence. Using UE4SS **Reload All Mods** for normal `main.lua` development/testing is fine and has been used successfully; it simply does not recreate the game's live weapon objects or revert their current runtime stats.

## Verification behaviour

🟢 A delayed independent `JSI_Slot_C` readback verifies each expected target after mutation.

🟢 Verification scheduling is debounced per physical weapon GUID. Multiple stat writes during one reconstruction pass now schedule one verification scan rather than one scan per modified stat.

## Native level-up notifications

🟢 **Confirmed 2026-09-05** — Weapon Progression can use SurrounDead's native right-side notification UI.

The safe text path is:

```text
Lua string
→ /Script/Engine.KismetTextLibrary:Conv_StringToText
→ valid Unreal FText
→ GameFunctionLibrary:CreateNotificationUI
→ HUD_Game:Notification
→ native SurrounDead notification
```

Directly passing a Lua string into an Unreal `FText` parameter caused a native UE4SS access violation during testing and should be avoided.

A production test successfully displayed dynamically generated messages such as:

```text
Crusher reached Level 8 - Critical Multiplier +2
```

## Inventory-add hook: current role

🟢 `JigTryAddItemSomewhere` was invaluable during reverse engineering because it supplied a known-good fresh slot/UID and exposed the player's `BP_JigMultiplayer` component.

🟢 It is no longer required for normal Weapon Progression operation. Direct player-Jig resolution plus the live `JSI_Slot_C` scan now provide the production path.

## Open research

🔵 Determine the exact source of vanilla XP scaling/world-setting differences.

🔵 Map additional GameplayTags and unusual/special-weapon stat layouts.

🔵 Strengthen the mapping between live `JSI_Slot_C` objects and their exact serialized `ItemInfo` records.

🔵 Test the production progression path across more firearm families, repeated weapon switching, long play sessions and additional full restart cycles.
