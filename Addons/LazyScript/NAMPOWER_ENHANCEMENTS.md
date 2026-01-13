# LazyScript nampower/SuperWoW Enhancements

## Overview
LazyScript has been enhanced to use nampower/SuperWoW functions when available, providing more reliable spell detection that doesn't depend on action bars being fully loaded.

## Changes Made

### 1. Detection System (`Actions.lua`)
Added automatic detection of nampower capabilities on load:
```lua
lazyScript.hasNampowerSupport = (GetSpellIdForName ~= nil and IsSpellUsable ~= nil)
lazyScript.hasNampowerRangeCheck = (IsSpellInRange ~= nil)
lazyScript.hasNampowerCastingInfo = (GetCurrentCastingInfo ~= nil)
```

### 2. Enhanced `GetSlot()` Function
- Now attempts to find spells in spellbook via `GetSpellIdForName()` first
- Caches spell IDs in `action.nampowerSpellId` for performance
- Falls back to original action bar scanning if nampower unavailable
- **Benefit**: Works immediately on login, even before UI fully loads

### 3. Enhanced `IsUsable()` Function
Uses nampower functions when available:
- **`IsSpellUsable(spellId)`** - Checks if spell is usable (reactive abilities, mana, requirements)
- **`GetCurrentCastingInfo()`** - Checks if already casting (replaces `IsCurrentAction(slot)`)
- **`IsSpellInRange(spellId, target)`** - Checks range (replaces `IsActionInRange(slot)`)
- **`GetActionCooldown(slot)`** - Still uses action bar for cooldown checks (nampower doesn't expose cooldown API)

**Important Note**: Hybrid approach - nampower for spell detection/usability, but action bar slot still needed for cooldown checks!

### 4. Enhanced `FindSpellRanks()` Function
- Uses `GetSpellSlotTypeIdForName()` to get spell slot directly from spellbook
- Uses `GetSpellNameAndRankForId()` to extract rank information
- Falls back to original texture matching if nampower unavailable
- **Benefit**: Faster, more reliable spell rank detection

### 5. Cache Management
- `DeCacheActionRanks()` now also clears `nampowerSpellId` cache
- Ensures fresh spell data after learning new ranks or spells

### 6. User Notifications (`LazyScript.lua`)
On login, displays status:
- **Green**: "LazyScript Enhanced: nampower detected - using improved spell detection"
- **Yellow**: Warnings if some nampower functions missing
- **Yellow**: Suggestion to install nampower if not detected

## Benefits

### Before (Original LazyScript)
❌ Required spells on action bars (slots 1-120)
❌ Relied on texture matching which could fail
❌ Dependent on UI loading order
❌ Errors on first login: "Couldn't find [spell] on your action bar, PLEASE ADD IT"
❌ Had to scan all 120 slots every check

### After (With nampower)
✅ **Reduced action bar dependency** - reads spell info directly from spellbook
✅ **Works on first login** - spellbook data available immediately
✅ **More reliable** - uses spell IDs instead of texture matching
✅ **Faster** - direct lookups instead of scanning 120 slots
✅ **Better range checking** - can check any target (player, mouseover, etc.)
✅ **Backward compatible** - falls back to original method if nampower not available
⚠️ **Still needs action bar slot** - required for cooldown checks (nampower limitation)

## Technical Details

### nampower Functions Used

1. **GetSpellIdForName(spellName)** → spellId
   - Gets max rank spell ID from spellbook by name

2. **IsSpellUsable(spellId)** → usable, notEnoughMana
   - Checks if spell can be cast (reactive abilities, mana, etc.)
   - **NOTE**: Does NOT check cooldowns - still need action bar for that

3. **GetCurrentCastingInfo()** → castId, visId, autoId, casting, channeling, onswing, autoattack
   - Returns info about current casting state

4. **IsSpellInRange(spellId, target)** → inRange
   - Checks if spell is in range of target
   - Returns: 1 (in range), 0 (out of range), -1 (not single target)

5. **GetSpellSlotTypeIdForName(spellName)** → slot, bookType, spellId
   - Gets spell slot number and book type from spellbook

6. **GetSpellNameAndRankForId(spellId)** → spellName, spellRank
   - Reverse lookup from spell ID to name and rank

### Fallback Behavior
If nampower is not detected, LazyScript uses original methods:
- Action bar slot scanning (1-120)
- Texture comparison
- Tooltip name matching
- Standard WoW API functions

## Installation Requirements

### Required
- **nampower.dll** (provides enhanced spell functions)
- **SuperWoW** (DLL injection framework)

### Recommended
- Latest version of nampower with all functions
- UnitXP_SP3 (for additional game improvements)

## Testing

Test the enhancements by:
1. **Fresh login** - LazyScript should work immediately without errors
2. **Check chat** - Look for green "LazyScript Enhanced" message
3. **Remove spells from action bars** - LazyScript should still work
4. **Use `/lazyscript test`** - Should complete without "spell not found" errors

## Debug Logging

Enhanced debug logging has been added to track spell usability checks.

### Enable Debug Mode
Use the command: `/lazyscript spelldebug` to toggle detailed logging on/off

The debug output shows:

### nampower Path:
```
[DEBUG] IsUsable [Sinister Strike]: Using nampower (ID: 1752)
  IsSpellUsable: 1, OOM: 0
  CD: start=12345.6, dur=1.5
  FAIL: On CD, remaining: 0.85s
```

### Action Bar Path:
```
[DEBUG] IsUsable [Sinister Strike]: Using action bar method
  Slot: 12, Usable: 1
  CD: 0/0, Current: false, Range: 1
  SUCCESS
```

### Common Failure Reasons:
- **"On cooldown, remaining: X.XXs"** - Spell still on cooldown (GCD or spell-specific)
- **"Spell not usable"** - Not enough mana, wrong form, or requirements not met
- **"Already casting this spell"** - Spell is currently being cast
- **"Out of range"** - Target is too far away
- **"No action slot found"** - Spell not on action bars (needed for cooldown check with nampower)

## Troubleshooting

### "Using standard spell detection" message
- nampower not loaded or wrong version
- Check dlls.txt includes nampower.dll
- Verify nampower version supports required functions

### Still getting "spell not found" errors
- Some spells may not have texture data
- Check if spell exists in your spellbook
- Try adding spell to action bar as fallback

### Performance issues
- Should be faster than before
- Caching prevents repeated lookups
- Cache cleared on SPELLS_CHANGED event

## Future Improvements

Possible enhancements:
- Use nampower's spell queuing for smoother casting
- Integrate spell cooldown tracking from nampower
- Add support for spell ID-based actions
- Eliminate action bar dependency entirely

## Compatibility

- ✅ Works with or without nampower
- ✅ Backward compatible with original LazyScript forms
- ✅ Compatible with Lazy<Class> addons
- ✅ No changes required to existing forms/macros

## UnitXP_SP3 Targeting Enhancements

### Overview
LazyScript has been enhanced to use UnitXP_SP3's advanced targeting functions when available, solving the "stuck on dead target" problem.

### Detection System
Added automatic detection of UnitXP_SP3:
```lua
lazyScript.hasUnitXPSupport = false
if UnitXP then
    local test, result = pcall(UnitXP, "nop", "nop")
    if test then
        lazyScript.hasUnitXPSupport = true
    end
end
```

### Enhanced `targetNearest` Action
The `targetNearest` action now uses UnitXP_SP3 when available for more reliable targeting:
```lua
function lazyScript.pseudoActions.targetNearest:Use()
    -- Use UnitXP_SP3 if available (more reliable, skips dead targets automatically)
    if lazyScript.hasUnitXPSupport then
        local success = pcall(UnitXP, "target", "nearestEnemy")
        if not success then
            TargetNearestEnemy()  -- Fallback to vanilla
        end
    else
        TargetNearestEnemy()
    end
end
```

### How It Works
1. **With UnitXP_SP3**: Uses `UnitXP("target", "nearestEnemy")` which automatically skips dead targets
2. **Without UnitXP_SP3**: Falls back to vanilla `TargetNearestEnemy()` (may target dead enemies)
3. **Explicit control**: Only retargets when you use `targetNearest` in your form

### Benefits
❌ **Before**: `targetNearest` could target dead enemies, causing rotation to fail
✅ **After**: `targetNearest` with UnitXP always finds alive enemies

### Usage Example
```
targetNearest-ifNotUnitExists=target
targetNearest-ifNotTargetAlive
```
With UnitXP_SP3, these conditions will now work reliably - dead targets are never selected by `targetNearest`.

### UnitXP_SP3 Targeting Functions Available
While LazyScript now uses `nearestEnemy` automatically, you can also access these via keybinds:
- `UnitXP("target", "nearestEnemy")` - Target nearest enemy (used by LazyScript)
- `UnitXP("target", "nextEnemyInCycle")` - Cycle to next enemy
- `UnitXP("target", "previousEnemyInCycle")` - Cycle to previous enemy
- `UnitXP("target", "mostHP")` - Target enemy with most HP
- `UnitXP("target", "nextMarkedEnemyInCycle")` - Cycle through marked enemies
- All these functions automatically skip dead targets

### Requirements
- **UnitXP_SP3.dll** installed and loaded
- **UnitXP_SP3_Addon** enabled

### Compatibility
- ✅ Works with or without UnitXP_SP3
- ✅ Falls back to vanilla behavior if UnitXP not available
- ✅ No changes required to existing forms/macros

## Version
Enhanced: November 24, 2025
Based on: LazyScript Revision 743
