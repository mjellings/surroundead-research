# UE4SS / Runtime Research

This section documents observations about SurrounDead's live Unreal objects and functions, primarily on **0.8 / Unreal Engine 5.6**.

It is deliberately about the **game's exposed runtime behaviour**, not redistribution of third-party mod code.

## Environment

🟢 SurrounDead 0.8 moved from an earlier Unreal Engine version to **UE 5.6**, which can affect UE4SS compatibility and assumptions made by older mods.

UE4SS provides useful reverse-engineering facilities including Lua scripting, UObject reflection, hooks, property inspection, runtime object construction and SDK/header dumping.

## Hook registration timing

🟢 **Do not assume Blueprint `UFunction`s are available when a Lua mod's `main.lua` first executes.**

This was confirmed during Weapon Progression integration testing on SurrounDead 0.8 / UE 5.6. The Lua mod itself loaded successfully, but immediate calls to `RegisterHook` for known-valid Blueprint functions such as `BP_JigHelperComp:GetEquipmentUID`, `LevellingComponent:AddXP`, `BP_MasterZombie:Death`, and `BP_FirearmPickup:SERVER_DamageEvent` all failed because those `UFunction`s were not yet present in memory.

This matches UE4SS's `RegisterHook` requirement: the target `UFunction` must already exist in memory at registration time.

### Recommended pattern

For SurrounDead Blueprint hooks, use delayed/retried registration rather than treating an initial registration failure as proof that the function path is wrong. A practical pattern is:

```lua
local registered = false
local attempts = 0
local maxAttempts = 40

local function tryRegister()
    attempts = attempts + 1

    local ok = pcall(function()
        RegisterHook("/Game/.../Blueprint.Blueprint_C:FunctionName", callback)
    end)

    if ok then
        registered = true
        print("Hook registered\n")
        return
    end

    if attempts < maxAttempts then
        ExecuteWithDelay(3000, tryRegister)
    end
end

ExecuteWithDelay(3000, tryRegister)
```

For several hooks, track registration separately and retry only those that have not yet succeeded. Stop retrying once all required hooks are active, and use a finite retry limit so a genuinely invalid path does not retry forever.

### Diagnostic implication

🔴 An early error such as `no UFunction with the specified name was found` is **not sufficient evidence that a Blueprint path is invalid** when it occurs during initial mod startup.

First allow the game to construct/load the relevant Blueprint classes and retry registration. A path should only be treated as disproven after appropriate delayed retries or independent object-dump/runtime evidence.

This distinction is especially important when comparing probes: a known-valid function may fail during immediate startup registration and then register successfully several seconds later.

## Current confirmed runtime findings

🟢 During live inventory/weapon research we have confirmed that:

- inventory/runtime paths expose physical item UIDs and live `JSI_Slot_C` objects;
- live slots expose `ItemStats` arrays whose GameplayTag-backed entries correspond to values visible in game;
- one physical GUID can appear on more than one live `JSI_Slot_C`, including zero-stat/stub representations;
- `BP_PlayerCharacter_C.BP_JigMultiplayer` is a valid player Jig component for the tested stat-update route;
- `BP_JigComponent:UpdateStatByUID` can change a live physical weapon when it receives the matched live slot's native `ItemUniqueID` wrapper;
- `GetEquipmentUID` is useful for identifying the physical weapon GUID value but its returned FGuid wrapper was not interchangeable with the live slot wrapper for mutation;
- runtime stat mutations in the tested Weapon Progression route do **not** survive a full game restart by themselves; mod-owned persistence and reconstruction are required for permanent progression;
- stable physical GUID values remain useful across restart for associating external metadata with the same weapon instance;
- SurrounDead's existing native tooltip widgets can be extended from Lua;
- complete standalone UMG interfaces can also be assembled directly from Lua with `StaticConstructObject`, `UserWidget`, `WidgetTree` and native UMG controls, then added to the viewport without a custom cooked Blueprint.

See [Runtime weapon-stat research](../research/runtime-weapon-stats.md), [Runtime tooltip / UI research](../research/runtime-tooltip-ui.md) and [Standalone Lua UMG research](../research/runtime-standalone-ui.md) for detailed evidence.

## Runtime UI distinction

Two separate UI techniques are now confirmed on the tested 0.8 / UE 5.6 build:

1. **Extend existing SurrounDead widgets** — useful when native visual consistency and existing game context are desired, such as adding Level / XP / Kills rows to the firearm tooltip.
2. **Create a standalone widget tree from Lua** — useful for dedicated mod panels. A tested status card uses a Lua-created `UserWidget`, `WidgetTree`, `CanvasPanel`, `Border`, `TextBlock` and `ProgressBar` and calls `AddToViewport` directly.

Custom cooked Blueprint/UMG `.pak` assets may still be useful for more elaborate interfaces, but they are not required for ordinary runtime panels built from standard native UMG controls.

## Important distinction

Some discoveries arose while diagnosing existing third-party mods that were incompatible with the current game/runtime environment. Those third-party mods are not part of this repository. Only independently observed information about SurrounDead's runtime interfaces is documented here.

## Research direction

🔵 Useful next targets include stronger authority rules for duplicate populated live weapon slots, special-weapon stat layouts, interactive standalone UMG controls, input/focus behaviour, reusable lists/images, and further mapping between live inventory objects and serialized `PlayerInfo.sav` structures.
