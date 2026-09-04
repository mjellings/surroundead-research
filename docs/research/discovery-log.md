# Discovery Log

A dated record of notable findings. This exists so that changing conclusions remain traceable rather than being silently rewritten.

## 2026-09-04

### Runtime kill and XP pipeline

🟢 Confirmed exact vanilla runtime paths for `BP_FirearmPickup:SERVER_DamageEvent`, `BP_MasterZombie:Death`, `LevellingComponent:AddXP` and `GameFunctionLibrary:XpMultiplierCalc`.

🟢 For tested firearm kills, observed callback order is `XpMultiplierCalc → AddXP → Death → SERVER_DamageEvent`. The killed zombie can be correlated by UObject identity and the Death callback identifies the player as killer in the tested path.

🟢 Vanilla zombie XP is a continuous floating-point value; tested 4–8 ranges produced non-integer values. This does not establish firearm stat-roll behaviour.

### Equipped weapon and UID bridge

🟢 Confirmed `JSIContainer:GetEquippedItemRef` and established that the top-level `CPrimary` `JSI_Slot_C` is the equipped weapon slot. Nested magazine/chamber slots also invoke inventory functions, so naïvely caching the latest slot is unsafe.

🟢 Confirmed vanilla `JSI_Slot:GetUniqueID` returns `UniqueServerID : FGuid`.

🟢 Confirmed `/Script/Engine.KismetGuidLibrary:Conv_GuidToString` converts the FGuid, and UE4SS `FString:ToString()` exposes the actual Lua string. Generic `tostring()` only exposes temporary wrapper/address-like values.

🟢 A tested Crusher consistently resolves to `19BB6DEF-481C-1781-72EF-62A20CFED911`, including after a full game restart and reload of the same save. This establishes a persistent per-item identity for the tested lifecycle.

### Weapon Progression persistence

🟢 Completed the first persistent progression vertical slice: confirmed zombie kill → persistent equipped-weapon GUID → exact vanilla XP → per-weapon record → `data.db`.

🟢 `data.db` persistence was verified across a complete game restart. The tested Crusher continued from 2 kills / 10.094894224405 XP to 3 kills / 16.650929206609 XP using the same GUID.

🟢 High-frequency runtime hooks are now deliberately silent. Verbose probe logging was observed to affect game responsiveness, so normal logging is limited to startup/readiness, confirmed awards, level-ups and errors.

🔴 Earlier direct reflected UID-property reads returned `TrivialObject`/wrapper representations rather than GUID contents. Likewise, generic `tostring()` on an FString returned changing wrapper values. These approaches are superseded and their outputs must not be persisted.

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
