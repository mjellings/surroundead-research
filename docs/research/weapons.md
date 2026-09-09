# Weapon reference — base stats, recoil and spread

[Research index](README.md) · [DataAsset research](weapon-data-assets.md) · [Runtime stat updates](runtime-weapon-stats.md)

Vanilla **SurrounDead 0.8 / UE5.6** research, compiled 2026-09-08 and updated 2026-09-09. Covers **49 firearm definitions**, including four special weapons; melee weapons are outside this page's scope.

**Confidence key:** 🟢 **Confirmed** — demonstrated in the stated tests · 🟡 **Probable** — supported interpretation · 🔵 **Research** — unresolved · 🔴 **Failed / obsolete** — superseded conclusion. See the [repository definitions](../../README.md#confidence-markers).

## How to read the tables

- **Base stats are generation ranges**, not a fixed starting value for every copy. Damage, crit, RPM and falloff below are the canonical tables for the 45 conventional-weapon records. [Weapon DataAsset Research](weapon-data-assets.md) preserves their extraction method and live Crusher cross-check. The exact roll distribution remains unresolved.
- **Recoil/spread numbers are explicit cooked component values**, decoded from the uploaded `Items.zip` with the supplied UE5.6 `.usmap`. These tables remain static asset observations; selected live values, UE4SS writes and gameplay effects were subsequently verified in the [runtime tests](#runtime-tests--2026-09-08--2026-09-09).
- **`—` means absent from the inspected record / unresolved**, never an assumed zero. A component can inherit a value. Shared defaults are listed separately below; the complete archetype chain and runtime initialization have not been resolved.
- Recoil keeps its original sign. Values are internal units; no degree, percentage, metre or critical-damage formula is asserted here. `RecoilRate` is not weapon RPM. Shotgun damage is not asserted to be total damage across all pellets.
- Names use asset identifiers for reliable lookup. `BarrettM821` is the DataAsset identifier; its pickup is `BP_BarrettM82Pickup`.

Each category has a base-stat table followed by a handling table, avoiding one excessively wide table. **V/H recoil** and **V/H deviation** mean vertical/horizontal. See [what spread and deviation mean](#what-spread-and-deviation-mean) for the practical interpretation, and the [field dictionary](#field-dictionary) for exact property names.

## What spread and deviation mean

Initial interpretations came from field names and cooked values. **2026-09-08 and 2026-09-09 runtime tests now establish several effects on the HK416 and Benelli M4.** Conclusions combine live property logs with controlled gameplay observations and a screen recording. The recording is suitable as qualitative visual evidence but was not sufficiently smooth for frame-accurate timing or pixel measurements.

| Field | Status of effect | Demonstrated effect | Scope / remaining limit |
| --- | --- | --- | --- |
| `RecoilVertical` | 🟢 **Confirmed** | HK416 `−2` caused much stronger upward kick than baseline `−0.05`. | Positive vertical recoil has not been tested. |
| `RecoilHorizontal` | 🟢 **Confirmed** | HK416 `+2` pulled left; `−2` pulled right. | Directions refer to observed player aim movement in this test. |
| `VerticalRandomDeviation` | 🟢 **Confirmed** | With directional recoil and horizontal deviation zero, `1` introduced strongly variable vertical aim/recoil displacement compared with `0`; a positive value produced displacement both above and below the starting aim. | Exact distribution, bounds and formula remain unknown. The value behaves as a magnitude/range rather than a direct up/down sign in the tested case. |
| `HorizontalRandomDeviation` | 🟢 **Confirmed** | With directional recoil and vertical deviation zero, `1` introduced predominantly left/right random aim/recoil displacement compared with `0`. | Demonstrated in earlier ADS/hip-fire tests and isolated again in the deviation matrix; exact distribution and bounds remain unknown. |
| `HipfireSpread` | 🟢 **Confirmed** | HK416 hip-fire impacts scattered at `3`, but became almost pinpoint at `0`, with recoil/deviation zero. | Intermediate scaling and ADS interaction remain unmeasured. |
| `ShootingSpread` | 🟢 **Confirmed** | Benelli M4 hip-fire grouping was tight at `0` and widened at `3`, with hip-fire spread and recoil/deviation zero. | This is **not an ADS-only field** on the tested shotgun. ADS behaviour and effects on ordinary rifles remain untested. |

🟡 **Probable — progression application:** these results support separate progression concepts: lower directional recoil for less kick, lower vertical/horizontal deviation for more predictable aim, and reduced spread for tighter impact grouping. The 50%-baseline deviation presets were subtle compared with the exaggerated `1` tests, which is encouraging for small progression bonuses, but the experiment does not establish linear visible scaling.

### Reading the numbers

- 🔵 **Research — units:** all handling values remain **raw internal parameters**, not established degrees, percentages or distances.
- 🟡 **Probable — scaling application:** recoil magnitude and sign are separate. Moving `−0.35` to `−0.315` reduces magnitude by 10% while preserving sign; it does not prove 10% less visible kick.
- 🟢 **Confirmed — tested recoil directions:** on the HK416, negative vertical recoil produced upward movement; positive horizontal produced leftward movement and negative horizontal rightward movement. Earlier assumptions of positive horizontal meaning right were contradicted by the tests. Positive vertical recoil meaning down is still unverified.
- 🟢 **Confirmed — deviation is independently active:** both deviation fields produce aim/recoil displacement while `RecoilVertical` and `RecoilHorizontal` are zero. They are therefore not merely percentages of those zero directional-recoil parameters under the tested conditions.
- 🟢 **Confirmed — deviation sign is not screen direction:** `VerticalRandomDeviation=1` produced both upward and downward displacement. The positive parameter therefore acts as a randomisation magnitude/range in the tested case rather than directly selecting upward movement.
- 🟢 **Confirmed — axis character:** isolated vertical deviation was predominantly vertical and isolated horizontal deviation predominantly lateral. With both set to `1`, the weapon showed a much more two-dimensional randomised displacement pattern. This supports the two fields contributing separate components to a combined recoil/aim offset.
- 🟡 **Probable — random-vector interpretation:** the combined behaviour is consistent with the game sampling random vertical/horizontal components from magnitudes derived from these fields. The exact sampling formula, distribution, clamping and whether the two axes are statistically independent are not established.
- 🟢 **Confirmed — observed recovery:** spaced shots can move away from and then back towards the original aim, while rapid fire can wander. The newer deviation test does **not** establish that deviation specifically controls the final recovery destination. 🔵 **Research:** recovery timing/mechanics remain to be isolated with `RecoilRate` and `ResetRecoilDivider`.
- 🟡 **Probable — cooked-property interpretation:** only four cooked weapon exports explicitly override `ShootingSpread`: Exterminator `2`, Benelli M4 `3`, PumpActionShotgun `3.25`, SawnOffShotgun `3.5`; all enable `UseShotgunSpread?`. This does not establish that rifles ignore the field.
- 🟢 **Confirmed — fixed bounds during the test:** Benelli shotgun bounds remained `−20000 / +20000` even when `ShootingSpread=0` produced tight grouping. 🟡 **Probable:** the bounds and shooting-spread field may interact; independent additive spread and any particular multiplication formula are not established.
- 🔵 **Research:** `RecoilRate` and `ResetRecoilDivider` still require isolated timing/recovery tests. Their names alone do not establish the direction of improvement.

### Superseded interpretations

- 🔴 **Failed / obsolete:** positive horizontal recoil was initially expected to pull right. HK416 testing showed positive pulls left and negative pulls right.
- 🔴 **Failed / obsolete:** treating `ShootingSpread` as ADS-only is contradicted by the Benelli hip-fire comparison.
- 🔴 **Failed / obsolete:** `VerticalRandomDeviation` is no longer an untested counterpart inferred only from its name; it has now been independently isolated at runtime.
- 🔵 **Research:** positive vertical recoil pulling down remains an untested prediction, not a confirmed inverse-sign result.

### A concrete comparison

HK416 explicitly stores vertical/horizontal recoil of `−0.05 / 0.1`, with deviations `0.015 / 0.025`. M249 stores `−0.35 / 0.35`, with deviations `0.05 / 0.025`. The M249 therefore has a larger serialized vertical recoil magnitude and vertical-deviation parameter. This does **not** establish that its visible kick is seven times greater: timing, attachments, inheritance and runtime calculations may change the outcome.

### What still needs testing

🔵 **Research — not yet established.**

Positive vertical recoil, the effects of `RecoilRate` and `ResetRecoilDivider`, shotgun min/max bounds, and `ShootingSpread` in ADS and on ordinary rifles remain open. Deviation should still be tested at additional realistic intermediate values and on other weapon families before assuming proportional scaling or a universal formula. Attachment interactions, runtime reconstruction after equipment changes and persistence also need production validation.

## Runtime tests — 2026-09-08 / 2026-09-09

**Environment:** SurrounDead 0.8 / UE5.6; WeaponHandlingResearch Lua probes against live equipped components. Fixed-distance comparisons were performed with an attachment-free HK416 for the deviation matrix. Live readback and firing-event logs verified the selected component values. A screen recording supplied qualitative visual confirmation; because that recording was stuttery/low quality it is not used for frame-accurate timing or pixel-distance claims. Exact game patch and UE4SS build were not recorded.

### Live captured baselines

🟢 **Confirmed — live readbacks for these two tested instances.**

These are observed live baselines, not replacements for missing values in every cooked asset table. Named weapon identifiers are the actual pickup classes.

| Field | HK416 | Benelli M4 |
| --- | ---: | ---: |
| Vertical recoil | −0.05 | −1.5 |
| Horizontal recoil | 0.1 | 0.3 |
| Vertical deviation | 0.015 | 0.5 |
| Horizontal deviation | 0.025 | 0.15 |
| HipfireSpread | 3 | 3 |
| ShootingSpread | 0 | 3 |
| RecoilRate | 2 | 3 |
| ResetRecoilDivider | 1.5 | 1.5 |
| ShotgunMinSpread | −25000 | −20000 |
| ShotgunMaxSpread | 25000 | 20000 |
| UseShotgunSpread? | false | true |

### Comparisons and observations

🟢 **Confirmed — scoped to the reported experiments below.** Explanations of why a behaviour occurs remain interpretations unless explicitly tested.

| Experiment | Controlled change | Visual result |
| --- | --- | --- |
| HK416 vertical recoil | Baseline `−0.05` to `−2`; other monitored values retained | Much stronger upward kick. An earlier zero test verified writable state but did not establish a visual result. |
| HK416 horizontal recoil | `+2` versus `−2`; normal vertical recoil/deviation retained | Positive pulled up-left; negative pulled right. Residual upward movement is consistent with unchanged negative vertical recoil. |
| HK416 horizontal deviation, earlier ADS test | A: all four recoil/deviation values zero. B: only horizontal deviation raised to `1` | A: steady aim, consistent hits. B: left/right aim movement; single shots returned towards starting aim, rapid fire wandered. |
| HK416 horizontal deviation, hip-fire | Same A/B; HipfireSpread stayed `3`, ShootingSpread stayed `0` | A: impacts scattered. B: sideways aim movement plus broad impact scatter during full auto. |
| HK416 deviation matrix control | A: `Vertical=0`, `Horizontal=0`, `VDeviation=0`, `HDeviation=0` | Stable control with ordinary directional recoil and deviation removed. |
| HK416 vertical deviation isolated | B: only `VDeviation=1`; directional recoil and HDeviation zero | Strong random vertical displacement; positive value produced movement both above and below the original aim rather than a single signed direction. |
| HK416 horizontal deviation isolated | C: only `HDeviation=1`; directional recoil and VDeviation zero | Predominantly lateral left/right random displacement. |
| HK416 combined deviation | D: `VDeviation=1`, `HDeviation=1`; directional recoil zero | Strong two-dimensional randomised aim/recoil displacement, consistent with both components contributing together. |
| HK416 reduced vertical deviation | E: `VDeviation=0.0075`, 50% of captured `0.015`; other recoil/deviation zero | Effect was subtle compared with the exaggerated `1` preset; useful as an initial realistic-scale progression probe, not a quantitative scaling result. |
| HK416 reduced horizontal deviation | F: `HDeviation=0.0125`, 50% of captured `0.025`; other recoil/deviation zero | Effect was subtle compared with the exaggerated `1` preset; useful as an initial realistic-scale progression probe, not a quantitative scaling result. |
| HK416 hip-fire spread | A: HipfireSpread `3`. B: `0`. All recoil/deviation zero; ShootingSpread `0` | A: scattered impacts. B: almost pinpoint grouping. |
| Benelli M4 shooting spread, hip-fire | A: ShootingSpread `0`. B: `3`. HipfireSpread and all recoil/deviation zero; shotgun bounds fixed at `−20000 / +20000` | A: tight spot. B: grouping widened again. Three `Svr_WeaponShot` events logged for each condition. |

### Evidence and limits

🔵 **Research — limits of what these tests establish.**

Initial HK416 logs pasted in the conversation at `11:59–12:00` and `14:15` verify capture and sampled recoil writes. The exaggerated vertical and positive-horizontal directions are tester observations; the negative-horizontal test also has live readback at `−2` during two logged shots.

The 2026-09-09 deviation-matrix run used the proven active-weapon resolver (`BP_JigHelperComp:GetActiveWeapon` → active pickup actor → owned `BP_WeaponsPickupComponent`) rather than choosing from all live weapon components. F6 captured the attachment-free HK416 baseline `Vertical=-0.05`, `Horizontal=0.1`, `VDeviation=0.015`, `HDeviation=0.025`. Each A–F preset was written to the active component and read back, with firing-event snapshots showing the selected values remained active during the test. The accompanying screen recording was reviewed against the preset sequence and supports the qualitative axis/direction conclusions above, but its frame quality is insufficient for precise timing measurements.

Earlier uploaded log sources (times are recorded as printed, without timezone conversion):

| Source | Relevant time | Evidence |
| --- | --- | --- |
| `Pasted text(1).txt` | 14:22:52–14:23:09 | HK416 standard reset, deviation A/B and event readbacks; tester identified this as ADS. |
| `Pasted text(2).txt` | 14:26:46–14:27:12 | Hip-fire repeat of deviation A/B; manual baseline restore logged. Also contains earlier ADS entries. |
| `Pasted text(3).txt` | 14:54:20–14:54:42 | HK416 HipfireSpread `3` versus `0`, matching sampled values and manual restore. |
| `Pasted text(4).txt` | 15:01:21–15:01:27 onward | Benelli baseline, ShootingSpread `0` versus `3`, matching firing snapshots. Tester subsequently confirmed hip-fire and wider grouping in B; excerpt has no final restore. |

`LIVE` values match the selected targets in the sampled comparisons. Polling and post-hooks do not exclude a transient change between samples. `StartRecoil` can appear multiple times relative to `Svr_WeaponShot`; event counters must not be treated as bullet/pellet counts. The experiments establish observable effects, not exact units, distributions, formulas or behaviour on all weapon types.

## Rifles

🟡 **Probable — cooked reference:** stat-range mapping and serialized handling values; not every row has been verified live. The green runtime sections above identify the tested cases.

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

🟡 **Probable — cooked reference:** stat-range mapping and serialized handling values; not every row has been verified live. The green runtime sections above identify the tested cases.

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

🟡 **Probable — cooked reference:** stat-range mapping and serialized handling values; not every row has been verified live. The green runtime sections above identify the tested cases.

| Weapon | Damage | Crit mult | Crit chance | RPM | Falloff |
| --- | ---: | ---: | ---: | ---: | ---: |
| M249 | 40–44 | 20–25 | 4–7 | 750–850 | 55–75 |
| XM250 | 42–46 | 14–20 | 10–15 | 725–775 | 60–80 |

| Weapon | V recoil | H recoil | V deviation | H deviation | Hip spread | Shooting spread | Recoil rate | Reset divider |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| M249 | -0.35 | 0.35 | 0.05 | 0.025 | 4 | — | 2 | — |
| XM250 | -0.3 | 0.3 | 0.025 | 0.025 | 4 | — | — | — |

## Marksman

🟡 **Probable — cooked reference:** stat-range mapping and serialized handling values; not every row has been verified live. The green runtime sections above identify the tested cases.

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

🟡 **Probable — cooked reference:** stat-range mapping and serialized handling values; not every row has been verified live. The green runtime sections above identify the tested cases.

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

🟡 **Probable — cooked reference:** stat-range mapping and serialized handling values; not every row has been verified live. The green runtime sections above identify the tested cases.

| Weapon | Damage | Crit mult | Crit chance | RPM | Falloff |
| --- | ---: | ---: | ---: | ---: | ---: |
| BenelliM4 | 19–23 | 25–32 | 3–6 | 200–300 | 15–30 |
| PumpActionShotgun | 16–21 | 20–25 | 3–6 | — | 15–25 |

| Weapon | V recoil | H recoil | V deviation | H deviation | Hip spread | Shooting spread | Recoil rate | Reset divider |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| BenelliM4 | -1.5 | 0.3 | 0.5 | 0.15 | — | 3 | 3 | — |
| PumpActionShotgun | -2.25 | 0.5 | 0.5 | 0.25 | 3.25 | 3.25 | 3 | 2 |

## Sidearm

🟡 **Probable — cooked reference:** stat-range mapping and serialized handling values; not every row has been verified live. The green runtime sections above identify the tested cases.

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

🟡 **Probable — cooked reference:** stat-range mapping and serialized handling values; not every row has been verified live. The green runtime sections above identify the tested cases.

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

🔵 **Research — special-weapon stat mappings are unresolved.** Handling values below are 🟡 **Probable** cooked-property interpretations.

Special-weapon damage/stat identities remain unresolved; the earlier extraction found Crossbow pairs `35–50`, `14–24`, `22–32` and a GrenadeLauncher pair `300–375`, but did not establish their tag mapping. FlareGun and RocketLauncher had no conventional matching records. These are not assigned to damage or crit columns.

| Weapon | V recoil | H recoil | V deviation | H deviation | Hip spread | Shooting spread | Recoil rate | Reset divider |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Crossbow | 0 | 0 | 0 | 0 | — | — | — | — |
| FlareGun | 0.5 | 0.5 | 0.25 | 0.25 | — | — | — | 1 |
| GrenadeLauncher | -1.5 | 1 | 0.5 | 0.5 | — | — | — | 2 |
| RocketLauncher | -0.1 | 0.1 | 0.025 | 0.025 | — | — | — | 1 |

## Shotgun spread bounds

🔵 **Research — gameplay role of the bounds.** The serialized values are 🟡 **Probable** cooked-property interpretations; the Benelli bounds were also confirmed by live readback, but were not independently varied.

These four pickups explicitly enable `UseShotgunSpread?`. Bounds are signed internal values, not pellet counts or angles. PumpActionShotgun omits both bounds in its component export; the shared component has −25000 / 25000, but this is not presented as a resolved per-weapon runtime value.

| Weapon | UseShotgunSpread? | ShotgunMinSpread | ShotgunMaxSpread |
| --- | ---: | ---: | ---: |
| BenelliM4 | true | -20000 | 20000 |
| PumpActionShotgun | true | — | — |
| SawnOffShotgun | true | -27500 | 27500 |
| Exterminator | true | -17500 | 17500 |

## Shared component defaults

🟡 **Probable — decoded shared defaults.** 🔵 **Research — full inheritance resolution across weapons.**

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

| Table field | Status of interpretation | Property / stat tag | Interpretation and limit |
| --- | --- | --- | --- |
| Damage | 🟡 **Probable** | `Jig.Stat.FirearmDamage` | DataAsset lower/upper bounds; actual hits may have further modifiers. |
| Crit mult | 🟡 **Probable** | `Jig.Stat.CriticalHitMultiplier` | Raw bounds; do not read `20` as a proven 20× multiplier. |
| Crit chance | 🟡 **Probable** | `Jig.Stat.CriticalHitChance` | Raw bounds; conversion to a probability has not been established here. |
| RPM | 🟡 **Probable** | `Jig.Stat.FirearmRPM` | Rate-of-fire stat bounds; omitted records are not zero RPM. |
| Falloff | 🟡 **Probable** | `Jig.Stat.DamageFallOff` | Raw falloff bounds; distance units and curve require runtime verification. |
| V recoil | 🟢 **Confirmed** (tested weapon) | `RecoilData.RecoilVertical` | Signed kick; negative caused upward movement on HK416; positive untested. |
| H recoil | 🟢 **Confirmed** (tested weapon) | `RecoilData.RecoilHorizontal` | Signed kick; positive left / negative right on tested HK416. |
| V deviation | 🟢 **Confirmed** (tested weapon) | `RecoilData.VerticalRandomDeviation` | Random vertical aim/recoil displacement magnitude; positive `1` produced both upward and downward displacement when other recoil/deviation components were zero. Exact distribution/formula unknown. |
| H deviation | 🟢 **Confirmed** (tested weapon) | `RecoilData.HorizontalRandomDeviation` | Predominantly left/right random aim/recoil displacement, independently demonstrated with other recoil/deviation components zero. Exact distribution/formula unknown. |
| Hip spread | 🟢 **Confirmed** (tested weapon) | `HipfireSpread` | Controls tested HK416 hip-fire grouping: 3 scattered, 0 almost pinpoint. |
| Shooting spread | 🟢 **Confirmed** (tested weapon) | `ShootingSpread` | Controls tested Benelli hip-fire grouping: 0 tight, 3 wider; ADS untested. |
| Recoil rate | 🔵 **Research** | `RecoilRate` | Recoil behaviour parameter; direction of improvement needs testing. |
| Reset divider | 🔵 **Research** | `ResetRecoilDivider` | Recoil-reset parameter; direction of improvement needs testing. |
| Shotgun bounds | 🔵 **Research** | `ShotgunMinSpread` / `ShotgunMaxSpread` | Separate signed spread bounds used alongside the shotgun-spread flag. |

The four `RecoilData` member names have generated suffixes in the mapping; the [decoded evidence](weapon-handling-values.json) retains their full names.

## Evidence and reproduction

Base-stat ranges come from [the existing DataAsset investigation](weapon-data-assets.md); they have not been independently re-decoded by tag for this page. Its four/five-record positional mapping and special-weapon limitations remain in force.

Handling values come from `BP_WeaponsPickupComponent_GEN_VARIABLE` exports in pickup assets under `/Game/Inventory/Items/Pickups/Weapons/Firearms/`, plus the shared component CDO. The supplied mapping archive is `SurrounDead-5.6.1-44394996+++UE5+Release-5.6-24b12662.zip`. That identifies the mapping source, not independently the exact game patch of every exported asset.

The analysis decoded unversioned property fragments and zero masks against the component and `Recoil_Struct` schemas. All **51 exports** (49 weapons, one generic firearm template, one shared default) decoded without reported errors and consumed their property data with only a 4- or 8-byte zero trailer remaining. Explicit zero-mask values are retained as zero, including all four Crossbow recoil members. This is a structural validation, not an in-game behaviour test.

[Decoded handling evidence](weapon-handling-values.json) contains normalized derived research values rather than raw cooked exports. Only derived research data is committed; no cooked game files or mapping files are redistributed.

Source archive SHA-256 (`Items.zip`): `f87a6051868e9831fb5d0bed62af10f7cfd809e474556e51059111036450b4f2`.

## Implications for WeaponProgression

🟡 **Probable — proposed integration based on confirmed effects.** 🔵 **Research — production lifecycle and persistence.**

The existing UID tracking, milestone/config logic and UI can be reused to select and display deterministic bonuses. Damage/crit/RPM/falloff use the already researched item-stat route. WeaponHandlingResearch has now demonstrated live component writes and observable directional recoil, **vertical and horizontal random deviation**, and spread effects. Production integration still needs attachment/equip lifecycle and persistence validation.

For integration, capture the equipped instance's effective baseline and verify reconstruction after weapon swaps and attachment changes. If scaling signed recoil, scaling toward zero preserves its sign (for example, −0.35 × 0.9 = −0.315). For deviation, the current evidence supports reducing the positive magnitude toward zero to make aim/recoil behaviour more predictable; small percentage reductions are a plausible milestone reward, but the exact perceptual scaling still needs validation. Do not repeatedly compound bonuses onto an already modified value or mutate a shared class default. The safe direction for `RecoilRate` and `ResetRecoilDivider`, inheritance resolution and persistence all remain research tasks.