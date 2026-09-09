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
→ existing Lua ui.lua module is loaded
→ custom UMG panel is shown
```

The successful prototype displayed:

```text
DAVE'S GUNSMITH
WEAPON PROGRESSION SERVICE
Reroll progression bonuses (prototype)
```

The same research script could hide the UI and destroy only its own spawned NPC.

This means a mod can reuse a native SurrounDead human NPC shell and native interaction prompt while replacing the service experience with its own Lua UMG, without requiring a custom cooked character or Widget Blueprint for the confirmed prototype.

## Reusing a Lua UI module

🟢 The NPC prototype successfully loaded the existing Weapon Progression presentation module from a sibling UE4SS mod directory and displayed it after native NPC interaction.

An initial relative-path assumption failed because UE4SS's process working directory was not the mod script directory. The working approach derives the currently executing script path and resolves the sibling mod from there.

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

## Current gunsmith direction

🔵 The intended service is a weapon-progression reroll NPC.

The current design is to preserve:

- physical weapon UID
- captured/base weapon stats
- weapon level
- XP
- kills
- mastery rank
- deterministic milestone bonuses

and reroll only the ordinary random progression reward history earned by that weapon.

Conceptually:

```text
current weapon
→ determine number of earned ordinary rewards
→ replay ordinary reward selection rules
→ generate a new upgrade distribution
→ preserve milestone bonuses
→ rebuild effective stats from authoritative base + ordinary + milestone data
```

The exact number of ordinary rewards must be derived from the production progression semantics rather than assuming `level` or `level - 1`.

🔵 A future UI should show the current weapon and its current progression distribution, then offer a controlled reroll/preview/accept path rather than immediately mutating the weapon.

## UI limitation found in the first gunsmith prototype

🔵 The existing compact Weapon Progression status card is sufficient to prove the NPC handoff but is not yet sized/layouted for the longer gunsmith service text. The prototype's `WEAPON PROGRESSION SERVICE` line overflowed the current panel width.

This is a presentation issue rather than an interaction failure. A dedicated service layout or wider configurable panel is the next UI step.

## Next research targets

1. Find the actual property/data source used by the native QuestGiver interaction label so `Settlement Leader` can become `Dave`.
2. Build a dedicated gunsmith service layout using the proven Lua UMG framework.
3. Bridge the active physical weapon and Weapon Progression DB state into the service UI.
4. Verify the exact ordinary-reward count for a weapon at each level.
5. Generate a non-destructive proposed reroll and compare before/after distributions.
6. Only after preview logic is proven, add acceptance, atomic DB persistence, live stat reconciliation and delayed verification.
7. Add currency cost only after the reroll transaction itself is safe.
8. Place the NPC at one of several hand-picked world coordinates on save/session load; ground placement and session persistence remain future work.

## Publication boundary

This page records derived runtime findings only. Cooked SurrounDead assets used during local research are not redistributed by this repository.
