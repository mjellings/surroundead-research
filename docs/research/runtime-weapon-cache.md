# Runtime Weapon Cache and Duplicate Live-Slot Research

> Findings from WeaponProgression v0.14.0 development against SurrounDead 0.8 / Unreal Engine 5.6. This page documents observed SurrounDead/UE4SS runtime behaviour rather than reproducing the mod implementation.

## Status

🟢 **Confirmed 2026-09-05** — the live firearm resolution path can be safely cached per physical weapon GUID, provided the cached object is revalidated before use and a full `JSI_Slot_C` scan remains available as a fallback.

🟢 **WeaponProgression v0.14.0** is the current stable cache-enabled baseline. v0.13.0 remains the frozen pre-cache stable baseline.

## Why caching was investigated

The proven pre-cache mutation path enumerated all live `JSI_Slot_C` objects to locate the slot whose `ItemUniqueID` matched the physical firearm GUID returned by `GetEquipmentUID`.

That route is reliable, but `FindAllOf("JSI_Slot_C")` was being performed on ordinary firearm activity. Test sessions commonly exposed hundreds of candidates, so repeated full scans were unnecessary once the correct live representation had already been resolved.

The v0.14 research therefore tested whether the resolved slot could be cached without weakening the correctness guarantees established by v0.13.

## Duplicate `JSI_Slot_C` representations for one physical GUID

🟢 **Confirmed** — a single physical firearm GUID may simultaneously appear on more than one live `JSI_Slot_C` UObject.

Repeated tests with three firearms produced a consistent pattern in which one same-GUID candidate contained the firearm's expected `ItemStats`, while another same-GUID candidate exposed zero decoded firearm stats.

Examples observed during one session:

```text
Crusher
  index 414 -> ItemStats=5  POPULATED
  index 429 -> ItemStats=0  EMPTY

Hunting Rifle
  index 435 -> ItemStats=4  POPULATED
  index 448 -> ItemStats=0  EMPTY

Black Ops USP
  index 454 -> ItemStats=5  POPULATED
  index 469 -> ItemStats=0  EMPTY
```

The zero-stat objects are described here as **empty/stub representations**. Their exact game-side purpose has not yet been established.

### Important consequence

🔴 A matching physical GUID by itself is **not sufficient** to identify the useful live stat/mutation representation.

🔴 Scan order is also not authoritative. An experimental policy that selected the highest-index same-GUID candidate repeatedly selected the zero-stat representation.

🟢 In the tested firearm paths, `ItemStats` population is a substantially stronger empirical discriminator: prefer a same-GUID candidate that exposes the expected decoded firearm stat set over a same-GUID candidate with zero decoded firearm stats.

## Current populated-slot selection rule

The tested safe discovery strategy is:

```text
GetEquipmentUID
        ↓
physical weapon GUID value
        ↓
FindAllOf("JSI_Slot_C")
        ↓
collect every readable same-GUID candidate
        ↓
read current ItemStats from each candidate
        ↓
reject zero-stat candidates when a populated candidate exists
        ↓
select populated representation
        ↓
use that slot's native ItemUniqueID wrapper for UpdateStatByUID
```

🟢 Different firearm families may expose different stat counts. For example, the tested Crusher and Black Ops USP exposed five decoded firearm stats, while the tested Hunting Rifle exposed four because it had no RPM entry. Selection therefore must not require an absolute count of five; the meaningful distinction in current testing is **populated versus zero decoded firearm stats**.

🟡 If multiple same-GUID candidates are simultaneously populated, current evidence does not yet establish a universally authoritative tie-breaker. Such a state should remain diagnostic rather than being treated as solved by scan index alone.

## Cache validation

🟢 **Confirmed** — a cached slot should not be trusted merely because a Lua reference still exists.

Before reuse, the tested cache path validates:

1. the cached UObject still passes `IsValid()`;
2. its current `ItemUniqueID` still decodes to the expected physical weapon GUID;
3. its current `ItemStats` are re-read rather than relying on a previously captured stat table;
4. the re-read slot still exposes a populated decoded firearm stat set.

If validation fails, the cache entry is discarded and the proven full `JSI_Slot_C` discovery path runs again.

This retains the full scan as correctness/recovery infrastructure while avoiding it during normal stable combat.

## Real UObject lifecycle invalidation

🟢 **Confirmed 2026-09-05** — cached `JSI_Slot_C` UObjects can genuinely become invalid during normal inventory/equipment lifecycle activity.

After drop/pickup and equipment reshuffling tests, a previously successful cached Crusher slot later failed `IsValid()`. The cache correctly rejected it and performed a full scan.

The recovery scan found two same-GUID representations:

```text
index 633 -> ItemStats=5
index 674 -> ItemStats=0
```

The populated representation was selected, all expected persistent Crusher stats were read correctly, and normal progression continued.

This is direct evidence that cache invalidation/fallback is not merely defensive theory: it is required by observed runtime UObject lifecycle behaviour.

## Drop/pickup and equipment reshuffling

🟢 Three progressed physical firearms were dropped, picked up again and moved through different equipment positions. One weapon temporarily entered a different equipment slot and was manually rearranged.

The physical GUIDs remained the progression identities. Populated-slot resolution continued to locate the correct live representations, and the three weapons retained independent stats/progression without cross-contamination.

This reinforces the distinction between **physical item identity** and **current equipment position**.

## Fresh mutation after cache recovery

🟢 After the lifecycle/duplicate-slot tests, the Crusher continued progressing and received another permanent Critical Hit Chance reward.

Observed transition:

```text
base Critical Hit Chance = 14
upgrade count            = 4
target                   = 18
live write               = 17 -> 18
```

An independent delayed full scan again found one populated and one empty same-GUID representation, selected the populated representation, and read Critical Hit Chance `18` together with the other expected persistent targets.

Verification completed with no mismatches.

This demonstrates that the populated representation selected after lifecycle changes is not merely readable: its native `ItemUniqueID` wrapper remains suitable for the proven `UpdateStatByUID` mutation route.

## v0.14.0 production smoke test

🟢 The exact production v0.14.0 file was tested after a full game restart.

On first Crusher activity the cache was empty, so the resolver performed one full scan and selected the populated same-GUID representation. The game had restored Critical Hit Chance to the original/live value `17`; the mod-owned persistence layer reconstructed the saved target and reapplied `17 -> 18`.

Independent verification confirmed all expected targets.

Subsequent firearm activity reused the validated cache repeatedly rather than performing a full discovery scan for every event.

The same production session then reached Crusher Level 12 and awarded a second RPM upgrade:

```text
base RPM       = 958.5
upgrade count  = 2
target RPM     = 997.223
live write     = 977.67 -> 997.223
```

The native level-up notification was issued, and an independent delayed scan read RPM `997.223` and verified every expected Crusher stat with no mismatches. The following firearm activity returned to the normal cache-hit path.

## Performance/correctness model

The resulting architecture is:

```text
ordinary firearm activity
        ↓
physical GUID
        ↓
validated populated cached slot ──────────────┐
        │                                      │
        │ invalid/missing/stale                │ valid
        ↓                                      ↓
full JSI scan                           read fresh ItemStats
        ↓                                      │
collect same-GUID candidates                   │
        ↓                                      │
select populated representation                │
        ↓                                      │
cache it ──────────────────────────────────────┘
        ↓
reconcile/apply progression

level-up/stat write
        ↓
independent delayed full-scan verification
```

The optimisation therefore changes **how often discovery is required**, not the identity or mutation rules established by the earlier research.

## Current rules

- `GetEquipmentUID` identifies the physical firearm GUID.
- The live `JSI_Slot_C.ItemUniqueID` wrapper remains the proven wrapper for `UpdateStatByUID`.
- Same GUID does not imply one unique live `JSI_Slot_C` UObject.
- Prefer a populated same-GUID firearm representation over a zero-stat same-GUID representation.
- Re-read `ItemStats` when validating a cache hit.
- Treat failed `IsValid()`, GUID mismatch or zero decoded firearm stats as cache invalidation.
- Retain full `JSI_Slot_C` scanning as fallback/recovery.
- Retain independent delayed full-scan verification after stat mutation.
- Do not use scan index alone as an authority signal.

## Open research

🔵 Determine the game-side purpose and lifecycle of the zero-stat same-GUID `JSI_Slot_C` representations.

🔵 Establish a stronger authority discriminator if multiple same-GUID candidates are ever observed with populated firearm stat sets simultaneously.

🔵 Test cache behaviour across additional firearm families and unusually long inventory-heavy sessions.

🔵 Continue mapping live `JSI_Slot_C` representations back to exact serialized inventory/container records.
