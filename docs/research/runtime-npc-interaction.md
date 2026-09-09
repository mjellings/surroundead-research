# Runtime NPC Interaction Research

Research into creating native SurrounDead NPCs at runtime with UE4SS Lua and routing their normal interaction flow into custom mod UI.

## Confidence key

- 🟢 **Confirmed** — reproduced in the tested runtime.
- 🟡 **Probable** — supported by evidence but not yet fully isolated.
- 🔵 **Research** — active investigation or unresolved behaviour.
- 🔴 **Failed / obsolete** — tested approach that failed or was superseded.

## Tested environment

- SurrounDead 0.8 experimental branch
- Unreal Engine 5.6
- UE4SS 3.0.1 Beta
- Runtime Lua only; no redistributed cooked game assets are required for the confirmed path below.

## Native trader spawning

🟢 **Confirmed:** an existing native trader class can be spawned at runtime with `UWorld:SpawnActor`.

The tested `BP_SuppliesTrader_C` path produced a visible human NPC near the player. The spawned actor initialized its normal multiplayer/inventory components, presented SurrounDead's native interaction prompt, opened the normal trader interface, allowed actual trading, and could be cleanly destroyed by the research script.

This established the first useful runtime NPC route:

```text
existing cooked NPC Blueprint class
→ UWorld:SpawnActor
→ runtime actor in PersistentLevel
→ native interaction prompt
→ native game behaviour
```

🔴 Calling `JigCanInteract()` with no parameters was incorrect; the runtime function expects one parameter. The failed diagnostic call did not prevent the spawned trader from functioning normally.

## Trader interaction lifecycle

🟢 **Confirmed:** runtime tracing of the trader interaction path observed the following important callbacks:

```text
OnExecuteInteract
→ OnServerExecuteInteract
→ OnExecuteInteractDialogue
→ OnExecuteInteractEnded
```

Additional lifecycle functions such as `OnStopExecuteInteract`, `OnBeginInteract` and `OnEndInteract` are exposed by the native interaction architecture.

🟢 The generic interaction surface also exposes:

```text
GetInteractOptions(Options : Map)
SetInteractOption(Option : Struct)
OnExecuteInteract(InteractingActor, Option : Struct)
OnServerExecuteInteract(Option : Struct, ByActor, Payload)
```

🔴 Ending the local interaction immediately after `OnExecuteInteract` did **not** cancel the already-scheduled server/dialogue trader path. Roughly four seconds later the native server/dialogue chain still executed.

🔴 Broad removal of `JSIContainer_C` widgets is not a safe way to suppress trader UI. In testing this issued hundreds of `RemoveFromParent` requests and dismantled unrelated inventory UI.

### UE4SS hook semantics

🟢 For the tested Blueprint UFunctions, `RegisterHook` is useful for observing the interaction flow and handing off into mod logic.

Do not describe these Blueprint hooks as a general pre-hook suppression mechanism. Returning a value is not a demonstrated way to skip the original Blueprint implementation.

## Quest-giver architecture

The trader path proved native spawning and interaction, but a quest-giver shell is cleaner for a non-shop service NPC.

Candidate assets identified during local content research included:

```text
BP_QuestGiver
BP_QuestGiver_SettlementOfficer
AnimBP_StationaryQuestNPC
```

🟢 **Confirmed:** exact runtime spawning works for both `BP_QuestGiver_C` and `BP_QuestGiver_SettlementOfficer_C`.

🔴 `AnimBP_StationaryQuestNPC` is an animation Blueprint rather than an Actor class and is not directly spawnable as an NPC.

### Settlement Officer

🟢 The Settlement Officer is a native dialogue/service NPC rather than a trader-inventory shell.

Observed behaviour:

```text
interact
→ native service/dialogue flow
→ currency check (~£2,500 in the tested build)
→ world container is spawned
→ GPS/map marker can be created
```

This is strong evidence that SurrounDead already contains an NPC architecture suitable for service-style interactions that are not normal buy/sell shops.

### Generic QuestGiver / Settlement Leader

🟢 Exact `BP_QuestGiver_C` can be spawned as a normal human NPC.

🟢 The native game presents it as **Settlement Leader** and shows the normal `[F] ... Talk To` interaction prompt.

🟢 Animation comparison showed that the generic QuestGiver and Settlement Officer both use the same tested stationary NPC setup, including the sheriff skeletal mesh and `AnimBP_StationaryQuestNPC_C` animation class. An earlier T-pose is therefore more likely to have been an initialization/loading hiccup than missing animation configuration.

🟢 The Settlement Officer's `Dialogue` property resolves to a real dialogue Blueprint class. The generic QuestGiver's exact dialogue value remains unresolved.

## Reliable Blueprint callback context

An early trace incorrectly reported zero QuestGiver interaction events because the research script filtered callback context too aggressively.

🟢 **Confirmed:** the callback context supplied by UE4SS is a `RemoteUnrealParam`, and:

```lua
ctx:get()
```

resolves the actual `BP_QuestGiver_C` UObject.

With the incorrect filter removed, the generic QuestGiver produced the expected native interaction activity, including repeated `GetInteractOptions` calls and an `OnExecuteInteract` event when the player pressed F.

🟢 `OnExecuteInteract` supplies the player and interaction option data through UE4SS parameter wrappers. `OnServerExecuteInteract` exposes the interaction GameplayTag, player actor and Jig payload.

### UObject wrapper identity caveat

🔴 Lua userdata identity is not reliable for deciding whether two UE4SS wrappers represent the same Unreal UObject.

A tested callback and the stored spawned actor printed the exact same UObject full name, but:

```lua
actor ~= spawned
```

still evaluated as different wrappers.

🟢 In the tested research build, comparing the resolved UObject full name reliably identified the specific runtime-spawned QuestGiver.

## Native NPC → custom Lua UMG handoff

🟢 **Major milestone:** a runtime-spawned generic QuestGiver can use SurrounDead's native interaction prompt and route the player's interaction into a custom Lua-created UMG panel.

The proven chain is:

```text
UWorld:SpawnActor(BP_QuestGiver_C)
→ native Settlement Leader appears
→ native outline / [F] Talk To prompt
→ player presses F
→ BP_QuestGiver_C:OnExecuteInteract
→ ctx:get() resolves the spawned NPC
→ spawned-NPC identity check succeeds
→ Lua gunsmith UI is loaded
→ custom UMG panel is shown
```

The same research script can hide the UI and destroy only its own spawned NPC.

This means a mod can reuse a native SurrounDead human NPC shell and native interaction prompt while replacing the service experience with its own Lua UMG, without requiring a custom cooked character or Widget Blueprint for the confirmed prototype.

## Dedicated gunsmith UI

🟢 The original compact Weapon Progression card was sufficient to prove the handoff but was too small for a service screen. It was superseded by a dedicated gunsmith UI constructed directly from Lua.

🟢 A later card-layout rewrite removed the nested `Border → Canvas` geometry dependency that caused child content and the action button to extend beyond the visible panel. The current prototype uses one root Canvas coordinate space for the frame, text, dividers, stat rows and action button.

🟢 The current tested panel displays:

- weapon name
- weapon level
- mastery rank
- base → effective Damage
- base → effective Critical Hit Multiplier
- base → effective Critical Hit Chance
- base → effective RPM
- base → effective Damage Falloff
- reroll service text
- an interactive `REROLL` button

The panel has been visually verified in-game with a tracked Crusher at Level 13 / Proven II.

## UI input capture and lifecycle

🟢 Merely showing a mouse cursor was not sufficient for an interactive service panel. The first mouse click could pass through to normal game input and fire the held weapon.

🟢 The tested fix uses `WidgetBlueprintLibrary` input-mode functions when opening the gunsmith UI. `UIOnly` successfully captures UI input in the tested build, and normal `GameOnly` input is restored when the panel closes.

Observed log sequence:

```text
[DaveGunsmith.UI] input mode | UIOnly
...
[DaveGunsmith.UI] input mode | GameOnly restored
```

🟢 ESC closes the panel and restores game input.

🟢 Proximity-close behaviour has also been reproduced: moving more than roughly 5 m from the spawned gunsmith closes the service UI.

🟢 A no-firearm service response can be shown and auto-closed after approximately three seconds.

### Interaction-key debounce

🔴 A global F-to-close keybind initially consumed the same F press that opened the QuestGiver service panel, causing the UI to open and immediately close.

🟢 The current research build arms F-to-close only after a short delay following successful handoff, preventing the opening interaction press from being interpreted as a close request.

## Weapon Progression bridge

Duplicating Weapon Progression's active-weapon resolver inside the NPC mod proved unreliable, particularly when the player loaded a save with a firearm already equipped.

🔴 The duplicated resolver path was therefore superseded.

🟢 Weapon Progression now remains the authoritative owner of active-weapon resolution and publishes a small read-only cross-mod snapshot (`active_weapon.api`).

The proven read path is:

```text
WeaponProgression active-weapon resolver
→ physical weapon UID + progression record
→ active_weapon.api
→ gunsmith reads snapshot
→ gunsmith UI displays exact tracked weapon
```

🟢 The gunsmith has successfully read a tracked Crusher with a stable physical UID and correct level, kills and mastery rank.

🟢 A startup UID bootstrap was added to Weapon Progression so an already-equipped tracked firearm can be resolved without requiring the player to switch weapons first. The tested tracked-weapon path now works when approaching the NPC directly after load.

## Authoritative reroll service

🟢 **Confirmed:** the current prototype performs a real weapon-progression reroll from the gunsmith UI.

The NPC mod does **not** write Weapon Progression's database directly. Instead:

```text
player clicks REROLL
→ gunsmith writes reroll.request
→ WeaponProgression validates the physical UID and eligibility
→ WeaponProgression rerolls ordinary progression rewards
→ WeaponProgression saves authoritative data
→ existing live-stat reconciliation runs
→ active_weapon.api is refreshed
→ gunsmith refreshes the displayed stats
```

This preserves the ownership boundary: Weapon Progression owns progression state and mutation; the NPC mod is a client/service UI.

### Preserved state

🟢 The tested reroll preserves:

- physical weapon UID
- captured/base stats
- level
- XP
- kills
- mastery rank
- deterministic milestone bonuses

Only the ordinary random progression upgrade distribution is rerolled.

### Ordinary reward count

🟢 Runtime testing resolved an earlier uncertainty. A Level 11 Crusher reported `rolls=10`, and a Level 13 Crusher reported 12 ordinary progression bonuses in the service UI.

For the tested current Weapon Progression semantics:

```text
ordinary earned rewards = level - 1
```

This should still be treated as a property of the tested progression implementation rather than a vanilla SurrounDead rule.

### Reroll generation rules

🟢 The current reroll service preserves the existing number of ordinary rewards, then generates a new distribution using the same eligible-stat pool and the existing no-consecutive-stat rule when alternatives are available.

🟢 The research implementation retries a reroll when the resulting distribution is identical to the previous one, avoiding an apparently successful reroll with no visible change where alternatives exist.

### End-to-end runtime result

🟢 The tested Crusher completed multiple consecutive rerolls successfully in one session.

Observed sequence:

```text
REROLL REQUEST
→ WeaponProgression ACTIVE API update
→ GUNSMITH REROLL | SUCCESS
→ WeaponProgression ACTIVE API update
→ REROLL RESULT | SUCCESS
```

The panel then showed the newly reconciled effective stats.

## Effective-stat display edge case

🟢 Reroll testing exposed a formatting edge case once effective RPM exceeded 999. A displayed value such as `1,058.2614` was not accepted by a later numeric conversion because of the thousands separator.

The production-side formatter was updated to sanitize comma-separated numeric text before conversion.

This was a display/formatting issue, not a reroll or persistence failure.

## NPC display-name research

🔵 **Unresolved:** writing the spawned QuestGiver's `VendorName` property to an FText containing `Dave` succeeds as a property write, but the native interaction prompt still displays **Settlement Leader**.

Observed result:

```text
VendorName = Dave        ← write succeeds
native prompt = Settlement Leader
```

Therefore `VendorName` is **not proven** to own the QuestGiver interaction label. The source of the native Settlement Leader name remains a research target.

The FText write itself used the already-confirmed safe construction route:

```text
Lua string
→ KismetTextLibrary:Conv_StringToText
→ genuine FText
→ live property write
```

## Current gunsmith status

🟢 The gunsmith is now a functioning runtime service prototype rather than only an interaction/UI proof of concept.

Current confirmed capabilities:

```text
spawn native QuestGiver
→ native SurrounDead interaction prompt
→ identify exact spawned actor
→ open dedicated Lua UMG gunsmith panel
→ resolve exact held tracked firearm through WeaponProgression
→ show full progression/effective stats
→ capture mouse/UI input
→ reroll ordinary progression bonuses
→ persist through WeaponProgression's authoritative save path
→ reconcile live weapon stats
→ refresh the UI with new values
→ close via ESC / distance / interaction lifecycle
```

The reroll currently has no currency cost. This is intentional for the research prototype.

## Next research targets

1. Find the actual property/data source used by the native QuestGiver interaction label so `Settlement Leader` can become `Dave`.
2. Choose several hand-picked world coordinates for the gunsmith and spawn him at one selected location on save/session load.
3. Decide whether the chosen spawn location should remain stable for a save/session and, if so, persist the selected location index.
4. Add a currency cost only after a suitable native currency/payment path is identified and tested safely.
5. Consider a confirmation or before/after preview step before a paid reroll.
6. Continue cosmetic UI polish only where it improves native fit; the current card geometry and interaction flow are functional.
7. Ground-placement tracing can be added later if hand-picked Z coordinates prove insufficient.

## Publication boundary

This page records derived runtime findings only. Cooked SurrounDead assets used during local research are not redistributed by this repository.
