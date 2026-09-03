# Third-Party Blueprint Candidate Notes

> These notes come from inspecting a third-party cooked `.pak` that overrides SurrounDead's firearm Blueprint. The third-party mod is not authored by this repository's maintainer. No source code or assets from that mod are reproduced here.
>
> Because the asset is modified, names observed inside it cannot yet be assumed to be unchanged vanilla SurrounDead behaviour. Treat everything on this page as **candidate runtime surface** until confirmed independently in-game.

## Package shape

🟢 **Observed in the supplied package** — the archive contains only two cooked files under SurrounDead's firearm pickup path:

```text
BP_FirearmPickup.uasset
BP_FirearmPickup.uexp
```

The package therefore works by overriding the cooked `BP_FirearmPickup` asset rather than by using UE4SS Lua.

This strongly suggests `BP_FirearmPickup_C` is an important class to inspect during firearm runtime research, but it does not prove that every observed function/name is vanilla.

## Candidate firearm functions/events

🟡 Names present in the cooked asset include:

```text
FireBullet
BulletFired
Svr_WeaponShot
MC_WeaponShot
SERVER_DamageEvent
TryUseBullet
ChamberWeapon
Local_StartShooting
Local_StopShooting
```

These are useful UE4SS hook/reflection candidates for future experiments.

## Candidate weapon-state names

🟡 The asset also contains names including:

```text
ActiveWeapon
ActiveWeapons
PlayerActiveWeapon
WeaponOwner
CurrentMag
PendingMag
```

These may help identify the currently active/equipped weapon, but exact ownership and semantics must be confirmed against live objects.

## Candidate damage / kill-attribution surface

🟡 Particularly interesting names include:

```text
DamagedActor
Damage
HitActor
Headshot
ApplyDamage
AI Is Dead?
SERVER_DamageEvent
```

The presence of an `AI Is Dead?` call/name in the firearm asset makes the firearm damage path a promising place to investigate kill detection. However, we do not yet know whether this is a vanilla function, a function-library call, or logic altered by the third-party mod.

### Next verification step

🔵 Use UE4SS reflection/hooking against the running unmodified game to determine whether `BP_FirearmPickup_C` exposes the candidate functions above and inspect their actual parameters/behaviour.

If `SERVER_DamageEvent` and/or the `AI Is Dead?` path can be independently reproduced without the third-party `.pak`, promote the relevant findings to the normal runtime research documentation.
