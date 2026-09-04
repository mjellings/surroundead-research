# Discovery Log

A dated record of notable findings. This exists so that changing conclusions remain traceable rather than being silently rewritten.

## 2026-09-04

### Runtime kill and XP pipeline

🟢 Confirmed exact vanilla runtime paths for `BP_FirearmPickup:SERVER_DamageEvent`, `BP_MasterZombie:Death`, `LevellingComponent:AddXP` and `GameFunctionLibrary:XpMultiplierCalc`.

🟢 For tested firearm kills, observed callback order is `XpMultiplierCalc → AddXP → Death → SERVER_DamageEvent`. The killed zombie can be correlated by UObject identity and the Death callback identifies the player as killer in the tested path.

🟢 Vanilla kill XP can be continuous floating-point data. Later testing under different game/difficulty settings produced substantially larger XP awards than the initial 4–8 observations; in-game verification confirmed the captured `AddXP` value is the XP SurrounDead actually awards. Weapon Progression therefore mirrors the game's final XP rather than reproducing difficulty/multiplier rules.

🟢 Repeated hits on one zombie do not produce repeated progression awards: the correlated death boundary awards one weapon kill and one XP award when the enemy actually dies.

### Equipped weapon and UID bridge

🟢 Confirmed exact top-level firearm equipment containers: Primary=`CPrimary`, Secondary=`CSecondary`, Sidearm=`CPistol`.

🟢 Earlier `JSIContainer:GetEquippedItemRef` work established the top-level weapon slot and nested magazine/chamber problem, but weapon switching can refresh multiple slot objects. Slot-object change correlation is therefore not reliable enough for generic attribution.

🟢 Confirmed vanilla `JSI_Slot:GetUniqueID` returns `UniqueServerID : FGuid` and `/Script/Engine.KismetGuidLibrary:Conv_GuidToString` converts it. UE4SS `FString:ToString()` exposes the actual Lua string; generic `tostring()` only exposes temporary wrapper/address-like values.

🟢 A tested Crusher consistently resolved to `19BB6DEF-481C-1781-72EF-62A20CFED911`, including after a full game restart and reload of the same save. This established persistent per-item identity for the tested lifecycle.

### Generic firearm attribution breakthrough

🟢 Object-dump inspection identified the game's own equipment UID lookup:

`/Game/JigSInventory/Jigsaw/Components/BP_JigHelperComp.BP_JigHelperComp_C:GetEquipmentUID`

It accepts `Slot : GameplayTag` and returns `Value : FGuid`, and is used by firearm durability paths.

🟢 Runtime testing showed the returned GUID tracks the physical weapon being used. Tested mappings included Crusher=`19BB6DEF-481C-1781-72EF-62A20CFED911`, BenelliM4=`740F4C2F-4329-7F16-F907-A18F69C8BF53`, and BattleReadyGlock=`18E57E18-40C7-D733-8CFD-17B43F84391A`.

🟢 The Crusher value exactly matched the independently established persistent `CPrimary → GetUniqueID` GUID. This cross-validates `GetEquipmentUID` as the physical weapon instance identity rather than an ammo/chamber/transient GUID.

🟢 Generic firearm→persistent-GUID attribution is therefore confirmed across tested Primary, Secondary and Sidearm paths. The kill pipeline no longer needs to hardcode `CPrimary` to decide which weapon receives XP.

### `TryUseBullet_UID` correction

🔴 `TryUseBullet_UID` is not a UFunction. It is an output/local name associated with the actual `/Game/JigSInventory/Jigsaw/Widgets/JSI_Slot.JSI_Slot_C:TryUseBullet` function. The latter outputs `Return`, `UID` and `ItemUsed` and appears ammunition-oriented. Earlier guessed `TryUseBullet_UID` hook paths are obsolete.

### Weapon Progression — multi-weapon persistence

🟢 Completed the generic persistent progression pipeline:

`confirmed zombie kill → actual firearm actor → GetEquipmentUID FGuid → exact vanilla awarded XP → per-weapon record → data.db`

🟢 Separate tested firearms produced separate progression records, and repeated non-lethal shots did not inflate kill counts.

### Weapon Progression — levelling milestone

🟢 Confirmed increasing progression thresholds using `100 + ((level - 1) * 50)`: Level 1→2 consumes 100 XP, 2→3 consumes 150, 3→4 consumes 200, and 4→5 consumes 250.

🟢 Overflow XP is preserved correctly across level transitions. A single test sequence progressed a Crusher through multiple levels rather than merely proving the first Level 2 transition.

🟢 Full restart persistence is confirmed for multi-level progression. After completely restarting SurrounDead, the existing database was loaded and the same Crusher was recognised as already Level 4. Progression then continued from the saved XP/kill state.

🟢 In the post-restart run, the Crusher continued to 217.560/250 XP at 11 kills, then the next kill awarded 54.185 XP, crossed Level 4→5, consumed the 250-XP threshold and preserved 21.744 overflow. The resulting record was Level 5, 21.744/300 XP, 12 kills, with `saved=true`.

🟢 This confirms the core levelling engine: persistent weapon identity, actual-weapon attribution, confirmed-kill awards, vanilla XP mirroring, per-weapon XP/kills, escalating thresholds, overflow, multiple levels, database persistence, full restart persistence and continued progression after restart.

🔵 Weapon stat mutation remains intentionally disabled. The next milestone is one controlled stat upgrade on level-up through the game's UID-based stat update path, followed by save/restart verification on the exact physical weapon.

### Hook registration timing

🟢 Integration testing exposed an important UE4SS timing rule: known-valid Blueprint UFunctions may not yet exist in memory when a Lua mod first executes. Immediate registration failed for several confirmed Weapon Progression functions.

🟢 Delayed/retried registration using `ExecuteWithDelay` solved the issue. Missing hooks are retried individually until available. An early `no UFunction with the specified name was found` message is not sufficient evidence that a Blueprint path is wrong.

### Useful negative findings

🔴 Direct reflected UID-property reads returned `TrivialObject`/wrapper representations rather than GUID contents. Generic `tostring()` on an FString likewise returned changing wrapper values.

🔴 Direct arbitrary Lua invocation of `slot:GetUniqueID()` failed in later probes; the successful route relied on the game's own execution/output path.

🔴 FireBullet pre/post correlation was not reliable in the tested runtime. `SERVER_DamageEvent` remains the proven firearm activity boundary.

🔴 Slot-object changes during switching are ambiguous because multiple top-level slots may refresh together.

See `runtime-damage-and-death-hooks.md` for the consolidated technical details.

## 2026-09-03

### Save-file documentation consolidated

🟢 Consolidated current SurrounDead 0.8 / UE5.6 save research into a single public research repository.

🟢 Recorded the observed save folder layout and the roles currently attributed to `PlayerInfo.sav`, `Player.sav` and `Slot.sav`.

🟢 Recorded the Oodle wrapper magic and observed 49-byte block header layout, including the distinction between fixed block-size metadata and compressed/uncompressed size fields.

🟡 Documented current UE5.6 property-stream quirks, including variable `StructProperty` assumptions and versioned struct-prefix observations.

### Inventory/container research

🟢 Documented `ItemInfo` / `RepItemInfo`-style records and observed identity, placement, stat and container fields in `PlayerInfo.sav`.

🟡 Updated the working model for `MainJigContainers`: its declared ArrayProperty count should not automatically be interpreted as that many independent top-level serialized `S_ReplicatedContainerInfo` objects. Examined data instead suggests repeated inner container sections within a much larger region.

### Live runtime / save bridge

🟢 Recorded that a live inventory item can be resolved by UID to a `JSI_Slot_C` object exposing `ItemStats`.

🟢 Recorded runtime GameplayTags observed for firearm damage, critical values, RPM and damage falloff.

🟢 Confirmed that changes made through the game's live UID/stat update path are subsequently persisted by SurrounDead itself in save data.

🟢 Confirmed in tested saves that item UIDs can remain useful across reloads for associating metadata with the same item instance.

### Vanilla firearm DataAssets

🟢 Analysed the clean SurrounDead 0.8 `Weapons/Firearms` DataAsset export containing 49 firearm definitions.

🟢 Conventional firearm definitions contain paired floating-point bounds corresponding to `FirearmDamage`, `CriticalHitMultiplier`, `CriticalHitChance`, `FirearmRPM` and `DamageFallOff`.

🟢 Cross-checked `DA_Crusher`: cooked ranges are Damage 54–64, Crit Multiplier 24–29, Crit Chance 12–22, RPM 600–700 and Falloff 110–120. A previously observed live Crusher instance (63 / 28 / 18 / 645 / 117) falls inside every corresponding range, validating the conventional record ordering and strongly supporting the interpretation as instance-generation bounds.

🟡 The current working model is that firearm instances roll/generate their live stats from these bounds. The exact distribution, rounding behaviour and any rarity-stage modifiers remain to be established.

🔵 Special weapons such as the Crossbow, Grenade Launcher, Flare Gun and Rocket Launcher do not all follow the ordinary five/four-record layout and require separate mapping.

### Failed / useful negative findings

🔴 A direct property-iteration attempt against a live slot's `CustomData` did not expose the expected structure because the value surfaced through UE4SS as a `TrivialObject`. The approach was abandoned rather than treating the result as evidence that CustomData is empty.

🔴 A save-write experiment demonstrated that rewriting the Oodle wrapper `BlockSize` based on compressed payload size corrupts the file. In the tested format the block-size field remained `131072`; relevant compressed/uncompressed size fields are separate.

### Third-party-mod boundary

🟢 Established a publication rule for this repository: third-party mods may be mentioned as historical research context, but their source, assets and original implementation are not to be presented as this project's work or redistributed without appropriate permission/licensing.
