# Flux Open Home Irrigation Controller

An ESP32-S3 based smart irrigation controller with up to 32 zones, designed for ESPHome and Home Assistant integration.

**Made for ESPHome** compliant - works with ESPHome Dashboard adoption and OTA updates.

---

## Features

- **Up to 32 Zones**: 8 built-in zones + 24 expansion zones via MCP23017 I2C boards
- **Seamless Zone Transitions**: Valve overlap prevents pump cycling between zones
- **Intelligent Pump/Master Valve Control**: Automatic 2-second startup delay, stays on throughout cycle
- **Flexible Zone 8 Configuration**: Use as normal zone, master valve, or pump start relay
- **Built-in Scheduling**: 4 programmable start times, 7-day scheduling
- **Auto Advance**: Automatically cycles through enabled zones (resets after each cycle; scheduled runs handle this automatically)
- **Repeat Cycles**: Run multiple passes through all zones
- **Wi-Fi Provisioning**: Bluetooth, captive portal, or serial configuration
- **OTA Firmware Updates**: Update directly from Home Assistant
- **Expansion Board Auto-Detection**: Automatically detects connected I2C expansion boards
- **Rain Sensor Support**: Optional wired rain sensor with configurable NO/NC type, automatic irrigation shutoff, and rain delay timer (1-72 hours, default 48)
- **Status LED**: RGB LED indicates Wi-Fi connection, rain detection, and system state

---

## Hardware Requirements

### Main Controller
- ESP32-S3 DevKitC-1 or compatible
- 8-channel relay module (directly connected to GPIO)
- RGB LED for status indication (WS2812 on GPIO42)

### GPIO Pin Assignments (Main Board)
- Zone 1: GPIO45
- Zone 2: GPIO35
- Zone 3: GPIO36
- Zone 4: GPIO37
- Zone 5: GPIO38
- Zone 6: GPIO39
- Zone 7: GPIO40
- Zone 8: GPIO41
- Rain Sensor: GPIO4 (configurable via substitution)

### Expansion Boards (Optional)
- MCP23017 I2C I/O Expanders
- Board 1 (Zones 9-16): Address 0x20
- Board 2 (Zones 17-24): Address 0x21
- Board 3 (Zones 25-32): Address 0x22
- I2C: SDA on GPIO8, SCL on GPIO9
- Boards are sequential: Board 2 requires Board 1, Board 3 requires Board 1 and 2

---

## Installation

### Option 1: ESPHome Dashboard Adoption
1. Add the device to your ESPHome Dashboard
2. The configuration will be imported automatically from this repository
3. Configure Wi-Fi and install

### Option 2: Manual Installation
1. Clone this repository
2. Copy `IrrigationControllerMain.yaml` to your ESPHome config directory
3. Customize substitutions as needed
4. Compile and flash via USB or OTA

### Option 3: Web Installer
Visit the project's web installer page to flash directly from your browser (Chrome/Edge).

### Device Provisioning (Wi-Fi Setup)

After flashing, the controller needs to be connected to your Wi-Fi network. Three methods are available:

1. **Bluetooth (Recommended)**: The controller advertises via Bluetooth on first boot. Open the Home Assistant Companion app, go to Settings → Devices & Services, and the device will appear under "Discovered." Tap to configure and enter your Wi-Fi credentials.

2. **Captive Portal (Fallback)**: If Bluetooth is unavailable, the controller creates a Wi-Fi access point named `irrigation-controller-XXXXXX`. Where 'XXXXXX' is the specific MAC address for your device. Connect to it (no password required), and a portal page will open where you can enter your Wi-Fi credentials. If the portal does not open automatically, navigate to `http://192.168.4.1` in the browser on the device connected to the Wi-Fi access point.

3. **USB Serial**: Connect via USB and use the ESPHome Dashboard or CLI to enter Wi-Fi credentials over the Improv Serial protocol.

Once connected to Wi-Fi, Home Assistant will automatically discover the device. Accept the integration to add all entities.

---

## Configuration

### Substitutions
Customize these values in your configuration:

```yaml
substitutions:
  name: "irrigation-controller"
  friendly_name: "Irrigation Controller"
  zone_1_name: "Front Lawn"
  zone_2_name: "Back Lawn"
  # ... customize zone names as needed
```

### Zone 8 Modes
Zone 8 can operate in three modes (configurable via Home Assistant):

1. **Zone Mode**: Normal irrigation zone
2. **Master Valve Mode**: Opens immediately when any zone activates, closes when all zones complete
3. **Pump Start Relay Mode**: Activates 2 seconds after first zone opens, stays on until all zones complete

### Rain Sensor
An optional wired rain sensor can be connected to GPIO4 (default) to automatically suspend irrigation during and after rainfall.

- **Wiring**: Connect one wire to GPIO4 (or your configured rain sensor pin), the other wire to GND. The controller uses an internal pull-up resistor — no external resistor is needed.
- **Sensor Type**: Configurable as Normally Open (NO) or Normally Closed (NC) via the "Rain Sensor Type" selector in Home Assistant. Default is NO.
  - **Normally Open (NO)**: Circuit is open when dry, closes when wet. Rain is detected when the pin is pulled to GND.
  - **Normally Closed (NC)**: Circuit is closed when dry, opens when wet. Rain is detected when the pin floats HIGH (via the internal pull-up).
- **Rain Sensor Enabled**: Master toggle — when OFF, the rain sensor hardware is still monitored but has no effect on irrigation.
- **Rain Detection**: When rain is detected (and the sensor is enabled):
  - Any actively running irrigation is **immediately stopped**, including manually started runs
  - The status displays "Rain Detected - Stopped"
  - The status LED changes to alternating green/yellow (if Wi-Fi connected) or green/blue (if Wi-Fi disconnected)
  - **Scheduled** irrigation will not start while rain is actively detected
  - **After the initial shutoff**, you can manually override by turning on zones, using auto-advance, or starting irrigation from Home Assistant. The rain sensor will not prevent manual operation after the initial stop — however, if the sensor transitions from dry to wet again, it will stop irrigation again.
- **Rain Delay**: Optional feature that prevents scheduled irrigation from resuming too soon after rain stops.
  - Enable via the "Rain Delay Enabled" switch
  - Set the delay period with "Rain Delay Hours" (1-72 hours, default 48)
  - The delay timer starts when rain is first detected and persists across reboots
  - While the delay is active, **scheduled** runs are skipped with status "Schedule Skipped - Rain Delay"
  - Manual zone operation and auto-advance remain available during the delay period
  - The "Rain Delay Active" sensor shows ON in Home Assistant during the delay period
- **Rain Cleared**: When the sensor dries out, the status updates to "Rain Cleared", the LED returns to normal (green pulse or blue pulse), and irrigation can resume (subject to rain delay if enabled).
- **Rain Sensor entity**: Shows the raw GPIO pin state. Note: this reflects the physical pin state and does not account for the NO/NC setting. The controller's internal logic correctly interprets the pin state based on the NO/NC configuration for all rain detection, irrigation shutoff, and delay behavior.
- **Regulatory Compliance**: Some US states require rain sensors on automatic irrigation systems (including Florida, California, New Jersey, and Georgia). California requires a 48-hour delay after rainfall, which is the default. Users are responsible for complying with local regulations.

---

## Valve Overlap (v1.3.0+)

The controller uses valve overlap for seamless zone transitions:

- Next zone opens 3 seconds BEFORE current zone closes
- Pump/master valve stays on throughout entire cycle
- Eliminates pump cycling between zones
- Reduces water hammer and pressure spikes

### Timing
| Event | Timing |
|-------|--------|
| Zone valve opens | Immediate |
| Pump starts | 2 seconds after first zone |
| Next zone opens | 3 seconds before current closes |
| Pump stops | When last zone closes |

---

## Home Assistant Entities

### Controls
- Start/Stop/Resume (main switch)
- Auto Advance (on/off)
- Repeat (number of cycles)
- Zone 1-32 switches
- Enable Zone 1-32 switches
- Zone 1-32 Duration (minutes)
- Schedule Enabled
- Schedule days (Mon-Sun)
- Schedule Start Times 1-4
- Zone 8 Mode selector
- Rain Sensor Enabled
- Rain Delay Enabled
- Rain Sensor Type (NC/NO)
- Rain Delay Hours

### Sensors
- Status (current activity)
- Time Remaining
- Detected Zones
- IP Address
- Connected SSID
- ESPHome Version
- Rain Sensor (raw GPIO pin state — does not reflect NO/NC logic)
- Rain Delay Active (ON when delay timer is counting down)

### Updates
- Firmware Update entity for OTA updates

---

## Status LED

The RGB status LED (WS2812 on GPIO42) indicates system state:

| Color | Pattern | Meaning |
|-------|---------|---------|
| Green | Pulsing | Wi-Fi connected, no rain |
| Green ↔ Yellow | Alternating | Wi-Fi connected, rain detected |
| Blue | Pulsing | Wi-Fi disconnected, no rain |
| Green ↔ Blue | Alternating | Wi-Fi disconnected, rain detected |

---

## OTA Updates

Firmware updates are hosted on GitHub Releases. When a new version is available:

1. The "Firmware Update" entity in Home Assistant will show an update
2. Click Install to download and apply the update
3. The device reboots automatically with new firmware

---

## Manual Switch Safety

If using physical manual switches on the controller board:

**WARNING**: When Zone 8 is configured as Pump Start Relay or Master Valve, NEVER activate Zone 8 manually without first activating at least one other zone. Running a pump with no open valves ("dead head") can cause pump damage.

Safe manual operation:
1. First, activate zone valve(s)
2. Wait 2-3 seconds for valve to open
3. Then activate Zone 8 (pump) if needed
4. When stopping: First deactivate Zone 8, then zone valves

This safety timing is handled automatically when operating via Home Assistant.

---

## File Structure

```
/
├── IrrigationControllerMain.yaml    # Main ESPHome configuration
├── manifest.json                     # OTA update manifest
├── README.md                         # This file
└── releases/                         # Firmware releases
    └── v1.3.0/
        └── firmware.bin
```

---

## Version History

### v1.4.0
- Rain sensor support with configurable GPIO pin and NO/NC type selection
- Rain delay feature (1-72 hours, default 48) for water conservation compliance
- Immediate irrigation shutoff on rain detection
- Scheduled runs skipped during active rain or rain delay period
- Rain Delay Active binary sensor for Home Assistant
- Status LED reflects rain state (green/yellow alternating when raining)

### v1.3.0
- Added valve overlap (3 seconds) for seamless zone transitions
- Integrated pump management via ESPHome sprinkler component
- Pump/master valve no longer cycles between zones
- Simplified codebase for better reliability

### v1.2.5
- OTA updates via GitHub Releases
- Auto Advance resets after each cycle completes; scheduled runs enable it automatically and restore your preference afterward
- Zone 8 mode enforcement
- Timer icons for duration entities

### v1.2.0
- Made for ESPHome compliance
- Bluetooth/Improv Wi-Fi provisioning
- Dashboard import support

---

## Troubleshooting

### Device not connecting to Wi-Fi
- Ensure 2.4GHz network (5GHz not supported)
- Try captive portal: connect to the "irrigation-controller-XXXXXX" AP (no password required), then navigate to `http://192.168.4.1` if the portal does not open automatically
- Verify Wi-Fi credentials are correct

### Pump not starting
- Verify Zone 8 Mode is set to "Pump Start Relay"
- 2-second delay is normal before pump starts
- Ensure at least one zone is active

### Zones not running
- Check zone is enabled (Enable Zone X = ON)
- Verify duration is greater than 0
- Check Schedule Enabled if using scheduling

### Expansion zones not working
- Verify I2C wiring (SDA to GPIO8, SCL to GPIO9)
- Check "Detected Zones" sensor for board detection
- Confirm correct I2C addresses (0x20, 0x21, 0x22)

### Rain sensor not working
- Verify "Rain Sensor Enabled" is ON
- Check "Rain Sensor Type" matches your hardware (NC vs NO)
- Wiring: one wire to rain sensor GPIO pin (default GPIO4), other wire to GND
- Check the "Rain Sensor" binary sensor for the raw GPIO state
- Note: The "Rain Sensor" entity shows the raw pin state and does NOT reflect the NO/NC logic. The status text, LED, and irrigation behavior all correctly interpret the NO/NC setting internally. If the Rain Sensor entity appears inverted from what you expect, that is normal — check the Status text and LED color to confirm rain is being detected correctly.

### Irrigation not resuming after rain
- Check if "Rain Delay Active" is ON — the delay timer may still be counting down
- Reduce "Rain Delay Hours" or disable "Rain Delay Enabled" if not needed
- The rain delay timer starts when rain is first detected and persists across reboots
- To cancel an active rain delay, turn off "Rain Delay Enabled"

---

## License

This project is open source. See LICENSE file for details.

---

## Support

- GitHub Issues: https://github.com/FluxOpenHome/IrrigationController/issues
- Documentation: See User Manual in releases

---

Made for ESPHome | Flux Open Home