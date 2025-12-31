# 🤖 Dreame Vacuum – Multi-Floor Control

[![Version](https://img.shields.io/badge/version-0.9.14-blue.svg)](https://github.com/errormastern/dreame-multifloor-control/releases)
[![Home Assistant](https://img.shields.io/badge/Home%20Assistant-2024.10%2B-green.svg)](https://www.home-assistant.io/)
[![License](https://img.shields.io/badge/license-MIT-orange.svg)](LICENSE)

A Home Assistant blueprint for controlling Dreame vacuums across multiple floors. Supports scheduled cleaning with automatic preparation and transport notifications for non-base-station maps and manual control via buttons or other triggers. **Never care about the right robot settings for seamless multi-floor cleaning again.**


## ✨ Features

🤖 Auto-detection of vacuum entities (select vacuum, rest detected automatically)<br>
⚙️ Automatically verify and configure optimal robot settings prior to each cleaning cycle<br>
📅 Per-map schedules with sweep/mop modes (3 maps, 6 schedules)<br>
🔔 Notification workflow with action buttons for transport<br>
👥 Multi-recipient notifications with presence checking<br>
🏠 Segment-based cleaning with configurable repeats<br>
✨ Optional customised cleaning using room settings from Dreame app<br>
⚠️ Safety checks: schedule conflicts, robot and cleaning options, dock status, mop readiness<br>
🗺️ Auto-discard temporary maps for seamless multi-floor operation<br>
🏠 **Auto-switch to base map** after multi-floor cleaning **(v0.9.14+)**<br>
🐛 Debug mode with timing measurements


## 📋 Requirements

- Home Assistant ≥ 2024.10.0
- [Dreame Vacuum Integration](https://github.com/Tasshack/dreame-vacuum) ≥ v2.0.0b19
- At least one saved map configured
- Optional: Schedule helpers for time-based automation
- Optional: Mobile app for notifications


## 💾 Installation

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://github.com/errormastern/dreame-multifloor-control/raw/main/vacuum_control.yaml)

Or manually: **Settings** → **Automations & Scenes** → **Blueprints** → **Import Blueprint**


## 🚀 Quick Start

1. Create automation from blueprint
2. Select your vacuum entity
3. Configure triggers for the functions you need
4. Save and test

Related entities (status, mode, map, camera) are auto-detected.


## 🔄 Workflows

### 🚿 Preparation Workflow

For maps without a base station, the robot prepares at the dock and pauses for transport.

**Sweep + Mop mode:**
1. Robot washes mop at base station
2. Robot starts and undocks
3. After a short delay, robot pauses automatically
4. Transport robot to target floor
5. Press button or notification action to resume

**Sweep only mode:**
Steps 1 is skipped – robot starts directly and pauses for pickup.

> [!NOTE]
> The delay before pausing allows the robot to move away from charging contacts for easier pickup. See [Timeouts](#-timeouts) for configuration.


### 💧 Mop Readiness Check

Before starting **Sweep + Mop** mode, the blueprint verifies:
- Mop pads are installed (`sensor.*_mop_pad` = "installed")
- Water tank is not empty (`sensor.*_low_water_warning` = "no_warning")
- Selfclean option is disabled on non-base-station-maps.

**If not ready:**
- Scheduled cleaning sends a warning notification with "Start Sweep Only" option
- Manual start shows a persistent notification and aborts

This prevents failed cleaning attempts when hardware isn't ready.


### 📅 Scheduled Cleaning

The core workflow for time-based cleaning across floors.

**Base station maps:** Schedule triggers → Robot cleans immediately.

**Maps without basestation:**
1. Schedule triggers → Notification with "Prepare Robot" / "Skip"
2. Confirm → Robot prepares and pauses (see [Preparation Workflow](#-preparation-workflow))
3. Pickup notification → Transport robot
4. Press "Start Cleaning" → Robot resumes

**Conflict handling:** Only one schedule runs at a time. New schedules abort silently if robot is busy.

Schedule helpers let you define when cleaning should start. Create them via **Settings** → **Helpers** → **Schedule**. See [Schedule helper documentation](https://www.home-assistant.io/integrations/schedule/).


### 🔘 Manual Control

Direct control via buttons, switches, or other triggers.

| Function | Description |
|----------|-------------|
| **Sweep Only Mode** | Sets cleaning mode to sweep-only for quick cleaning without mopping. |
| **Sweep + Mop Mode** | Sets cleaning mode with mop enabled for full cleaning. |
| **Smart Start/Pause/Resume** | Context-aware control – adapts to robot status. See details below. |
| **Map 1 / Map 2 / Map 3** | Switches between floor maps for multi-floor setups. |


### ⚡ Trigger Setup

Use any Home Assistant trigger type. MQTT and device triggers auto-detect action values from the payload.

> [!WARNING]
> For state or event triggers, you must set a **Trigger ID** manually (e.g., `fn_start`, `fn_sweep_mode`). 
Without an ID, the automation cannot determine which function to execute.


## ⏱️ Timeouts & Move out delay

The blueprint employs timeouts as a safety fallback mechanism to ensure the preparation process terminates even when automatic detection of individual step completion fails.

| Timeout | Default | Purpose |
|---------|---------|---------|
| **Start Timeout** | 120s | Max wait for robot to move out of the basestation after start/preparation |
| **Moistening Timeout** | 60s | Max wait for mop washing to complete (sweep+mop mode) |
| **Move Out Delay** | 4.5s | Delay before pausing after undock – allows robot to move a bit away from charging contacts |

Adjust these in the blueprint configuration if your robot behaves differently. Check Debug Messages for measured timings.

### 🏠 Auto-Switch to Base Station Map (v0.9.14+)

Automatically switches to the base station map after completing a multi-floor cleaning task.

**Configuration:**
1. Enable/disable via Advanced Settings (default: enabled)
2. Select task status sensor: `sensor.{vacuum_name}_task_status`

**Smart Behavior:**
- Only triggers after room/floor cleaning (not washing/drying)
- Skips if already on base station map
- 20-second safety buffer after schedule preparation
- Won't interfere with active schedule workflows

**Example:** Robot cleans "Obergeschoss" (upper floor) → returns to dock → auto-switches back to "Wohnzimmer" (base station map)

## 🌐 Localisation

Customise notification variables in the Localisation section:
- Mode labels (Sweep/Mop)
- Button labels (Prepare, Skip, Start, Cancel, Sweep Only)
- Warning messages (Mop Not Ready)

## 📝 Technical Notes

**Automation mode:** `queued` (max: 10) – required for button devices sending press + release events.

**Tested with:** Xiaomi X10+ (Dreame L10s Ultra and other supported models should work).


## 🔧 Troubleshooting

### Cleaning Mode Not Changing

**Symptom:** Mode selection functions (Sweep/Mop) don't change robot cleaning mode.

**Cause:** Mop pad not installed → `cleaning_mode` entity becomes unavailable.

**Explanation:**
- Dreame integration disables `cleaning_mode` when only 1 option available
- Robot will use current/default cleaning mode
- Automation continues successfully (won't abort)

**Solutions:**
1. Attach mop pad to robot
2. Use debug mode to verify entity status

**Debug Mode Output:**
```
⚙️ Mode: Sweep+Mop
Status: ⚠️ Unavailable
Entity: select.robot_name_cleaning_mode
Reason: Mop pad not installed or entity disabled
Time: 14:30:15
```


## 🔗 Links

- [Dreame Vacuum Integration](https://github.com/Tasshack/dreame-vacuum) – Required integration
- [Repository](https://github.com/errormastern/dreame-multifloor-control) – Source and releases
- [Issues](https://github.com/errormastern/dreame-multifloor-control/issues) – Bug reports and feature requests

---

**Licence:** MIT
