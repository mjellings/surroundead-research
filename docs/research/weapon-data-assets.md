# Weapon DataAsset Research

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

`DA_Crusher` contains these ranges:

| Stat | DataAsset range |
| --- | ---: |
| FirearmDamage | 54–64 |
| CriticalHitMultiplier | 24–29 |
| CriticalHitChance | 12–22 |
| FirearmRPM | 600–700 |
| DamageFallOff | 110–120 |

🟢 **Confirmed by runtime cross-check** — a previously observed live Crusher instance had Damage 63, Crit Multiplier 28, Crit Chance 18, RPM 645 and Falloff 117. Every live value lands inside the corresponding cooked DataAsset range. This establishes the stat-record ordering used for the conventional firearm table below and strongly supports the interpretation of the paired values as generation bounds.

This is an important bridge between cooked item definitions, live `ItemStats`, and serialized save data.

## Conventional firearm ranges

For assets with five records, the layout resolves cleanly as Damage / Crit Multiplier / Crit Chance / RPM / Falloff. Several manually cycled or bolt/pump weapons contain four records and omit RPM; those are marked with `—` below.

| Weapon | Group | Damage | Crit Mult | Crit Chance | RPM | Falloff |
| --- | --- | ---: | ---: | ---: | ---: | ---: |
| M249 | LMG | 40–44 | 20–25 | 4–7 | 750–850 | 55–75 |
| XM250 | LMG | 42–46 | 14–20 | 10–15 | 725–775 | 60–80 |
| CR308 | Marksman | 58–70 | 25–35 | 15–20 | 575–725 | 80–90 |
| FNFal | Marksman | 55–63 | 20–30 | 10–15 | 600–700 | 75–85 |
| SVD | Marksman | 54–65 | 22–32 | 8–13 | 200–300 | 90–100 |
| BattleReadyGlock | Named | 29–35 | 24–33 | 14–22 | 1100–1200 | 30–40 |
| Cerberus | Named | 68–78 | 20–30 | 30–35 | 750–850 | 120–130 |
| ColonelsRevenge | Named | 50–60 | 25–35 | 10–20 | 750–850 | 85–95 |
| Crusher | Named | 54–64 | 24–29 | 12–22 | 600–700 | 110–120 |
| Deadeye | Named | 75–90 | 40–50 | 25–35 | 75–150 | 55–65 |
| Exterminator | Named | 22–27 | 40–60 | 20–30 | — | 30–40 |
| Guardian | Named | 61–70 | 25–35 | 10–20 | 625–725 | 110–125 |
| Lechie | Named | 215–235 | 20–25 | 45–60 | — | 175–200 |
| MarksmansLegacy | Named | 165–185 | 18–23 | 40–55 | — | 160–180 |
| Phantasm | Named | 49–59 | 25–30 | 18–23 | 800–850 | 80–90 |
| Survivor | Named | 53–62 | 19–26 | 12–22 | 575–675 | 100–105 |
| Valkyrie | Named | 50–60 | 25–35 | 10–20 | 750–850 | 100–105 |
| ACR | Rifles | 43–48 | 25–30 | 5–8 | 650–700 | 55–65 |
| AK15 | Rifles | 45–51 | 21–26 | 5–10 | 575–700 | 65–85 |
| AK74 | Rifles | 43–48 | 20–26 | 2–7 | 500–675 | 60–80 |
| AR15 | Rifles | 40–46 | 18–24 | 3–8 | 675–775 | 45–65 |
| HK416 | Rifles | 43–50 | 12–18 | 5–10 | 750–850 | 50–60 |
| KS1 | Rifles | 41–48 | 16–23 | 10–15 | 700–900 | 60–70 |
| M4CQB | Rifles | 38–44 | 18–24 | 3–8 | 700–800 | 30–50 |
| RadianMod1 | Rifles | 45–53 | 15–20 | 4–9 | 700–800 | 45–70 |
| ScarH | Rifles | 47–53 | 20–25 | 8–13 | 550–650 | 75–90 |
| SteyrAug | Rifles | 41–48 | 20–25 | 4–9 | 650–720 | 40–60 |
| AR45 | SMG | 35–40 | 16–20 | 7–12 | 925–1050 | 25–40 |
| BlackOpsMP5 | SMG | 38–43 | 12–17 | 8–12 | 1100–1250 | 30–40 |
| KrissVector | SMG | 32–38 | 14–22 | 4–9 | 1100–1250 | 25–40 |
| MP5 | SMG | 32–38 | 14–22 | 4–9 | 1100–1250 | 25–35 |
| P90 | SMG | 30–36 | 19–26 | 3–7 | 850–1100 | 30–40 |
| BenelliM4 | Shotgun | 19–23 | 25–32 | 3–6 | 200–300 | 15–30 |
| PumpActionShotgun | Shotgun | 16–21 | 20–25 | 3–6 | — | 15–25 |
| BlackOpsUSP | Sidearm | 24–29 | 14–20 | 10–15 | 325–375 | 20–30 |
| Deagle | Sidearm | 87–96 | 12–18 | 12–18 | 250–300 | 30–40 |
| G18 | Sidearm | 20–25 | 10–16 | 3–6 | 1050–1200 | 15–30 |
| M9 | Sidearm | 20–25 | 10–16 | 3–6 | 1050–1200 | 15–30 |
| MP7 | Sidearm | 19–26 | 13–18 | 4–7 | 900–1000 | 25–35 |
| SawnOffShotgun | Sidearm | 12–16 | 20–25 | 2–4 | — | 10–20 |
| Winchester45 | Sidearm | 36–46 | 20–30 | 7.5–12.5 | 60–110 | 35–45 |
| BarrettM821 | Sniper | 225–265 | 20–25 | 20–30 | 350–425 | 185–215 |
| HuntingRifle | Sniper | 82–92 | 16–24 | 8–12 | — | 90–110 |
| M2010 | Sniper | 140–162 | 18–24 | 22–28 | — | 175–200 |
| MosinNagant | Sniper | 83–93 | 16–24 | 8–12 | — | 90–105 |

## Special weapons

🔵 **Research** — several `Special` assets do not use the ordinary five/four-record firearm layout and should not yet be forced into the same schema:

- `DA_Crossbow`: three detected paired records: 35–50, 14–24, 22–32.
- `DA_GrenadeLauncher`: one detected paired record: 300–375.
- `DA_FlareGun`: no records matching the conventional firearm pattern.
- `DA_RocketLauncher`: no records matching the conventional firearm pattern.

Their exact tag-to-range mapping still needs explicit property-level decoding.

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
