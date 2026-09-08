# Weapon DataAsset Research

[Weapon reference — all tables and field explanations](weapons.md) · [Runtime stat updates](runtime-weapon-stats.md)

This page preserves extraction methods, validation evidence and open questions. Maintain weapon values in the [weapon reference](weapons.md).

> Vanilla SurrounDead 0.8 / UE5.6 cooked-asset research. This page records derived observations only; no game assets are redistributed.

## Source and scope

A clean 0.8 installation was opened with FModel and the game's `Content/Inventory/Items/DataAssets` tree was exported for local analysis. The firearm set examined here contains 49 `DA_*` assets under `Weapons/Firearms`.

🟢 **Confirmed** — conventional firearm DataAssets contain repeated stat records whose two floating-point values behave as lower/upper generation bounds.

The five recurring firearm stat identities are:

```text
Jig.Stat.FirearmDamage
Jig.Stat.CriticalHitMultiplier
Jig.Stat.CriticalHitChance
Jig.Stat.FirearmRPM
Jig.Stat.DamageFallOff
```

## Crusher cross-check

🟢 **Confirmed by runtime cross-check** — a previously observed live Crusher instance had Damage 63, Crit Multiplier 28, Crit Chance 18, RPM 645 and Falloff 117. Every live value lands inside the corresponding cooked DataAsset range in the [named-weapon reference table](weapons.md#named). This cross-check supports the positional stat-record mapping and the interpretation of the paired values as generation bounds; explicit property-level decoding is still needed to verify that mapping across every asset.

This is an important bridge between cooked item definitions, live `ItemStats`, and serialized save data.

## Conventional firearm ranges

The 45 conventional-weapon ranges are maintained in the [weapon reference](weapons.md#rifles). For assets with five records, the interpreted order is Damage / Crit Multiplier / Crit Chance / RPM / Falloff. Several manually cycled or bolt/pump weapons contain four records and omit RPM. The reference marks these omissions with `—`, not zero.

## Special weapons

🔵 **Research** — the four `Special` assets do not all match the ordinary five/four-record firearm layout. Their detected pairs and unresolved mappings are recorded in the [special-weapons section](weapons.md#special). Do not assign those pairs to damage or crit without explicit property-level decoding.

## Current interpretation

🟢 **Confirmed for conventional firearms** — weapon instances are not simply using one fixed cooked number for these stats. The DataAsset stores a range and live instances can hold values from within those bounds.

🟡 **Probable** — the game rolls/generates an instance value from each DataAsset bound pair when constructing a weapon instance. More samples are needed to establish the exact distribution, rounding rules, and whether rarity or other systems further modify the rolled result.

This matters directly for progression/modding design: upgrades should be based on the weapon instance's live values or on normalized position within its vanilla range, rather than assuming every weapon of a given type starts with one fixed stat block.

## Next research

- Decode the exact cooked `S_ItemStat` property schema rather than relying on the now-validated positional layout.
- Collect multiple newly spawned copies of the same firearm to determine roll distribution and integer/float rounding behaviour.
- Determine whether rarity changes the bounds, the roll, or a later modifier stage.
- Map special weapons separately.
- Compare cooked bounds against `PlayerInfo.sav` values for the same live weapon UID.
