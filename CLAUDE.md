# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Project Overview

Pebble watchface for displaying blood glucose data from xDrip. Shows BG readings, trend graph, delta, time ago, IoB (insulin on board from Minimed pump companion app), and time/date.

**Target Hardware**: Pebble 2 Duo - 144x168 px black & white display (similar to Pebble 2). Use `PBL_BW` conditional compilation paths.

## Building

```bash
pebble build  # Always run after making changes to verify they build
```

## Code Navigation

The entire watchface is in `src/xdrip.c` (2694 lines). Since this file is too large to load all at once, use these references to find specific sections:

### File Structure Overview

**Preprocessor & Constants** (lines 1-250):
- Lines 6-10: Debug flags (`DEBUG_LEVEL`, `TEST_MODE`)
- Lines 21-50: UI layer declarations
- Lines 150-220: Configuration constants (BG ranges, alert settings, timeouts)

**Data Loading Functions** (lines 954-1550):
- Line 954: `load_icon()` - Update trend arrow/icon
- Line 1057: `load_bg()` - Update BG value display
- Line 1251: `load_cgmtime()` - Update time ago display
- Line 1326: `load_bg_delta()` - Update delta value
- Line 1420: `load_battlevel()` - Update battery levels

**Message & Update Handlers** (lines 1550-1900):
- Line 1563: `inbox_received_handler_cgm()` - Handle data from xDrip app
- Line 1817: `timer_callback_cgm()` - Check for stale data, trigger alerts
- Line 1854: `handle_second_tick_cgm()` - Update time/date display

**UI Setup & Teardown** (lines 1997-2450):
- Line 1997: `window_load_cgm()` - Create all UI layers (layer positioning here)
- Line 2312: `window_unload_cgm()` - Cleanup layers
- Line 2346: `init_cgm()` - Initialize app, subscribe to services
- Line 2428: `deinit_cgm()` - Shutdown app

### Platform-Specific Compilation

Use these flags to target the Pebble 2 Duo (B&W display):
- `PBL_BW` - Black & white displays (aplite, diorite, emery, flint)
- `PBL_PLATFORM_APLITE` - Original B&W Pebble
- `PBL_COLOR` - Color displays (basalt, chalk)
- `PBL_ROUND` - Round displays (Pebble Time Round)

### Key Data Sources

**AppMessage keys** (defined in appinfo.json):
- AppKey 0: `icon` - Trend arrow code
- AppKey 1: `bg` - Blood glucose value
- AppKey 2: `tcgm` - CGM timestamp
- AppKey 4: `dlta` - Delta value
- AppKey 5: `ubat` - Phone battery
- AppKey 7-9: `t_beg`, `t_dat`, `t_end` - Trend graph PNG chunks
- AppKey 10: `iob` - Insulin on Board (from Minimed companion app)

Message handling in `inbox_received_handler_cgm()` at line 1563.

### Common Tasks

**Adjust layout**: Edit `window_load_cgm()` around line 1997. Layers use `GRect(x, y, width, height)`.

**Change data displayed**: Add new AppKeys in appinfo.json, handle in `inbox_received_handler_cgm()` at line 1563.

**Test mode**: Toggle `TEST_MODE` in src/xdrip.c line 8, or use `#include "test_mode.h"` for test data.

**Debug mode**: Set `DEBUG_LEVEL 1` in src/xdrip.c line 6 for verbose logging (must be 0 for release).
