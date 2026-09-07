# Runtime Standalone UMG Research

> SurrounDead 0.8 / Unreal Engine 5.6, tested through UE4SS while developing UIResearch and the Weapon Progression status-card UI.

This page records reusable findings about creating a complete standalone Unreal UMG interface directly from UE4SS Lua, without requiring a custom cooked Widget Blueprint or `.pak`.

## Result

🟢 **Confirmed 2026-09-07** — SurrounDead 0.8 can display a fully standalone Lua-created `UserWidget` assembled from native UMG classes at runtime.

The proven high-level route is:

```text
UE4SS Lua
→ StaticConstructObject
→ UserWidget
→ WidgetTree
→ CanvasPanel root
→ native UMG child widgets
→ AddToViewport
```

This is distinct from the earlier tooltip work, which extended widgets that SurrounDead had already created. The standalone route creates a new widget tree owned by the mod.

## Proven widget construction

The tested implementation dynamically constructed these native classes:

```text
/Script/UMG.UserWidget
/Script/UMG.WidgetTree
/Script/UMG.CanvasPanel
/Script/UMG.Border
/Script/UMG.TextBlock
/Script/UMG.ProgressBar
```

A suitable live Unreal object, such as the game instance or player controller, can be used as the outer for the root widget. Child widgets are constructed with an appropriate widget/tree/container outer.

The root sequence used successfully was:

```text
construct UserWidget
→ construct WidgetTree with UserWidget as outer
→ assign root.WidgetTree
→ construct CanvasPanel with WidgetTree as outer
→ assign tree.RootWidget
→ add child widgets
→ root:AddToViewport(z_order)
```

## Canvas layout

🟢 `CanvasPanel:AddChildToCanvas` returns a canvas slot that can be positioned and sized at runtime.

The tested layout uses `KismetMathLibrary:MakeVector2D` and then:

```text
slot:SetPosition(...)
slot:SetSize(...)
```

This was sufficient to build a compact status card with title, weapon name, divider, Level / XP rows and an XP progress bar.

## Styling

🟢 Native UMG widgets can be styled from Lua without a custom Blueprint asset.

Tested examples include:

- `Border:SetBrushColor`;
- `TextBlock` font-size mutation through its `Font` property;
- `TextBlock:SetColorAndOpacity`;
- `ProgressBar:SetFillColorAndOpacity`;
- progress track tint through `ProgressBar.WidgetStyle.BackgroundImage.TintColor`;
- `Widget:SetRenderOpacity` for switching between stat and status presentations.

The resulting Weapon Progression card visually matched the game's subdued native UI style closely enough to be useful as a production-facing panel.

## FText safety remains mandatory

🟢 The same FText rule discovered during tooltip research applies to standalone widgets.

Do not rely on direct Lua-string coercion where Unreal expects `FText`. The safe route remains:

```text
Lua string
→ /Script/Engine.Default__KismetTextLibrary
→ Conv_StringToText
→ valid FText
→ TextBlock:SetText
```

Direct arbitrary Lua-string use in FText-sensitive calls previously caused a native UE4SS access violation.

## Presentation/data separation

🟢 The tested design keeps the UI module presentation-only.

The standalone `ui.lua` module does not own weapon GUID resolution, database access, XP formulae or gameplay hooks. Instead, the main mod supplies a small state object such as:

```text
weapon
level
xp_percent
status
```

The UI module then renders that state.

This separation proved useful because the same UI layer can be changed without altering authoritative progression or persistence code.

## Efficient refresh behaviour

🟢 The tested UI caches previously rendered string values and only rewrites a `TextBlock` when its displayed value changes.

The provider pattern is:

```text
main.lua owns state
→ UI.SetProvider(function)
→ UI.Refresh()
→ provider returns current state table
→ render only changed values
```

The XP bar is updated independently through `ProgressBar:SetPercent`.

## Open/close lifecycle

🟢 A standalone widget can be opened and closed repeatedly without intentionally retaining the old widget tree.

The tested lifecycle is:

```text
Show
→ build if no valid root exists
→ AddToViewport
→ Refresh

Hide
→ RemoveFromParent
→ clear retained UObject references
```

`IsValid()` checks are used before reusing retained UObjects.

Weapon Progression also demonstrated a popup lifecycle using a delayed close callback and a generation token. Because `ExecuteWithDelay` callbacks cannot be cancelled directly, incrementing a generation counter safely makes older pending close callbacks no-ops when the popup is reopened or its timer is restarted.

## Active-weapon UI bridge

🟢 A separate runtime bridge was demonstrated for supplying the standalone status card with the currently selected physical weapon.

Observed useful callbacks include:

```text
BP_JigHelperComp:GetActiveWeapon
BP_JigHelperComp:GetActiveWeaponSlot
JSIContainer:GetEquippedItemRef
```

The tested bridge maps the active slot tag to known player containers, obtains a physical `JSI_Slot_C.ItemUniqueID`, and then associates that GUID with mod-owned progression data.

Because SurrounDead may expose transient or alternate/stub slot objects during combat and weapon switching, UI identity should not blindly replace a known-good physical UID on every callback. The tested implementation queues ambiguous candidates and promotes them when the authoritative active-weapon name confirms the switch.

A one-time top-level `JSI_Slot_C` scan can also recover the active DB-backed weapon when the relevant callbacks have not fired since the Lua mod loaded.

## What this changes for mod UI design

Before this test, a plausible route for custom SurrounDead interfaces was:

```text
Unreal Editor
→ custom Widget Blueprint
→ cook assets
→ package .pak
→ Blueprint mod loader
→ Lua/Blueprint bridge
```

That route may still be useful for custom art, complex animations or assets unavailable in the base game, but it is **not required for ordinary standalone runtime UI**.

For interfaces composed from standard UMG controls, the confirmed Lua-only route avoids cooking/version compatibility and extra Blueprint-loader dependencies.

## Reusable implementation rules

1. Construct a real `UserWidget` + `WidgetTree`; do not require an existing game tooltip or panel as a parent.
2. Use live valid Unreal objects as outers and validate retained UObjects before reuse.
3. Build text through `KismetTextLibrary:Conv_StringToText` before calling FText APIs.
4. Keep authoritative gameplay/state logic outside the presentation module.
5. Clear retained widget references after `RemoveFromParent` rather than assuming removed widgets remain safe to reuse indefinitely.
6. Treat weapon/inventory callbacks as noisy lifecycle signals; validate physical identity rather than assuming every callback is authoritative.
7. Use Blueprint/cooked UI only when Lua-created native UMG is genuinely insufficient.

## Open research

🔵 Button construction and click/delegate handling from Lua.

🔵 Scroll boxes, lists and reusable row factories for larger data sets.

🔵 Image/texture assignment and discovery of reusable SurrounDead art assets.

🔵 Input focus, mouse cursor and modal behaviour for interactive full-screen panels.

🔵 DPI scaling / anchors for resolution-independent layout beyond the current fixed-position status card.

🔵 Whether custom Blueprint/UMG `.pak` assets remain useful or compatible on the current 0.8 / UE 5.6 runtime for UI features that native Lua construction cannot cover.
