# Configuration Guide

Configuration reference for Dreame Vacuum Multi-Floor Control blueprint.

**Prerequisites:**
- Dreame Vacuum Integration ≥ v2.0.0b21
- At least one saved map

**See also:** [Features & Installation](README.md)

---

## 🤖 Robot Configuration

![Robot Configuration](images/robot-configuration.png)

### 🤖 Vacuum Entity

Select your vacuum entity. All related entities (status, mode, map, camera) are auto-detected from this selection.

### ⏸️ Pause Delay After Start
**Default:** 4.5s

Time robot moves away from dock before pausing for transport.
- Only applies to non-base-station maps
- 0s: Robot stays docked (no start)

---

## 🏠 Map 1 / Map 2 / Map 3

![Map Configuration](images/map-configuration.png)

**Note:** Map names are auto-detected from the integration.

### 🔄 Cleaning Repeats
**Default:** 2

Number of cleaning passes per room. Overrides global fallback setting.

### 🔀 Map Switch Trigger

Switches to this map when trigger fires.

**Trigger ID requirement:**
- State/Event triggers: Must set Trigger ID to `fn_map1` (or `fn_map2`, `fn_map3`)
- Device/MQTT triggers: Auto-detected (no ID required)

Leave empty for schedule-only maps.

### 📅 Sweep-Only Schedule
**Default:** "none"

Optional schedule entity for automatic sweep-only cleaning.

### 📅 Sweep+Mop Schedule
**Default:** "none"

Optional schedule entity for automatic sweep+mop cleaning.

---

## 🗺️ Map Functions

![Map Functions](images/map-functions.png)

### 🗺️ Auto-Switch-Back to Base Map
**Default:** Enabled

Automatically returns to base station map after completing room cleaning. Includes 20-second safety buffer after schedule preparation.

### 🔍 Task Status Sensor
**Auto-detected:** `sensor.*_task_status`

Required for auto-switch-back feature.

### 🗺️ Auto-Discard Temporary Maps
**Default:** Enabled

Automatically deletes temporary maps before map switching. Prevents map selection blocking.

---

## 🎮 Control Functions

![Control Functions](images/control-functions.png)

### ▶️ Smart Start/Pause/Resume Trigger

Intelligent control based on robot status:
- Idle → Start cleaning
- Cleaning → Pause
- Paused → Resume

**Trigger ID:** `fn_start` (required for State/Event triggers)

### 🧹 Sweep Only Mode Trigger

Switches vacuum to sweep-only mode.

**Trigger ID:** `fn_sweep` (required for State/Event triggers)

### 💧 Sweep + Mop Mode Trigger

Switches vacuum to sweep+mop mode.

**Trigger ID:** `fn_mop` (required for State/Event triggers)

---

## ⚙️ Cleaning Settings

![Cleaning Settings](images/cleaning-settings.png)

### 🔄 Global Cleaning Repeats (Fallback)
**Default:** 2

Default cleaning passes used when per-map repeats are not configured.

### 🧩 Use Segment Service
**Default:** Enabled

Enables room-based cleaning via `dreame_vacuum.vacuum_clean_segment` service. Falls back to `vacuum.start` when disabled or segments unavailable.

### 📋 Use Cleaning Sequence
**Default:** Enabled

Cleans rooms in order defined in Dreame app (uses `order` attribute from map camera).

**Requirements:**
- Firmware ≥1156
- `switch.*_cleaning_sequence` available

### ⚙️ Use Customised Cleaning
**Default:** Enabled

Uses per-room settings from Dreame app (suction level, water volume, mop humidity, repeats per room).

**Important:** Only applied for Sweep+Mop modes. Sweep-only modes use sequence but NOT customised settings.

**Requirements:**
- Firmware ≥1156
- `switch.*_customized_cleaning` available
- Room settings configured in Dreame app

### Service Integration

| Segment Service | Sequence | Customised | Behavior |
|----------------|----------|------------|----------|
| ✓ | ✓ | ✓ | Rooms in order, custom settings (mop mode only) |
| ✓ | ✓ | ✗ | Rooms in order, blueprint repeats |
| ✓ | ✗ | ✗ | All rooms, blueprint repeats |
| ✗ | - | - | Full map clean (`vacuum.start`) |

---

## 🔔 Notification Settings

![Notification Settings](images/notification-settings.png)

### 🔔 Enable Notifications
**Default:** Off

Sends notifications for scheduled cleaning and pickup events. Includes action buttons (Prepare, Skip, Start, Cancel).

### 👤 Persons to Notify

Person entities to notify. Only sends to persons currently at home (presence-based filtering). Notification service auto-detected from device tracker.

### 👥 Notification Groups

Notification group names without `notify.` prefix. Always sends to groups (no presence check).

**Example:** `family_phones` → calls `notify.family_phones`

### 📅 Scheduled Notification Title
**Default:** "Scheduled Cleaning Ready"

Notification title for scheduled cleaning events. Supports template variables (see table below).

### 📅 Scheduled Notification Message
**Default:** "{{ robot_name }} is ready for scheduled cleaning on {{ map_name }} ({{ cleaning_mode_display }}). Please prepare the robot."

Notification message for scheduled cleaning events. Supports template variables (see table below).

### 📅 Schedule Repeat Count
**Default:** 2

Number of reminder notifications if no response.

### 📅 Schedule Repeat Interval
**Default:** 15 minutes

Time between reminder notifications. Set to 0 for no repeats.

### 🤖 Pickup Notification Title
**Default:** "Robot Ready for Transport"

Notification title for pickup events (robot paused and ready for transport). Supports template variables (see table below).

### 🤖 Pickup Notification Message
**Default:** "{{ robot_name }} is paused and ready for transport to {{ map_name }}. Please pick up the robot."

Notification message for pickup events. Supports template variables (see table below).

### 🤖 Pickup Repeat Count
**Default:** 2

Number of reminder notifications if robot not started.

### 🤖 Pickup Repeat Interval
**Default:** 10 minutes

Time between reminder notifications. Set to 0 for no repeats.

### 📱 iOS Interruption Level
**Default:** time-sensitive

Notification priority level for iOS devices.

**Options:**
- `passive`: Silent, background only
- `active`: Standard, shows on lock screen
- `time-sensitive`: Bypasses Focus modes
- `critical`: Always notifies, bypasses Do Not Disturb

### 🔊 iOS Sound
**Default:** "default"

Sound for iOS notifications. Options: `default`, `none`, or custom sound name.

### 🚨 iOS Critical Alert
**Default:** Off

Bypasses Do Not Disturb and mute switch. Requires iOS permission.

### 🔈 iOS Critical Volume
**Default:** 1.0

Volume level for critical alerts. Only applies when iOS Critical Alert is enabled.

### Notification Template Variables

| Variable | Description |
|----------|-------------|
| `robot_name` | Vacuum friendly name |
| `map_name` | Current/target map name |
| `cleaning_mode_display` | Localised mode text (e.g., "Sweep+Mop") |
| `current_time` | Notification time (HH:MM format) |
| `repeat_number` | Current repeat iteration count |

---

## 🌐 Localization

![Localization](images/localization.png)

Customise display texts for notifications and buttons in your preferred language.

### 🧹 Sweep Mode Display Name
**Default:** "Sweep"

Display name for sweep-only mode in notifications.

### 💧 Mop Mode Display Name
**Default:** "Mop"

Display name for mop-only mode in notifications.

### 🧹💧 Sweep + Mop Mode Display Name
**Default:** "Sweep + Mop"

Display name for combined sweep+mop mode in notifications.

### ✋ Prepare Button Label
**Default:** "Prepare Robot"

Button text for robot preparation action.

### ⏭️ Skip Button Label
**Default:** "Skip Cleaning"

Button text for skipping scheduled cleaning.

### ▶️ Start Cleaning Button Label
**Default:** "Start Cleaning"

Button text for starting cleaning after pickup.

### ❌ Cancel Button Label
**Default:** "Cancel Cleaning"

Button text for cancelling cleaning workflow.

### 🧹 Sweep Only Button Label
**Default:** "Start Sweep Only"

Button text for sweep-only fallback when mop not ready.

### ⚠️ Mop Not Ready - Title
**Default:** "⚠️ Mop Not Ready"

Notification title when mop pads missing or water tank empty.

### 💬 Mop Not Ready - Message
**Default:** "Mop pads not installed or water tank empty. Start sweep-only instead?"

Notification message when mop pads missing or water tank empty.

---

## 🔧 Advanced Settings

![Advanced Settings](images/advanced-settings.png)

### Timeouts

#### ⏱️ Moistening Timeout
**Default:** 215s

Maximum wait time for mop washing cycle completion at base station.

#### ⏱️ Sweep Start Timeout
**Default:** 30s

Maximum wait time for sweep-only cleaning to start.

#### ⏱️ Mop Start Timeout
**Default:** 120s

Maximum wait time for mop operations (washing to start, robot to start after washing).

### Mode Values

#### 🧹 Mode Value: Sweep Only
**Default:** "sweeping"

Exact value for sweep-only mode (case-sensitive). Must match `select.*_cleaning_mode` entity options.

#### 🧹💧 Mode Value: Sweep + Mop
**Default:** "sweeping_and_mopping"

Exact value for sweep+mop mode (case-sensitive). Must match `select.*_cleaning_mode` entity options.

### Status Detection

#### 🔍 Active Cleaning States
**Type:** Text  
**Default:** `cleaning,returning,zone_cleaning,room_cleaning,sweeping,mopping,sweeping_and_mopping`

Comma-separated list of status values indicating active cleaning. Used to detect if robot is currently working.

#### 🔍 Moistening Status
**Type:** Text  
**Default:** (empty)

Optional status value during mop moistening phase. Leave empty if unsure or not applicable.

**Examples:** `moistening`, `wetting_mop`

#### 🔍 Paused States
**Type:** Text  
**Default:** `paused,sleeping,standby`

Comma-separated list of status values when robot is paused. Used for resume logic.

#### �� Error States
**Type:** Text  
**Default:** `error`

Comma-separated list of status values indicating error conditions. Triggers persistent notification with error details.

#### 🔍 Idle States
**Type:** Text  
**Default:** `idle,sleeping,standby`

Comma-separated list of status values when robot is idle and ready for new tasks.

#### 🔍 Base Station Warning States
**Type:** Text  
**Default:** `paused,clean_add_water`

Comma-separated list of base station status values requiring attention (tank empty, washing paused, etc.). Triggers persistent notification.

### Debug

#### 🐛 Debug Level
**Default:** 0 (Off)

Controls debug notification verbosity.

**Options:**
- `0` (Off): No debug notifications
- `1` (Info): Summary after completion only
- `2` (Debug): All steps + summary

Error and warning notifications always appear regardless of debug level.

---

## Troubleshooting

### Automation Doesn't Trigger

**Symptom:** Nothing happens when trigger fires.

**Solutions:**
1. Check trigger configuration:
   - State/Event triggers: Verify Trigger ID set correctly (`fn_start`, `fn_map1`, `fn_map2`, `fn_map3`, `fn_sweep`, `fn_mop`)
   - Device triggers: Verify device is selected
2. Enable Debug Level 2 → Check if trigger is detected
3. Verify automation mode is `queued` (blueprint default)

---

### Cleaning Mode Not Changing

**Symptom:** Mode selection function doesn't change robot cleaning mode.

**Cause:** Mop pad not installed → `cleaning_mode` entity becomes unavailable.

**Solutions:**
1. Attach mop pad to robot
2. Check Developer Tools → States → `select.*_cleaning_mode` → Should not show "unavailable"
3. Automation continues with current/default mode (does not abort)

---

### Robot Doesn't Start/Pause

**Symptom:** Service call fails or robot doesn't respond.

**Solutions:**
1. Verify entity IDs: Developer Tools → States → Search for `vacuum.*`
2. Check robot status sensor matches configured status values
3. Increase timeout values: Advanced Settings → Sweep/Mop Start Timeout
4. Enable Debug Level 2 → Monitor state transitions

---

### Segments Not Found

**Symptom:** Fallback to `vacuum.start` instead of segment-based cleaning.

**Solutions:**
1. Verify map camera entity has `segments` attribute: `camera.*_map`
2. Check map is loaded and robot is online
3. Disable segment service: Cleaning Settings → Use Segment Service toggle off
4. Verify firmware supports segments (recent version required)

---

### Notifications Not Received

**Symptom:** No notifications appear on mobile device.

**Solutions:**
1. Enable notifications: Notification Settings → Enable Notifications toggle on
2. Check recipients:
   - Persons: Verify person entities exist and have device trackers
   - Groups: Verify notify service exists (e.g., `notify.group_name`)
3. Check presence: Person entities must be home (groups always send)
4. iOS: Verify app permission for notification type (especially Critical alerts)

---

### Schedule Conflicts

**Symptom:** Schedule doesn't start, no notification appears.

**Cause:** Another schedule already running (robot busy).

**Expected behavior:** Silent abort to prevent conflicts.

**Solutions:**
1. Stagger schedules: Ensure sufficient time between map schedules
2. Check robot status should be idle/docked when schedule triggers
3. Enable Debug Level 2 → Check for "Schedule conflict detected" message

---

### Auto-Switch-Back Not Working

**Symptom:** Robot doesn't return to base map after cleaning other floor.

**Solutions:**
1. Verify feature enabled: Map Functions → Auto-Switch-Back to Base Map toggle
2. Check task status sensor is selected and available
3. Verify robot is on different map (feature skips if already on base map)
4. Check activity: Only triggers after room cleaning completes (not washing/drying)
5. Enable Debug Level 2 → Monitor task status transitions

---

### Timeout Errors

**Symptom:** Persistent notification "Robot did not start" or "Timeout reached".

**Solutions:**
1. Increase timeout values: Advanced Settings → Moistening/Sweep/Mop Start Timeout
2. Verify status values match your robot: Advanced Settings → Check status strings
3. Monitor actual timing: Enable Debug Level 2 → Check measured times in notifications
4. Verify robot responds: Check Home Assistant logs for service call errors

---

**See also:** [Features & Installation](README.md)
