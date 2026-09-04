# Runtime Damage, Death, XP and Weapon Identity

Current findings from SurrounDead 0.8 / Unreal Engine 5.6 using UE4SS v3.0.1 zDEV.

Confidence markers: 🟢 confirmed, 🟡 probable, 🔵 research, 🔴 failed/obsolete.

## Confirmed vanilla runtime functions

🟢 Firearm damage event:

`/Game/Inventory/Items/Pickups/Weapons/Firearms/BP_FirearmPickup.BP_FirearmPickup_C:SERVER_DamageEvent`

Observed parameters include `Headshot : Bool`, `DamagedActor : Object`, and `ImpactPoint : FVector`.

🟢 Zombie death:

`/Game/AI/Zombies/BP_MasterZombie.BP_MasterZombie_C:Death`

The tested player-firearm kill path supplies the player actor to `Actor` and the killed zombie is the function context.

🟢 Vanilla XP award:

`/Game/Blueprints/Components/LevellingComponent.LevellingComponent_C:AddXP`

The `XP` input provides the exact player XP awarded for the tested kill.

🟢 XP calculation:

`/Game/Blueprints/GameFunctionLibrary.GameFunctionLibrary_C:XpMultiplierCalc`

Observed zombie XP is a continuous floating-point value. For example, a 4–8 range produced values such as 7.907284784317 and 7.6272164583206. This does not establish how firearm DataAsset stat ranges are rolled.

## Confirmed callback correlation

🟢 For tested killing firearm shots the observed sequence is:

`XpMultiplierCalc → AddXP → BP_MasterZombie:Death → BP_FirearmPickup:SERVER_DamageEvent`

🟢 UObject identity can correlate the zombie in `Death` with `DamagedActor` in the surrounding firearm damage event. This provides a practical confirmed-kill boundary rather than awarding progression for every hit.

## Equipped weapon hierarchy

🟢 Vanilla equipped-item retrieval:

`/Game/JigSInventory/Jigsaw/Widgets/JSIContainer.JSIContainer_C:GetEquippedItemRef`

🟢 The player's top-level `CPrimary` container resolves to the `JSI_Slot_C` representing the equipped weapon.

🟢 Firearms may also contain nested inventory slots such as magazine and chamber containers. Therefore caching the most recent `GetEquippedItemRef` result is unsafe: nested calls can replace the actual weapon slot.

Observed hierarchy:

```text
BP_Inventory_C
  └─ CPrimary
       └─ JSI_Slot_C              <- equipped weapon
            └─ firearm container
                 ├─ MagContainer
                 │    └─ JSI_Slot...
                 └─ ChamberContainer
                      └─ JSI_Slot...
```

The runtime bridge therefore explicitly filters for the top-level `CPrimary` slot.

## Persistent weapon UID

🟢 Exact vanilla slot UID getter:

`/Game/JigSInventory/Jigsaw/Widgets/JSI_Slot.JSI_Slot_C:GetUniqueID`

Output: `UniqueServerID : FGuid`.

🟢 Unreal GUID conversion:

`/Script/Engine.KismetGuidLibrary:Conv_GuidToString`

Input: `InGuid : FGuid`; output: `ReturnValue : FString`.

🟢 UE4SS `FString` wrappers must be converted with `FString:ToString()` to obtain the actual Lua string. Generic Lua `tostring()` only exposed changing wrapper/address-like values and must not be persisted as identity.

🟢 The same FGuid type is used by the game's UID-based inventory/stat update paths, including `BP_JigComponent:ClientFunc_UpdateStatByUID`.

### Persistence proof

A specific tested Crusher produced the GUID:

`19BB6DEF-481C-1781-72EF-62A20CFED911`

🟢 Repeated captures during one runtime session returned the same GUID while temporary UE4SS struct-wrapper addresses changed.

🟢 After fully restarting SurrounDead, loading the same save and using the same Crusher, the same GUID was returned again.

This confirms the GUID is suitable for persistent per-item progression metadata in the tested save/item lifecycle.

## Weapon Progression persistence proof

🟢 A read/write Weapon Progression vertical slice now works:

`confirmed zombie kill → CPrimary weapon slot → persistent weapon GUID → vanilla XP → per-weapon progression record → data.db`

🟢 The progression record stores weapon UID, progression level, accumulated XP, kill count and weapon name.

🟢 Persistence across a complete game restart has been demonstrated. The tested Crusher progressed from two recorded kills and 10.094894224405 XP to three kills and 16.650929206609 XP after restarting the game and continuing with the same weapon.

🟢 The implementation deliberately keeps high-frequency runtime hooks silent. Logging is restricted to startup/readiness, confirmed XP awards, level-ups and errors because verbose UE4SS logging was observed to affect game responsiveness.

## Failed and superseded approaches

🔴 Reading `UniqueID`, `UniqueServerID`, `ReplicationUID` and similar reflected properties directly produced UE4SS `TrivialObject`/wrapper representations rather than usable GUID values.

🔴 Generic `tostring()` on the `FString` returned by `Conv_GuidToString` produced wrapper/address-like strings that changed between calls. These were never stable weapon IDs.

🔴 A passive hook on `Conv_GuidToString` alone did not fire during the desired weapon path because that probe removed the active/natural UID conversion route. The useful solution is to capture the FGuid from the vanilla `GetUniqueID` call and explicitly convert its returned FString with `ToString()`.
