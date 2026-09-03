# `Player.sav`

🟢 **Confirmed** — `Player.sav` contains player progression and status values that are substantially smaller/simpler to work with than the inventory-heavy `PlayerInfo.sav`.

Observed properties include:

```text
CurrentLevel
CurrentXP
CurrentMaxXP / NeededXP
SkillPoints
Health / CurrentHealth
CurrentStamina
```

## Skills

Observed skill-related properties include groups for:

- Fitness
- Strength
- Toughness
- Sneaking
- First Aid
- Marksmanship
- Reloading
- Thief
- Fishing
- Scavenging

Some saves contain spelling variants, including the observed typo `CurrentToughessLevel`, alongside more conventional variants in other contexts.

## Write-path lesson

🟢 **Confirmed during controlled editing** — changing a property value safely requires preserving the surrounding compressed-wrapper structure. A previous write experiment corrupted the save when the Oodle wrapper `BlockSize` field was incorrectly rewritten. In the examined wrapper, that field remained `131072`, while compressed/uncompressed size fields needed to reflect the newly compressed payload.

This is a useful general warning: a successful decompression/recompression cycle does not by itself prove that wrapper metadata has been reproduced correctly.
