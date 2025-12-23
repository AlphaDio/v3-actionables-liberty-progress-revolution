# Script Errors Fix - LPR Variables

## Problem
The mod was experiencing multiple script system errors related to unset variables:

- "Failed to fetch variable for 'actionables_lpr_*' due to not being set"
- "Event target link 'var' returned an unset scope"
- "Invalid left side during comparison 'var'"
- "change_variable effect [ Variable not of the 'value' scope type. Type: empty ]"

## Root Causes Identified

1. **Missing Mod Descriptor**: The `actionables_lpr` mod was missing a `.mod` descriptor file, preventing proper loading.

2. **Variable Initialization Issues**: Initially tried using `set_country_variable`/`change_country_variable` but discovered Morgenrote uses `set_variable`/`change_variable` for country variables.

3. **Incorrect Trigger Syntax**: Used `var:variable_name >= value` in triggers instead of `check_variable = { variable_name >= value }`.

4. **Missing Failsafe Initialization**: Variables needed monthly initialization checks in case they became unset.

## Fixes Applied

### 1. Created Mod Descriptor
Created `actionables_lpr.mod` with proper metadata:
```txt
version="1.0.0"
tags={
	"Politics"
	"Gameplay"
}
name="LPR: Liberty, Progress, Revolution"
supported_version="1.7.*"
```

### 2. Fixed Variable Initialization (Following Morgenrote Pattern)
Used `set_variable` (not `set_country_variable`) in `common/history/global/00_global.txt` following Morgenrote's approach:
- `actionables_lpr_liberty`
- `actionables_lpr_progress`
- `actionables_lpr_revolution`
- `actionables_lpr_liberty_spent`
- `actionables_lpr_progress_spent`
- `actionables_lpr_revolution_spent`

### 3. Added Failsafe Variable Initialization
Added monthly initialization checks in `common/on_actions/lpr_on_actions.txt` to ensure variables exist before use.

### 4. Fixed Variable Modification (Following Morgenrote Pattern)
Updated all variable modification calls to use `change_variable` (not `change_country_variable`) in all event files:
- `events/actionables_lpr_builder_events.txt`
- `events/actionables_lpr_binary_events.txt`
- `events/actionables_lpr_big_finisher_binary_events.txt`
- `events/actionables_lpr_finisher_events.txt`
- `events/actionables_lpr_politics_events.txt`
- `events/actionables_lpr_climate_events.txt`

### 5. Moved Triggers to On_Actions
Removed event triggers and moved variable checks to `common/on_actions/lpr_on_actions.txt` using conditional random_events:
- Binary events only trigger when variable >= 5
- Big finisher events only trigger when variable >= 20
- Spent threshold events only trigger when spent >= 100 and not previously triggered
- Climate events only trigger when global variables >= 100

## Technical Details

In Victoria 3 modding (following Morgenrote's pattern):
- Use `set_variable` and `change_variable` for country-scoped variables (not `set_country_variable`/`change_country_variable`)
- Use `check_variable = { variable_name >= value }` in effects (not in event triggers)
- Use conditional random_events in on_actions for variable-based event triggering
- Use `var:variable_name` in effects and scripted effects
- Variables must be initialized at game start and have failsafe checks
- Use `has_variable = variable_name` to check if variables exist

## Testing
- No linter errors found after fixes
- All variable references now use consistent country variable syntax
- Mod should now load properly and initialize variables correctly

## Files Modified
- `actionables_lpr.mod` (created)
- `common/history/global/00_global.txt`
- `common/on_actions/lpr_on_actions.txt` (major changes - conditional random_events)
- `common/scripted_effects/lpr_finisher_effects.txt`
- `events/actionables_lpr_builder_events.txt`
- `events/actionables_lpr_binary_events.txt` (removed triggers)
- `events/actionables_lpr_big_finisher_binary_events.txt` (removed triggers)
- `events/actionables_lpr_finisher_events.txt`
- `events/actionables_lpr_politics_events.txt` (removed triggers)
- `events/actionables_lpr_climate_events.txt`
- `events/actionables_lpr_spent_threshold_events.txt` (removed triggers)
