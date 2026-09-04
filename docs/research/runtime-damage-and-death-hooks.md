# Runtime Damage, Death, XP and Weapon Identity

Current findings from SurrounDead 0.8 / Unreal Engine 5.6 using UE4SS v3.0.1 zDEV.

Confidence markers: 🟢 confirmed, 🟡 probable, 🔵 research, 🔴 failed/obsolete.

## Confirmed vanilla runtime functions

🟢 Firearm damage event:

`/Game/Inventory/Items/Pickups/Weapons/Firearms/BP_FirearmPickup.BP_FirearmPickup_C:SERVER_DamageEvent`

Observed parameters include `Headshot : Bool`, `DamagedActor : Object`, and `ImpactPoint : FVector`. The function context identifies the firearm actor that caused the damage.

🟢 Zombie death:

`/Game/AI/Zombies/BP_MasterZombie.BP_MasterZombie_C:Death`

The tested player-firearm kill path supplies the player actor to `Actor` and the killed zombie is the function context.

🟢 Vanilla XP award:

`/Game/Blueprints/Components/LevellingComponent.LevellingComponent_C:AddXP`

The `XP` input provides the exact player XP awarded for the tested kill. Later tests under different game/difficulty settings produced substantially larger values than the original 4–8 observations; in-game verification confirmed these larger values were the actual XP awarded by SurrounDead. Weapon Progression therefore mirrors the game's final awarded XP rather than reproducing its difficulty/multiplier calculation.

🟢 XP calculation:

`/Game/Blueprints/GameFunctionLibrary.GameFunctionLibrary_C:XpMultiplierCalc`

Observed zombie XP can be continuous floating-point data. This does not establish how firearm DataAsset stat ranges are rolled.

## Confirmed callback correlation

🟢 For tested killing firearm shots the observed sequence is:

`XpMultiplierCalc → AddXP → BP_MasterZombie:Death → BP_FirearmPickup:SERVER_DamageEvent`

🟢 UObject identity can correlate the zombie in `Death` with `DamagedActor` in the surrounding firearm damage event. This provides a practical confirmed-kill boundary rather than awarding progression for every hit.

🟢 Repeated non-lethal damage events do not award weapon kills. In testing, a weapon could hit the same zombie repeatedly and received one kill/XP award only when the correlated death occurred.

## Equipment containers and why passive slot tracking was superseded

🟢 Vanilla equipped-item retrieval:

`/Game/JigSInventory/Jigsaw/Widgets/JSIContainer.JSIContainer_C:GetEquippedItemRef`

🟢 Confirmed top-level firearm equipment containers:

- Primary: `CPrimary`
- Secondary: `CSecondary`
- Sidearm: `CPistol`

Firearms may also contain nested magazine/chamber slots. Passive `GetEquippedItemRef`/slot tracking is therefore unsafe because nested calls and equipment refreshes can replace or refresh multiple slot objects during weapon switching.

Observed hierarchy:

```text
BP_Inventory_C
  └─ CPrimary / CSecondary / CPistol
       └─ JSI_Slot_C              <- equipped weapon
            └─ firearm container
                 ├─ MagContainer
                 │    └─ JSI_Slot...
                 └─ ChamberContainer
                      └─ JSI_Slot...
```

The earlier CPrimary-specific bridge was useful for proving persistent identity, but it is no longer the preferred attribution mechanism.

## Persistent weapon UID

🟢 Exact vanilla slot UID getter:

`/Game/JigSInventory/Jigsaw/Widgets/JSI_Slot.JSI_Slot_C:GetUniqueID`

Output: `UniqueServerID : FGuid`.

🟢 Unreal GUID conversion:

`/Script/Engine.KismetGuidLibrary:Conv_GuidToString`

Input: `InGuid : FGuid`; output: `ReturnValue : FString`.

🟢 UE4SS `FString` wrappers must be converted with `FString:ToString()` to obtain the actual Lua string. Generic Lua `tostring()` only exposed changing wrapper/address-like values and must not be persisted as identity.

### Generic equipment UID bridge

🟢 The game's own equipment lookup provides a cleaner generic firearm attribution route:

`/Game/JigSInventory/Jigsaw/Components/BP_JigHelperComp.BP_JigHelperComp_C:GetEquipmentUID`

Parameters observed in the object dump:

- `Slot : GameplayTag`
- `Value : FGuid`

The function is used by firearm durability paths including `BP_FirearmPickup:DurabilityCheck` and `BP_FirearmPickup:ReduceDurability`.

Runtime testing showed `GetEquipmentUID` immediately preceding known-good firearm damage activity and returning a stable FGuid that changed correctly when switching physical weapons. Tested mappings in one session included:

- Crusher → `19BB6DEF-481C-1781-72EF-62A20CFED911`
- Benelli M4 → `740F4C2F-4329-7F16-F907-A18F69C8BF53`
- Battle Ready Glock → `18E57E18-40C7-D733-8CFD-17B43F84391A`

The Crusher value exactly matched the GUID independently established through `CPrimary → JSI_Slot:GetUniqueID` and across a full game restart. This cross-validation strongly confirms `GetEquipmentUID` returns the persistent physical weapon instance FGuid rather than an ammo/chamber/transient identifier.

🟢 Generic firearm → persistent weapon GUID attribution is therefore confirmed for tested Primary, Secondary and Sidearm weapon paths without hardcoding those equipment containers into the kill-award path.

A practical implementation caches the recent equipment UID against the firearm actor observed by `SERVER_DamageEvent`. A short recency guard remains sensible defensive engineering in case unrelated equipment UID queries occur between firearm operations.

## `TryUseBullet_UID` correction

🔴 `TryUseBullet_UID` is not a UFunction. Earlier attempts to register it as one were based on misreading object-dump locals/output names.

The actual function is:

`/Game/JigSInventory/Jigsaw/Widgets/JSI_Slot.JSI_Slot_C:TryUseBullet`

Observed outputs include `Return : Bool`, `UID : FGuid`, and `ItemUsed : Object`. `BP_FirearmPickup:FireBullet` contains locals such as `CallFunc_TryUseBullet_UID`. This path appears related to ammunition consumption and is not used as the persistent firearm identity bridge.

## Weapon Progression persistence and levelling proof

🟢 The working runtime pipeline is now:

`confirmed zombie kill → actual firearm actor → GetEquipmentUID persistent FGuid → exact vanilla awarded XP → per-weapon progression record → data.db`

🟢 The progression record stores weapon UID, progression level, current-level XP, kill count and weapon name.

🟢 Weapon XP mirrors the exact final XP awarded by SurrounDead for the kill. This naturally incorporates whatever difficulty/world/enemy modifiers the game applies upstream without Weapon Progression needing to reproduce them.

🟢 Current progression threshold formula tested successfully:

`XPRequired(level) = 100 + ((level - 1) * 50)`

This gives 100 XP for Level 1→2, 150 for 2→3, 200 for 3→4, 250 for 4→5, etc.

🟢 Level transitions preserve overflow XP. Testing progressed a Crusher through several consecutive levels rather than merely proving Level 2.

🟢 A complete SurrounDead restart then loaded the existing progression database and recognised the same Crusher as already Level 4. Continued kills resumed from the persisted Level 4 XP/kill state. The same run subsequently crossed Level 4→5, consuming 250 XP and preserving 21.744 overflow; the resulting kill record was Level 5 with 21.744/300 XP and 12 kills, with the database save reporting success.

This confirms, for the tested lifecycle:

- persistent physical weapon identity across restart
- Primary/Secondary/Sidearm attribution
- confirmed-kill-only awards
- exact vanilla kill-XP mirroring
- per-weapon XP and kill counts
- increasing level thresholds
- overflow preservation
- multiple consecutive level transitions
- database persistence
- full game restart persistence
- continued progression after restart

Weapon stat mutation remains deliberately disabled in this milestone. The next research step is to connect a confirmed level-up to the game's UID-based stat update path and verify that the modified stat belongs to the exact weapon and survives a save/restart.

## Hook registration timing

🟢 Blueprint UFunctions may not exist in memory when a Lua mod's `main.lua` first executes. Immediate registration of known-valid Weapon Progression functions failed during integration even though those same functions were confirmed by dumps and earlier probes.

The working approach is delayed registration with `ExecuteWithDelay`, retrying missing hooks individually until the Blueprint functions become available. A startup `no UFunction with the specified name was found` error is therefore not, by itself, evidence that a Blueprint path is invalid.

## Failed and superseded approaches

🔴 Reading `UniqueID`, `UniqueServerID`, `ReplicationUID` and similar reflected properties directly produced UE4SS `TrivialObject`/wrapper representations rather than usable GUID values.

🔴 Generic `tostring()` on the `FString` returned by `Conv_GuidToString` produced wrapper/address-like strings that changed between calls. These were never stable weapon IDs.

🔴 A passive hook on `Conv_GuidToString` alone did not fire during the desired weapon path because that probe removed the active/natural UID conversion route.

🔴 Calling `slot:GetUniqueID()` directly from arbitrary Lua callback context failed in later probes. Earlier successful UID extraction depended on observing the game's own getter/output path; direct invocation should not be assumed safe.

🔴 Slot-object change correlation is ambiguous. Switching weapons can refresh more than one top-level slot object, so a changed slot is not sufficient evidence that it is the weapon that fired.

🔴 A FireBullet pre/post bridge was not reliable in the tested runtime even though the function can be hookable. `SERVER_DamageEvent` is the proven firearm activity boundary used by the current pipeline.

🔴 `TryUseBullet_UID` was an output/local field, not a hookable UFunction. The real `JSI_Slot:TryUseBullet` path is ammunition-oriented and is not the chosen weapon identity route.
