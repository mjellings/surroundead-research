# UE4SS / Runtime Research

This section documents observations about SurrounDead's live Unreal objects and functions, primarily on **0.8 / Unreal Engine 5.6**.

It is deliberately about the **game's exposed runtime behaviour**, not redistribution of third-party mod code.

## Environment

🟢 SurrounDead 0.8 moved from an earlier Unreal Engine version to **UE 5.6**, which can affect UE4SS compatibility and assumptions made by older mods.

UE4SS provides useful reverse-engineering facilities including Lua scripting, UObject reflection, hooks, property inspection and SDK/header dumping.

## Hook registration timing

🟢 **Do not assume Blueprint `UFunction`s are available when a Lua mod's `main.lua` first executes.**

This was confirmed during WeaponProgression integration testing on SurrounDead 0.8 / UE 5.6. The Lua mod itself loaded successfully, but immediate calls to `RegisterHook` for known-valid Blueprint functions such as `BP_JigHelperComp:GetEquipmentUID`, `LevellingComponent:AddXP`, `BP_MasterZombie:Death`, and `BP_FirearmPickup:SERVER_DamageEvent` all failed because those `UFunction`s were not yet present in memory.

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

- an inventory-add path exposes an item data asset plus a UID;
- an item UID can be resolved to a live inventory slot object;
- the resolved object can expose an `ItemStats` array;
- those stat entries correspond to values visible in-game;
- GameplayTag-backed stat identities can be decoded at runtime;
- the game's own stat-update function can change live weapon stats;
- modified weapon stats are subsequently serialized by SurrounDead into its save data;
- item UIDs observed during this work remain useful across save/reload for associating external metadata with the same item instance.

See [Runtime weapon-stat research](../research/runtime-weapon-stats.md) for the detailed evidence.

## Important distinction

Some of these discoveries arose while diagnosing an existing third-party mod that was incompatible with the current game/runtime environment. The third-party mod itself is not part of this repository. Only independently observed information about SurrounDead's runtime interfaces is documented here.

## Research direction

🔵 Useful next targets include reliable equipped-weapon identification, enemy damage/death events, kill attribution, equipment-slot semantics and the relationship between live inventory objects and their serialized `PlayerInfo.sav` counterparts.
