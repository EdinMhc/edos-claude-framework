# Feature: CodeQualityFixes + Buff/Debuff System
**Status:** DONE
**Started:** 2026-04-20
**Completed:** 2026-04-21

## Summary
Fixed 7 code quality issues (.NET 8 upgrade, FighterFactory index, dead code removal, ability mutation fix, nullable cleanup, DefenseBoost, Vulnerability) and implemented a full DoT/debuff system: Poison/Bleed/Burn/Necrotic now tick HP each turn with duration counters defined in Constants, Vulnerability increases incoming damage, and all status effects show clear messages with turns remaining. Build clean at 0 warnings, 0 errors. Branch: feature/buff-debuff-system.
