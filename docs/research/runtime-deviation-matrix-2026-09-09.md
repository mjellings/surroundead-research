# Runtime deviation matrix — HK416 — 2026-09-09

[Research index](README.md) · [Weapon reference](weapons.md) · [Runtime weapon stats](runtime-weapon-stats.md)

SurrounDead **0.8 / UE5.6** runtime research using UE4SS Lua against the live equipped HK416.

**Confidence key:** 🟢 **Confirmed** — demonstrated in the stated test · 🟡 **Probable** — supported interpretation · 🔵 **Research** — unresolved.

## Purpose

This test independently isolated `VerticalRandomDeviation` and `HorizontalRandomDeviation` while ordinary directional recoil was neutralised. It was designed to determine whether the two fields produce observable aim displacement, whether positive values imply a fixed direction, and whether the two fields combine.

## Targeting and baseline

🟢 **Confirmed.** WeaponHandlingResearch v0.6 used the established active-weapon route:

```text
unique locally controlled BP_PlayerCharacter_C
→ BP_JigHelperComp:GetActiveWeapon
→ active BP_HK416Pickup_C actor
→ actor.BP_WeaponsPickupComponent
→ component ownership verification
```

The captured live HK416 baseline was:

| Field | Value |
| --- | ---: |
| RecoilVertical | -0.05 |
| RecoilHorizontal | 0.1 |
| VerticalRandomDeviation | 0.015 |
| HorizontalRandomDeviation | 0.025 |
| HipfireSpread | 3 |
| ShootingSpread | 0 |
| RecoilRate | 2 |
| ResetRecoilDivider | 1.5 |
| ShotgunMinSpread | -25000 |
| ShotgunMaxSpread | 25000 |

The UE4SS log reported v3.0.1 Beta #0, Git SHA `24b12662`, and detected Unreal Engine 5.6.

## Preset matrix

All laboratory presets set `RecoilVertical=0` and `RecoilHorizontal=0` so ordinary directional recoil did not mask the deviation fields.

| Preset | V recoil | H recoil | V deviation | H deviation | Purpose |
| --- | ---: | ---: | ---: | ---: | --- |
| A | 0 | 0 | 0 | 0 | Stable control |
| B | 0 | 0 | 1 | 0 | Isolate exaggerated vertical deviation |
| C | 0 | 0 | 0 | 1 | Isolate exaggerated horizontal deviation |
| D | 0 | 0 | 1 | 1 | Combine both deviation components |
| E | 0 | 0 | 0.0075 | 0 | Vertical deviation at 50% of captured HK416 baseline |
| F | 0 | 0 | 0 | 0.0125 | Horizontal deviation at 50% of captured HK416 baseline |

The log recorded successful write/readback verification for each preset and retained the expected values during sampled `Svr_WeaponShot` and `StartRecoil` events.

## Visual results

A screen recording was matched to the logged A→F sequence. The capture was visibly stuttery/compressed, so it is suitable for qualitative comparison but **not** for precise frame-time, pixel-distance or angular measurements.

### A — zero control

🟢 **Confirmed.** With both recoil and deviation components at zero, aim was substantially more stable and provided the control condition for the matrix.

### B — `VerticalRandomDeviation=1`

🟢 **Confirmed.** With only vertical deviation enabled, firing introduced strong random vertical aim displacement compared with A.

Most importantly, the positive value `1` produced displacement **both above and below** the starting aim. Therefore the sign of `VerticalRandomDeviation` is not a direct screen-space up/down direction in the way a signed directional-recoil value can be.

### C — `HorizontalRandomDeviation=1`

🟢 **Confirmed.** With only horizontal deviation enabled, firing introduced predominantly lateral random aim displacement compared with A and B.

A positive value produced displacement in **both left and right directions**, so `HorizontalRandomDeviation` is likewise a magnitude/range-like parameter rather than a direct left/right sign.

### D — both deviations at `1`

🟢 **Confirmed.** Enabling both deviation fields produced two-dimensional random aim wandering. This is consistent with separate vertical and horizontal random components being combined during recoil/aim displacement.

### E/F — half-baseline values

🟢 **Confirmed for the applied values; qualitative for visible magnitude.** The HK416's captured baseline values were halved to `0.0075` vertical and `0.0125` horizontal and tested independently with directional recoil still zero.

The visual effects were subtle compared with the exaggerated `1` presets. This is useful evidence for progression design: small reductions can plausibly improve predictability without immediately producing a recoil-free or laser-like weapon. It does **not** prove a linear relationship between the raw parameter and visible movement.

## Conclusions

### `VerticalRandomDeviation`

🟢 **Confirmed — runtime-observed:** controls the magnitude of a random vertical aim/recoil displacement component on the tested HK416.

🟢 **Confirmed — runtime-observed:** a positive value can move aim both upward and downward; the value is not a direct direction sign.

### `HorizontalRandomDeviation`

🟢 **Confirmed — runtime-observed:** controls a predominantly horizontal random aim/recoil displacement component on the tested HK416.

🟢 **Confirmed — runtime-observed:** a positive value can move aim both left and right; the value is not a direct direction sign.

### Combined behaviour

🟢 **Confirmed — runtime-observed:** the two fields combine to produce two-dimensional random displacement when both are non-zero.

🟡 **Probable:** the fields behave conceptually like magnitudes/ranges used to generate random vertical and horizontal components. No exact `random(-value,+value)` formula is asserted.

## Important correction on recovery

The footage supports deviation affecting the recoil/aim displacement event itself. It does **not** establish that either deviation field directly chooses the final post-recovery aim position.

The aim visibly moves back toward the starting region after shots, but recovery timing and reset behaviour have not been isolated. Therefore:

- 🔵 `RecoilRate` remains research.
- 🔵 `ResetRecoilDivider` remains research.
- 🔵 The exact relationship between deviation, kick, interpolation and recovery remains research.

A future high-frame-rate test should isolate those recovery fields independently.

## Progression implications

🟡 **Probable.** Both deviation fields are now credible candidates for mastery/progression bonuses. Reducing their magnitude should make recoil behaviour more predictable while preserving directional recoil and weapon character.

A safe implementation should scale the captured effective baseline rather than repeatedly subtracting from an already modified live value. Small percentage reductions are preferable until proportional response across weapon families is better characterised.

## Remaining questions

- Exact internal units and random distribution.
- Whether response is linear across realistic intermediate values.
- Behaviour on other firearm families and very large baseline deviations.
- Attachment/equip interactions and reconstruction after weapon swaps.
- Relationship to `RecoilRate` and `ResetRecoilDivider`.
- Whether negative deviation values have meaningful or clamped behaviour; no need to assume they are useful for progression.
