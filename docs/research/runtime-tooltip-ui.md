# Runtime Tooltip / UI Research

> SurrounDead 0.8 / Unreal Engine 5.6, tested through UE4SS while developing the original Weapon Progression research mod.

This page records reusable findings about SurrounDead's native firearm tooltip widgets, safe runtime text mutation, and a related live-slot selection caveat discovered during the same investigation.

For the later result proving that a complete standalone `UserWidget` can be assembled directly from Lua, see [Runtime Standalone UMG Research](runtime-standalone-ui.md).

## Hover-tooltip entry point

🟢 **Confirmed 2026-09-05** — the native item hover tooltip updates through:

```text
/Game/JigSInventory/Jigsaw/Widgets/HoverDrag/Hover/OnHoverTooltipWidget.OnHoverTooltipWidget_C:Update
```

The update path can be used to correlate the hovered inventory item with its physical item UID and a mod-owned per-item record.

## Numeric firearm stat widgets

🟢 **Confirmed** — numeric firearm rows are constructed as `BP_StatW_C` widgets.

Observed fields include:

```text
StatName    -> GameplayTag struct
VectValue   -> Vector2D struct
Prefix
ExtraText
TextBlock   -> displayed numeric value
TextBlock_56 -> displayed stat label
ExtraTxt    -> displayed suffix such as % or M
```

For the tested Crusher, the native tooltip constructed rows for:

```text
Damage
Critical Hit Multiplier
Critical Hit Chance
RPM
Damage Falloff Range
```

The displayed values reflect the current live weapon stats, including runtime values reconstructed by Weapon Progression after restart.

🟢 Replacing `TextBlock` after the widget has been constructed successfully changes the value displayed in the native tooltip without creating a separate overlay UI.

This was used to display compact enhancement suffixes such as:

```text
Damage                  96.8 (+3.8)
Critical Hit Multiplier 27% (+2)
Critical Hit Chance     18% (+4)
RPM                     997.2 (+38.7)
Damage Falloff Range    85.8 M (+3.3)
```

Zero-value enhancement suffixes are suppressed, so a stat with no earned upgrade remains visually vanilla.

## Prefer GameplayTag identity over display-label matching

🟢 **Confirmed** — identifying numeric stat rows by the underlying GameplayTag is more reliable than using the rendered FText label as the primary key.

Known tags used successfully:

```text
Jig.Stat.FirearmDamage
Jig.Stat.CriticalHitMultiplier
Jig.Stat.CriticalHitChance
Jig.Stat.FirearmRPM
Jig.Stat.DamageFallOff
```

Display labels can still be useful as a fallback, but the tag is the authoritative identity for the tested firearm rows.

## Text-stat grid anatomy

🟢 **Confirmed** — the tooltip's text-stat section is a `UniformGridPanel` named `TextStatsGrid`.

Existing text rows are `BP_StatTextW_C` widgets. On a tested Crusher the layout was:

```text
row 0, col 0 -> Firemodes
row 0, col 1 -> Caliber
row 1, col 0 -> Silenced
```

Observed `BP_StatTextW_C` fields include:

```text
StatName
StatValue
TextBlock_56 -> displayed label
TextBlock    -> displayed value
```

This leaves normal grid cells available for additional native-looking information.

## Dynamically adding native tooltip rows

🟢 **Confirmed 2026-09-05** — additional `BP_StatTextW_C` widgets can be created at runtime and inserted into the existing `TextStatsGrid` rather than drawing a separate custom UI.

Weapon Progression successfully added:

```text
Level
XP
Kills
```

Example tested presentation:

```text
Silenced        Level   12
XP              18.5%   Kills   62
```

The XP value is mod progression toward the next weapon level, not the player's vanilla level XP.

## Blueprint Construct timing caveat

🟢 **Confirmed** — newly created `BP_StatTextW_C` widgets run their Blueprint `Construct` path after creation. Writing custom label/value text too early is overwritten by the widget's defaults.

Observed failed result:

```text
Name
Name
Name
```

The reliable sequence is:

```text
create BP_StatTextW_C
→ add widget to TextStatsGrid
→ allow Blueprint Construct to complete
→ short delayed callback
→ write TextBlock_56 and TextBlock
→ custom label/value remains visible
```

A very short delayed write was sufficient in the tested build.

This is a useful general UE4SS/UMG rule: when dynamically creating Blueprint widgets, treat Blueprint construction as potentially authoritative over initial property values.

## FText safety

🟢 **Confirmed elsewhere in the same investigation** — arbitrary Lua strings should not be passed directly where Unreal expects an `FText`; that caused a native access violation in testing.

The safe conversion route is:

```text
Lua string
→ /Script/Engine.KismetTextLibrary:Conv_StringToText
→ valid FText
```

For existing TextBlock widgets, their normal text setter can then be called with a properly constructed FText.

## Live `JSI_Slot_C` duplicate/stub caveat

🟢 **Confirmed during Weapon Progression testing** — more than one live `JSI_Slot_C` can carry the same physical weapon GUID at the same time.

A GUID match alone is therefore not enough to select the authoritative live weapon representation.

Observed duplicate candidates included zero-stat/stub slots. The production resolver therefore:

```text
FindAllOf("JSI_Slot_C")
→ collect every slot whose ItemUniqueID matches the physical GUID
→ read/decode ItemStats
→ reject zero-stat matches
→ prefer a populated candidate
```

When multiple populated candidates exist, scan order alone should not be treated as authoritative. The tested Weapon Progression build chose the candidate with the richest decoded firearm-stat set, with a stable tie-break, and logged the ambiguity.

This refines the earlier simpler rule of "exact GUID match = live weapon slot".

## Current production tooltip result

🟢 Tested across Crusher, Hunting Rifle and Black Ops USP.

The native tooltip successfully showed:

- rounded live stat values;
- non-zero earned bonus suffixes only;
- percentage/unit formatting folded into the displayed value;
- weapon Level;
- weapon XP as percentage progress to the next level;
- per-weapon kill count.

The Hunting Rifle correctly lacked an RPM row because that physical weapon did not expose an RPM stat, reinforcing that tooltip/stat logic must remain data-driven rather than assuming five firearm stats.

## Tooltip extension versus standalone UI

🟢 **Confirmed 2026-09-07** — extending SurrounDead's existing tooltip is no longer the only demonstrated Lua UI technique.

The tooltip route remains useful when the information belongs naturally inside an existing game panel and native visual consistency is desired. Separately, a complete mod-owned `UserWidget` + `WidgetTree` can now be constructed directly from Lua and added to the viewport. That later route is documented in [Runtime Standalone UMG Research](runtime-standalone-ui.md).

The two techniques are complementary:

```text
existing game context needed
→ extend existing SurrounDead widget

dedicated mod panel needed
→ build standalone Lua UMG widget tree
```

## Reusable implementation rules

1. Identify physical items by stable GUID value, but do not assume one GUID maps to only one current `JSI_Slot_C` UObject.
2. For stat rows, prefer GameplayTag identity over display text.
3. Modify the game's existing UMG widgets where possible for native visual consistency.
4. When creating Blueprint widgets dynamically, apply final custom text after Blueprint `Construct` has run.
5. Build real `FText` values through Unreal's text library rather than relying on Lua-string coercion.
6. Keep calculated/storage precision separate from presentation precision; Weapon Progression retains full internal values while the tooltip rounds for readability.
7. Use a standalone Lua-created widget tree when a dedicated panel is more appropriate than extending an existing game widget.

## Open research

🔵 Determine whether the same tooltip widget anatomy is shared by melee weapons, armor and other item classes.

🔵 Determine a stronger authoritative rule when several populated `JSI_Slot_C` candidates share one GUID.

🔵 Map additional native tooltip panels and reusable row/widget classes for future mods.

🔵 Continue standalone UMG research for interactive controls, lists, images and input/focus handling; see [Runtime Standalone UMG Research](runtime-standalone-ui.md).
