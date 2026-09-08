# Weapon reference — base stats, recoil and spread

[Research index](README.md) · [DataAsset research](weapon-data-assets.md) · [Runtime stat updates](runtime-weapon-stats.md)

Vanilla **SurrounDead 0.8 / UE5.6** research, compiled 2026-09-08. Covers **49 firearm definitions**, including four special weapons; melee weapons are outside this page's scope.

## How to read the tables

- **Base stats are generation ranges**, not a fixed starting value for every copy. Damage, crit, RPM and falloff below are the canonical tables for the 45 conventional-weapon records. [Weapon DataAsset Research](weapon-data-assets.md) preserves their extraction method and live Crusher cross-check. The exact roll distribution remains unresolved.
- **Recoil/spread numbers are explicit cooked component values**, decoded from the uploaded `Items.zip` with the supplied UE5.6 `.usmap`. These are static asset observations, not proof of final in-game values or successful UE4SS writes.
- **`—` means absent from the inspected record / unresolved**, never an assumed zero. A component can inherit a value. Shared defaults are listed separately below; the complete archetype chain and runtime initialization have not been resolved.
- Recoil keeps its original sign. Values are internal units; no degree, percentage, metre or critical-damage formula is asserted here. `RecoilRate` is not weapon RPM. Shotgun damage is not asserted to be total damage across all pellets.
- Names use asset identifiers for reliable lookup. `BarrettM821` is the DataAsset identifier; its pickup is `BP_BarrettM82Pickup`.

Each category has a base-stat table followed by a handling table, avoiding one excessively wide table. **V/H recoil** and **V/H deviation** mean vertical/horizontal. See [what spread and deviation mean](#what-spread-and-deviation-mean) for the practical interpretation, and the [field dictionary](#field-dictionary) for exact property names.

## What spread and deviation mean

The field names and serialized numbers are observed facts. The gameplay interpretations below are **working hypotheses from those names and their placement in the weapon component**, not traced Blueprint formulas or measured effects.

| Concept | What it likely controls | What changing it might achieve |
| --- | --- | --- |
| Spread | How far a shot can depart from the aim direction. | A smaller spread parameter may tighten shot grouping, even while aim stays still. |
| Vertical / horizontal recoil | The up/down and left/right kick associated with firing. | Reducing the magnitude may reduce aim displacement after a shot. |
| Vertical / horizontal random deviation | Random variation in the recoil on each axis; these fields are inside `RecoilData`. | A smaller deviation may make kick more consistent from shot to shot, without removing the underlying recoil. |

For example, **less spread** would be an accuracy reward; **less recoil deviation** would be a predictability reward. They should not be presented as interchangeable until firing tests establish how this game uses them.

### Reading the numbers

- **`HipfireSpread`** suggests spread when firing without aiming down sights. A value of `4` is a raw parameter, not a confirmed 4-degree cone or 4% inaccuracy.
- **`ShootingSpread`** is a separate field. Its name does not establish that it is ADS-only spread, or whether it adds to, multiplies, or replaces hip-fire spread. Only four examined weapon exports explicitly override it, all shotgun-type weapons; omissions elsewhere do not prove perfect accuracy.
- **`VerticalRandomDeviation` / `HorizontalRandomDeviation`** suggest random recoil variation on their respective axes. We have not established the sampling distribution, whether the bounds are ± the value, or whether variation is added or multiplied. Do not calculate a recoil interval from these numbers yet.
- **Signed recoil:** compare magnitude when discussing kick. Moving `−0.35` toward zero to `−0.315` is a 10% reduction in the parameter's magnitude. Making it more negative would increase that magnitude. The observed sign alone does not establish the camera's direction of movement.
- **Shotgun min/max spread:** these are another pair of signed bounds. A narrower pair suggests a tighter pellet pattern, but their coordinate space, units and relationship with the other spread fields are unresolved. `−17500 / 17500` is not a known angle or distance.
- **`RecoilRate` / `ResetRecoilDivider`:** their names suggest recoil timing/recovery behaviour. We have not established whether raising or lowering either improves handling.

### A concrete comparison

HK416 explicitly stores vertical/horizontal recoil of `−0.05 / 0.1`, with deviations `0.015 / 0.025`. M249 stores `−0.35 / 0.35`, with deviations `0.05 / 0.025`. The M249 therefore has a larger serialized vertical recoil magnitude and vertical-deviation parameter. This does **not** establish that its visible kick is seven times greater: timing, attachments, inheritance and runtime calculations may change the outcome.

### What still needs testing

Hold the weapon, attachments, stance and firing conditions constant. Capture the live component values, then vary one field at a time: measure shot grouping separately from aim/camera movement and recovery. Test hip-fire and ADS separately, and compare repeated shots rather than one shot. Trace the consuming Blueprint logic to establish units and formulas. Until then, a “10% reduction” means 10% of the chosen parameter, not a verified 10% improvement in accuracy or handling.

## Rifles

| Weapon | Damage | Crit mult | Crit chance | RPM | Falloff |
| --- | ---: | ---: | ---: | ---: | ---: |
| ACR | 43–48 | 25–30 | 5–8 | 650–700 | 55–65 |
| AK15 | 45–51 | 21–26 | 5–10 | 575–700 | 65–85 |
| AK74 | 43–48 | 20–26 | 2–7 | 500–675 | 60–80 |
| AR15 | 40–46 | 18–24 | 3–8 | 675–775 | 45–65 |
| HK416 | 43–50 | 12–18 | 5–10 | 750–850 | 50–60 |
| KS1 | 41–48 | 16–23 | 10–15 | 700–900 | 60–70 |
| M4CQB | 38–44 | 18–24 | 3–8 | 700–800 | 30–50 |
| RadianMod1 | 45–53 | 15–20 | 4–9 | 700–800 | 45–70 |
| ScarH | 47–53 | 20–25 | 8–13 | 550–650 | 75–90 |
| SteyrAug | 41–48 | 20–25 | 4–9 | 650–720 | 40–60 |

| Weapon | V recoil | H recoil | V deviation | H deviation | Hip spread | Shooting spread | Recoil rate | Reset divider |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| ACR | — | 0.4 | 0.05 | 0.05 | — | — | — | — |
| AK15 | -0.35 | 0.15 | 0.05 | 0.05 | — | — | 2 | — |
| AK74 | -0.5 | — | — | 0.15 | — | — | 2 | — |
| AR15 | -0.04 | 0.4 | 0.02 | — | — | — | 2 | — |
| HK416 | -0.05 | 0.1 | 0.015 | 0.025 | — | — | 2 | — |
| KS1 | -0.01 | 0.5 | 0.005 | 0.025 | — | — | 3 | 1 |
| M4CQB | -0.25 | 0.25 | — | — | — | — | — | — |
| RadianMod1 | -0.05 | 0.1 | 0.015 | 0.025 | — | — | 2 | — |
| ScarH | -0.45 | 0.125 | 0.2 | — | — | — | 2 | — |
| SteyrAug | -0.4 | — | — | 0.05 | — | — | — | — |

## SMG

| Weapon | Damage | Crit mult | Crit chance | RPM | Falloff |
| --- | ---: | ---: | ---: | ---: | ---: |
| AR45 | 35–40 | 16–20 | 7–12 | 925–1050 | 25–40 |
| BlackOpsMP5 | 38–43 | 12–17 | 8–12 | 1100–1250 | 30–40 |
| KrissVector | 32–38 | 14–22 | 4–9 | 1100–1250 | 25–40 |
| MP5 | 32–38 | 14–22 | 4–9 | 1100–1250 | 25–35 |
| P90 | 30–36 | 19–26 | 3–7 | 850–1100 | 30–40 |

| Weapon | V recoil | H recoil | V deviation | H deviation | Hip spread | Shooting spread | Recoil rate | Reset divider |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| AR45 | -0.1 | 0.15 | 0.05 | 0.05 | — | — | — | — |
| BlackOpsMP5 | -0.04 | 0.4 | 0.01 | 0.15 | — | — | 3 | — |
| KrissVector | -0.125 | 0.125 | 0.05 | — | — | — | — | — |
| MP5 | -0.075 | 0.325 | 0.05 | — | — | — | — | — |
| P90 | -0.3 | 0.4 | — | 0.15 | — | — | 3 | — |

## LMG

| Weapon | Damage | Crit mult | Crit chance | RPM | Falloff |
| --- | ---: | ---: | ---: | ---: | ---: |
| M249 | 40–44 | 20–25 | 4–7 | 750–850 | 55–75 |
| XM250 | 42–46 | 14–20 | 10–15 | 725–775 | 60–80 |

| Weapon | V recoil | H recoil | V deviation | H deviation | Hip spread | Shooting spread | Recoil rate | Reset divider |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| M249 | -0.35 | 0.35 | 0.05 | 0.025 | 4 | — | 2 | — |
| XM250 | -0.3 | 0.3 | 0.025 | 0.025 | 4 | — | — | — |

## Marksman

| Weapon | Damage | Crit mult | Crit chance | RPM | Falloff |
| --- | ---: | ---: | ---: | ---: | ---: |
| CR308 | 58–70 | 25–35 | 15–20 | 575–725 | 80–90 |
| FNFal | 55–63 | 20–30 | 10–15 | 600–700 | 75–85 |
| SVD | 54–65 | 22–32 | 8–13 | 200–300 | 90–100 |

| Weapon | V recoil | H recoil | V deviation | H deviation | Hip spread | Shooting spread | Recoil rate | Reset divider |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| CR308 | -0.45 | 0.3 | — | 0.2 | 4 | — | 2 | — |
| FNFal | -0.55 | 0.4 | 0.2 | — | 4 | — | 2 | — |
| SVD | -0.3 | — | 0.15 | — | 4 | — | 2 | — |

## Sniper

| Weapon | Damage | Crit mult | Crit chance | RPM | Falloff |
| --- | ---: | ---: | ---: | ---: | ---: |
| BarrettM821 | 225–265 | 20–25 | 20–30 | 350–425 | 185–215 |
| HuntingRifle | 82–92 | 16–24 | 8–12 | — | 90–110 |
| M2010 | 140–162 | 18–24 | 22–28 | — | 175–200 |
| MosinNagant | 83–93 | 16–24 | 8–12 | — | 90–105 |

| Weapon | V recoil | H recoil | V deviation | H deviation | Hip spread | Shooting spread | Recoil rate | Reset divider |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| BarrettM821 | -2.35 | 0.9 | 0.5 | 0.25 | 6 | — | 2 | 2 |
| HuntingRifle | -1.85 | 0.6 | 0.3 | — | 5 | — | 1.75 | 2 |
| M2010 | -1.75 | 0.8 | 0.2 | 0.2 | 5 | — | 2 | — |
| MosinNagant | -2 | 0.8 | 0.3 | 0.25 | 5 | — | 1.75 | 2 |

## Shotgun

| Weapon | Damage | Crit mult | Crit chance | RPM | Falloff |
| --- | ---: | ---: | ---: | ---: | ---: |
| BenelliM4 | 19–23 | 25–32 | 3–6 | 200–300 | 15–30 |
| PumpActionShotgun | 16–21 | 20–25 | 3–6 | — | 15–25 |

| Weapon | V recoil | H recoil | V deviation | H deviation | Hip spread | Shooting spread | Recoil rate | Reset divider |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| BenelliM4 | -1.5 | 0.3 | 0.5 | 0.15 | — | 3 | 3 | — |
| PumpActionShotgun | -2.25 | 0.5 | 0.5 | 0.25 | 3.25 | 3.25 | 3 | 2 |

## Sidearm

| Weapon | Damage | Crit mult | Crit chance | RPM | Falloff |
| --- | ---: | ---: | ---: | ---: | ---: |
| BlackOpsUSP | 24–29 | 14–20 | 10–15 | 325–375 | 20–30 |
| Deagle | 87–96 | 12–18 | 12–18 | 250–300 | 30–40 |
| G18 | 20–25 | 10–16 | 3–6 | 1050–1200 | 15–30 |
| M9 | 20–25 | 10–16 | 3–6 | 1050–1200 | 15–30 |
| MP7 | 19–26 | 13–18 | 4–7 | 900–1000 | 25–35 |
| SawnOffShotgun | 12–16 | 20–25 | 2–4 | — | 10–20 |
| Winchester45 | 36–46 | 20–30 | 7.5–12.5 | 60–110 | 35–45 |

| Weapon | V recoil | H recoil | V deviation | H deviation | Hip spread | Shooting spread | Recoil rate | Reset divider |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| BlackOpsUSP | -0.15 | — | — | 0.125 | — | — | 3 | — |
| Deagle | -1 | — | 0.2 | 0.15 | 4 | — | — | — |
| G18 | -0.175 | 0.25 | — | 0.125 | — | — | — | — |
| M9 | — | 0.175 | — | — | — | — | — | — |
| MP7 | -0.7 | 0.15 | 0.15 | 0.05 | — | — | 3 | — |
| SawnOffShotgun | -2.25 | 1.25 | 0.25 | 0.25 | 3.5 | 3.5 | 3 | 2 |
| Winchester45 | -1.35 | 0.3 | 0.25 | — | — | — | 2 | — |

## Named

| Weapon | Damage | Crit mult | Crit chance | RPM | Falloff |
| --- | ---: | ---: | ---: | ---: | ---: |
| BattleReadyGlock | 29–35 | 24–33 | 14–22 | 1100–1200 | 30–40 |
| Cerberus | 68–78 | 20–30 | 30–35 | 750–850 | 120–130 |
| ColonelsRevenge | 50–60 | 25–35 | 10–20 | 750–850 | 85–95 |
| Crusher | 54–64 | 24–29 | 12–22 | 600–700 | 110–120 |
| Deadeye | 75–90 | 40–50 | 25–35 | 75–150 | 55–65 |
| Exterminator | 22–27 | 40–60 | 20–30 | — | 30–40 |
| Guardian | 61–70 | 25–35 | 10–20 | 625–725 | 110–125 |
| Lechie | 215–235 | 20–25 | 45–60 | — | 175–200 |
| MarksmansLegacy | 165–185 | 18–23 | 40–55 | — | 160–180 |
| Phantasm | 49–59 | 25–30 | 18–23 | 800–850 | 80–90 |
| Survivor | 53–62 | 19–26 | 12–22 | 575–675 | 100–105 |
| Valkyrie | 50–60 | 25–35 | 10–20 | 750–850 | 100–105 |

| Weapon | V recoil | H recoil | V deviation | H deviation | Hip spread | Shooting spread | Recoil rate | Reset divider |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| BattleReadyGlock | -0.175 | 0.15 | 0.075 | 0.075 | — | — | — | 1 |
| Cerberus | -0.3 | 0.25 | 0.125 | 0.125 | — | — | — | 1 |
| ColonelsRevenge | -0.15 | 0.35 | 0.075 | — | — | — | — | 1 |
| Crusher | -0.35 | 0.15 | 0.025 | 0.025 | — | — | — | 1 |
| Deadeye | -1 | 0.3 | 0.2 | — | — | — | — | 1 |
| Exterminator | -0.95 | 0.85 | 0.15 | 0.15 | 2 | 2 | 3 | 1 |
| Guardian | — | 0.3 | — | — | — | — | — | 1 |
| Lechie | -1.85 | 1 | 0.5 | 0.5 | 4 | — | 1.5 | 1 |
| MarksmansLegacy | -1.75 | 1.75 | 0.75 | 0.75 | 4 | — | 1.75 | — |
| Phantasm | -0.175 | 0.4 | 0.025 | 0.025 | — | — | — | 1 |
| Survivor | -0.4 | 0.1 | 0.15 | — | — | — | — | 1 |
| Valkyrie | -0.15 | 0.25 | 0.075 | — | — | — | — | 1 |

## Special

Special-weapon damage/stat identities remain unresolved; the earlier extraction found Crossbow pairs `35–50`, `14–24`, `22–32` and a GrenadeLauncher pair `300–375`, but did not establish their tag mapping. FlareGun and RocketLauncher had no conventional matching records. These are not assigned to damage or crit columns.

| Weapon | V recoil | H recoil | V deviation | H deviation | Hip spread | Shooting spread | Recoil rate | Reset divider |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Crossbow | 0 | 0 | 0 | 0 | — | — | — | — |
| FlareGun | 0.5 | 0.5 | 0.25 | 0.25 | — | — | — | 1 |
| GrenadeLauncher | -1.5 | 1 | 0.5 | 0.5 | — | — | — | 2 |
| RocketLauncher | -0.1 | 0.1 | 0.025 | 0.025 | — | — | — | 1 |

## Shotgun spread bounds

These four pickups explicitly enable `UseShotgunSpread?`. Bounds are signed internal values, not pellet counts or angles. PumpActionShotgun omits both bounds in its component export; the shared component has −25000 / 25000, but this is not presented as a resolved per-weapon runtime value.

| Weapon | UseShotgunSpread? | ShotgunMinSpread | ShotgunMaxSpread |
| --- | ---: | ---: | ---: |
| BenelliM4 | true | -20000 | 20000 |
| PumpActionShotgun | true | — | — |
| SawnOffShotgun | true | -27500 | 27500 |
| Exterminator | true | -17500 | 17500 |

## Shared component defaults

Explicitly serialized on `Default__BP_WeaponsPickupComponent_C` in `/Game/Inventory/Items/Other/Components/BP_WeaponsPickupComponent`. These provide inheritance context, not automatic replacements for every `—` above. The generic `BP_FirearmPickup` component template contains no explicit property overrides in the examined export.

| Property | Serialized default |
| --- | ---: |
| RecoilVertical | -0.2 |
| RecoilHorizontal | 0.2 |
| VerticalRandomDeviation | 0.1 |
| HorizontalRandomDeviation | 0.1 |
| RecoilRate | 2.5 |
| ResetRecoilDivider | 1.5 |
| HipfireSpread | 3 |
| ShootingSpread | — |
| ShotgunMinSpread | -25000 |
| ShotgunMaxSpread | 25000 |

## Field dictionary

| Table field | Property / stat tag | Interpretation and limit |
| --- | --- | --- |
| Damage | `Jig.Stat.FirearmDamage` | DataAsset lower/upper bounds; actual hits may have further modifiers. |
| Crit mult | `Jig.Stat.CriticalHitMultiplier` | Raw bounds; do not read `20` as a proven 20× multiplier. |
| Crit chance | `Jig.Stat.CriticalHitChance` | Raw bounds; conversion to a probability has not been established here. |
| RPM | `Jig.Stat.FirearmRPM` | Rate-of-fire stat bounds; omitted records are not zero RPM. |
| Falloff | `Jig.Stat.DamageFallOff` | Raw falloff bounds; distance units and curve require runtime verification. |
| V recoil | `RecoilData.RecoilVertical` | Signed vertical recoil parameter. |
| H recoil | `RecoilData.RecoilHorizontal` | Signed horizontal recoil parameter. |
| V deviation | `RecoilData.VerticalRandomDeviation` | Vertical random-deviation parameter. |
| H deviation | `RecoilData.HorizontalRandomDeviation` | Horizontal random-deviation parameter. |
| Hip spread | `HipfireSpread` | Hip-fire spread parameter; final trajectory effect remains to be measured. |
| Shooting spread | `ShootingSpread` | Shooting-spread parameter; not yet established as ADS-only accuracy. |
| Recoil rate | `RecoilRate` | Recoil behaviour parameter; direction of improvement needs testing. |
| Reset divider | `ResetRecoilDivider` | Recoil-reset parameter; direction of improvement needs testing. |
| Shotgun bounds | `ShotgunMinSpread` / `ShotgunMaxSpread` | Separate signed spread bounds used alongside the shotgun-spread flag. |

The four `RecoilData` member names have generated suffixes in the mapping; the [decoded evidence](weapon-handling-values.json) retains their full names.

## Evidence and reproduction

Base-stat ranges come from [the existing DataAsset investigation](weapon-data-assets.md); they have not been independently re-decoded by tag for this page. Its four/five-record positional mapping and special-weapon limitations remain in force.

Handling values come from `BP_WeaponsPickupComponent_GEN_VARIABLE` exports in pickup assets under `/Game/Inventory/Items/Pickups/Weapons/Firearms/`, plus the shared component CDO. The supplied mapping archive is `SurrounDead-5.6.1-44394996+++UE5+Release-5.6-24b12662.zip`. That identifies the mapping source, not independently the exact game patch of every exported asset.

The analysis decoded unversioned property fragments and zero masks against the component and `Recoil_Struct` schemas. All **51 exports** (49 weapons, one generic firearm template, one shared default) decoded without reported errors and consumed their property data with only a 4- or 8-byte zero trailer remaining. Explicit zero-mask values are retained as zero, including all four Crossbow recoil members. This is a structural validation, not an in-game behaviour test.

[Decoded handling evidence](weapon-handling-values.json) contains source paths, exact property names, explicit values and export-length checks. Only derived research data is committed; no cooked game files or mapping files are redistributed.

Source archive SHA-256 (`Items.zip`): `f87a6051868e9831fb5d0bed62af10f7cfd809e474556e51059111036450b4f2`.

## Implications for WeaponProgression

The existing UID tracking, milestone/config logic and UI can be reused to select and display deterministic bonuses. Damage/crit/RPM/falloff use the already researched item-stat route. Handling fields live on weapon components and need a separate runtime probe before promising working recoil or accuracy upgrades.

For that probe, capture the equipped instance's effective baseline, test a small change to one field, measure firing behaviour, then check weapon swaps and attachments. If scaling signed recoil, scaling toward zero preserves its sign (for example, −0.35 × 0.9 = −0.315). Do not repeatedly compound bonuses onto an already modified value or mutate a shared class default. The safe direction for `RecoilRate` and `ResetRecoilDivider`, inheritance resolution and persistence all remain research tasks.
