# Discovery Log

A dated record of notable findings. This exists so that changing conclusions remain traceable rather than being silently rewritten.

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
