# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Pebble watchface for displaying blood glucose data from xDrip. It's a fork of cgm-pebble-offline modified specifically for use with xDrip (not Nightscout). The watchface displays:
- Blood glucose readings with trend arrows
- Delta (change) values
- Time since last reading
- Battery levels (both phone and watch)
- Bluetooth connection status
- Optional health data logging (heart rate and movement)

## Build System

The project uses the Pebble SDK 3.0 build system with waf:

```bash
# Build the watchface for all platforms
pebble build

# Clean build artifacts
pebble clean

# Install to connected watch
pebble install

# View logs from watch
pebble logs
```

Target platforms are: aplite, basalt, chalk, diorite, and flint (configured in appinfo.json:11-16).

## Architecture

### Single-File Application

The entire watchface logic is in `src/xdrip.c` (2576 lines). This is typical for simple Pebble watchfaces where all UI, messaging, and state management are tightly coupled.

### Platform-Specific Compilation

The code uses conditional compilation extensively:
- `PBL_PLATFORM_APLITE` - Black & white Pebble (original)
- `PBL_PLATFORM_BASALT` - Color Pebble
- `PBL_ROUND` - Round displays (Pebble Time Round)
- `PBL_COLOR` - Any color display
- `PBL_BW` - Black & white displays (aplite, diorite)
- `PBL_HEALTH` - Health API support

### Key Components

**UI Layers** (src/xdrip.c:21-36):
- `bg_layer` - Blood glucose value display
- `icon_layer` - Trend arrows or special value icons
- `bg_trend_layer` - Graph of BG trend over time
- `delta_layer` - Change in BG value
- `message_layer` - Status messages
- `time_watch_layer` - Current time
- `upper_face_layer`, `lower_face_layer` - Background panels

**AppMessage Communication** (src/xdrip.c:48):
- Uses AppSync to receive data from xDrip phone app
- Message keys defined in appinfo.json:19-33
- Handlers: `inbox_received_handler_cgm`, `sync_error_callback_cgm`, `inbox_dropped_handler_cgm`, `outbox_failed_handler_cgm`

**Health Data Logging** (src/xdrip.c:51-63):
- Optional feature (requires `PBL_HEALTH` capability)
- Logs heart rate and step count data
- Uses DataLoggingSessionRef for persistent storage

**Trend Graph** (src/xdrip.c:80-87):
- Receives PNG image data in chunks from phone
- Dynamically allocated buffer (`trend_buffer`)
- Platform-specific rendering (custom bitmap layer update for BW displays)

### Alert System

The watchface has an extensive alert/vibration system (src/xdrip.c:123-134):
- Special value alerts (configurable BG threshold)
- DoubleDown alerts (rapid BG decrease)
- Bluetooth disconnection alerts
- Phone/CGM offline alerts
- Low battery alerts
- AppMessage error alerts

Alert behavior is controlled by constants (src/xdrip.c:150-200) including vibration levels, snooze times, and timeout periods.

### Configuration Constants

User-configurable constants are at src/xdrip.c:150-220:
- BG ranges for MGDL and MMOL
- Alert snooze times
- Vibration levels (0=none, 1=low, 2=medium, 3=high)
- Timeout periods for offline detection
- Message display toggles

Comments warn against changing these without understanding the implications.

## Development Notes

### Debug Mode

Debug features controlled by preprocessor defines (src/xdrip.c:6-10):
- `DEBUG_LEVEL` - Set to 1 for verbose logging (must be 0 for release)
- `TEST_MODE` - Enables test data display
- IDE debug: `//#define PBL_HEALTH` for testing health features

### Resources

Images are defined in appinfo.json:38-131 and stored in `resources/images/`:
- Trend arrows (up, down, flat variants)
- Status icons (bluetooth, antenna, battery, etc.)
- Platform-specific variants with `~color` suffix for color displays
- Old images kept in `resources/images/old/`

### Build Output

Build creates platform-specific binaries in `build/`:
- `build/<platform>/pebble-app.elf` - Compiled binary for each platform
- `build/appinfo.json`, `build/js/message_keys.json` - Generated configs
- Auto-generated files: `message_keys.auto.c`, `resource_ids.auto.c`, `appinfo.auto.c`

## Code Organization

Main functions in execution order:
1. `init_cgm()` - Initialize app, subscribe to services (src/xdrip.c:2346)
2. `window_load_cgm()` - Create all UI layers (src/xdrip.c:1997)
3. `inbox_received_handler_cgm()` - Handle data from phone (src/xdrip.c:1563)
4. `handle_second_tick_cgm()` - Update time display (src/xdrip.c:1854)
5. `timer_callback_cgm()` - Check for stale data, trigger alerts (src/xdrip.c:1817)
6. `window_unload_cgm()` - Cleanup layers (src/xdrip.c:2312)
7. `deinit_cgm()` - Shutdown app (src/xdrip.c:2428)

Data loading functions called from message handler:
- `load_icon()` - Update trend arrow/icon (src/xdrip.c:954)
- `load_bg()` - Update BG value display (src/xdrip.c:1057)
- `load_cgmtime()` - Update time ago display (src/xdrip.c:1251)
- `load_bg_delta()` - Update delta value (src/xdrip.c:1326)
- `load_battlevel()` - Update battery levels (src/xdrip.c:1420)

## Common Tasks

Building for specific platform:
```bash
pebble build --platform basalt
```

Analyzing binary size:
```bash
pebble analyze-size
```

Installing and viewing logs:
```bash
pebble install --logs
```

## Customization Guide

The user is customizing this watchface for personal use with the following goals:
- White background throughout
- Clock information: Time, Date, Week number, Battery indicators (more intuitive than "B:" and "W:")
- Glucose information: Current level, 3-hour graph, optional trend arrow, bolus dots in graph, insulin on board
- Make stale glucose data very obvious (grey out when old)

### Layout Customization

**Display dimensions**: Pebble watches are 144x168 pixels (rectangular) or 180x180 pixels (round, PBL_ROUND).

**UI layers are positioned using GRect(x, y, width, height)** in `window_load_cgm()` at src/xdrip.c:1997-2310:

Current layout (for rectangular color displays):
- `upper_face_layer`: GRect(0, 0, 144, 83) - White background, upper half
- `lower_face_layer`: GRect(0, 84, 144, 165) - Blue/Black background, lower half
- `bg_layer`: GRect(0, -5, 95, 42) - Blood glucose value (FONT_KEY_BITHAM_42_BOLD)
- `icon_layer`: GRect(85, -9, 78, 49) - Trend arrows
- `bg_trend_layer`: GRect(0, 0, 144, 84) - Graph PNG from xDrip
- `delta_layer`: GRect(0, 36, 143, 50) - BG delta value (FONT_KEY_GOTHIC_28)
- `cgmtime_layer`: GRect(52, 58, 40, 24) - Time ago display (FONT_KEY_GOTHIC_24_BOLD)
- `time_watch_layer`: GRect(0, 82, 143, 44) - Current time (FONT_KEY_BITHAM_42_BOLD)
- `date_app_layer`: GRect(0, 124, 143, 26) - Date display (FONT_KEY_GOTHIC_24_BOLD)
- `battlevel_layer`: GRect(0, 150, 143, 20) - Phone battery (FONT_KEY_GOTHIC_18_BOLD)
- `watch_battlevel_layer`: GRect(0, 150, 143, 20) - Watch battery (FONT_KEY_GOTHIC_18_BOLD)

**Fonts available** (system fonts, no need to load):
- FONT_KEY_BITHAM_42_BOLD, FONT_KEY_BITHAM_30_BLACK
- FONT_KEY_GOTHIC_28_BOLD, FONT_KEY_GOTHIC_24_BOLD, FONT_KEY_GOTHIC_18_BOLD, FONT_KEY_GOTHIC_14
- See full list: https://developer.rebble.io/developer.pebble.com/guides/app-resources/system-fonts/index.html

**Colors** (use GColorXXX constants):
- Current scheme: upper_face_layer is GColorWhite, lower_face_layer is GColorDukeBlue (color) or GColorBlack (BW)
- Common colors: GColorWhite, GColorBlack, GColorClear, GColorRed, GColorGreen, GColorYellow, GColorLightGray
- Full color palette: https://developer.rebble.io/developer.pebble.com/guides/tools-and-resources/color-picker/index.html

### Data Available from xDrip

**Currently received** (see appinfo.json:19-33 and `inbox_received_handler_cgm()` at src/xdrip.c:1563):
- `icon` (AppKey 0) - Trend arrow icon code
- `bg` (AppKey 1) - Blood glucose value as string
- `tcgm` (AppKey 2) - CGM timestamp (Unix time)
- `tapp` (AppKey 3) - App timestamp (Unix time)
- `dlta` (AppKey 4) - BG delta value as string
- `ubat` (AppKey 5) - Phone battery level
- `name` (AppKey 6) - Name string
- `t_beg`, `t_dat`, `t_end` (AppKeys 7-9) - Trend graph PNG data chunks
- `sync`, `platform`, `version` (AppKeys 1000-1002) - Sync/version info

**To add bolus dots or insulin on board**: These would need to be added to the xDrip app first. You would need to:
1. Define new AppKeys in appinfo.json (e.g., `"bolus": 10, "iob": 11`)
2. Handle them in `inbox_received_handler_cgm()` at src/xdrip.c:1563
3. Create new UI layers or draw on the graph to display them

### Stale Data Detection

Glucose data staleness is calculated in `load_cgmtime()` at src/xdrip.c:1251:
- `current_cgm_time` - Timestamp of last glucose reading (set from AppKey `tcgm`)
- `current_cgm_timeago` - Calculated age in seconds
- Currently displays as text ("5m", "12m", "2h", etc.) in `cgmtime_layer`

To grey out stale data:
- Check `current_cgm_timeago` value in your update functions
- Use `text_layer_set_text_color()` to change color based on age threshold
- Example: if `current_cgm_timeago > (5 * MINUTEAGO)`, set to GColorLightGray
- Apply to `bg_layer`, `delta_layer`, and `icon_layer` for visual indication

### Week Number Display

Currently NOT implemented. To add:
1. Use `strftime(buffer, size, "%V", tick_time)` to get ISO week number
2. Create a new TextLayer in `window_load_cgm()` at src/xdrip.c:1997
3. Update it in `handle_second_tick_cgm()` at src/xdrip.c:1854 alongside date updates
4. Example format: "Week 48" or "W48"

### Battery Display Improvements

Current implementation at src/xdrip.c:2202-2268:
- `battlevel_layer` shows phone battery with "B:" prefix
- `watch_battlevel_layer` shows watch battery with "W:" prefix
- Both layers share the same GRect, alternating visibility

To improve:
- Use battery icons instead of text (would need to create bitmap resources)
- Use different colors (already implemented: Green >60%, Yellow 30-60%, Red <30%)
- Position separately instead of overlaying same rectangle
- Remove "B:" and "W:" prefixes, rely on position/icons to distinguish
