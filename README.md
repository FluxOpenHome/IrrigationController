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
- **Auto Advance**: Automatically cycles through enabled zones
- **Repeat Cycles**: Run multiple passes through all zones
- **Wi-Fi Provisioning**: Bluetooth, captive portal, or serial configuration
- **OTA Firmware Updates**: Update directly from Home Assistant
- **Expansion Board Auto-Detection**: Automatically detects connected I2C expansion boards
- **Configurable Expansion Board Count**: Select how many boards are installed; errors only flagged for missing expected boards
- **Rain Sensor Support**: Optional wired rain sensor with configurable NO/NC type, automatic irrigation shutoff, and rain delay timer (1-72 hours, default 48)
- **9-State Status LED**: Priority-based RGB LED shows system health at a glance

---

## Hardware Requirements

### Main Controller
- ESP32-S3 DevKitC-1 or compatible
- 8-channel relay module (directly connected to GPIO)
- RGB LED for status indication (WS2812 on GPIO48)

### GPIO Pin Assignments (Main Board)
- Zone 1: GPIO45
- Zone 2: GPIO35
- Zone 3: GPIO36
- Zone 4: GPIO37
- Zone 5: GPIO38
- Zone 6: GPIO39
- Zone 7: GPIO40
- Zone 8: GPIO41
- Rain Sensor: GPIO18 (configurable via substitution)

### Expansion Boards (Optional)
- MCP23017 I2C I/O Expanders
- Board 1 (Zones 9-16): Address 0x20
- Board 2 (Zones 17-24): Address 0x21
- Board 3 (Zones 25-32): Address 0x22
- I2C: SDA on GPIO8, SCL on GPIO9
- Boards are sequential: Board 2 requires Board 1, Board 3 requires Board 1 and 2
- Use the "Expansion Boards Installed" selector in Home Assistant to set your board count

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
An optional wired rain sensor can be connected to automatically suspend irrigation during and after rainfall.

- **Wiring**: Connect one wire to the rain sensor GPIO pin, the other to GND
- **Sensor Type**: Configurable as Normally Closed (NC) or Normally Open (NO) via Home Assistant
- **Rain Detection**: Immediately stops all active irrigation when rain is detected
- **Rain Delay**: Optional configurable delay (1-72 hours, default 48) that prevents scheduled irrigation from resuming too soon after rain. Helps comply with local water conservation regulations.
- **Schedule Protection**: Scheduled runs are skipped during active rain or while the rain delay timer is active

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
- Expansion Boards Installed selector (0, 1, 2, or 3)
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
- Rain Sensor (raw GPIO state)
- Rain Detected (effective state when enabled)
- Rain Delay Active

### Updates
- Firmware Update entity for OTA updates

---

## Status LED

The RGB status LED uses a priority system (highest priority shown first):

| Priority | Color | Pattern | Meaning |
|----------|-------|---------|---------|
| 1 | Red | Pulsing | I2C bus error |
| 2 | Orange | Pulsing | Expansion board mismatch (expected board missing) |
| 3 | Cyan | Fast Pulsing | OTA update in progress |
| 4 | White | Pulsing | Unknown I2C device detected |
| 5 | Green | Solid | Zone actively running |
| 6 | Blue | Pulsing | Wi-Fi disconnected |
| 7 | Yellow | Pulsing | Home Assistant not connected |
| 8 | Magenta | Pulsing | Captive portal active |
| 9 | Green | Pulsing | Normal operation (idle) |

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
- Rain Detected and Rain Delay Active binary sensors for Home Assistant

### v1.3.1
- Added "Expansion Boards Installed" selector (0, 1, 2, or 3)
- Added board mismatch error detection with orange LED indicator
- Full 9-state priority status LED system
- I2C bus error only flagged when expansion boards are expected
- Detection re-runs automatically when board count setting changes

### v1.3.0
- Added valve overlap (3 seconds) for seamless zone transitions
- Integrated pump management via ESPHome sprinkler component
- Pump/master valve no longer cycles between zones
- Simplified codebase for better reliability

### v1.2.5
- OTA updates via GitHub Releases
- Auto-advance preference memory
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
- Verify "Expansion Boards Installed" selector matches your physical hardware
- Verify I2C wiring (SDA to GPIO8, SCL to GPIO9)
- Check "Detected Zones" sensor for board detection
- Confirm correct I2C addresses (0x20, 0x21, 0x22)

### Orange pulsing LED (board mismatch)
- An expected expansion board was not detected on the I2C bus
- Check that "Expansion Boards Installed" matches the boards you have connected
- Verify I2C wiring and board address jumpers
- If you removed a board, update the selector to the new count

### Red pulsing LED (I2C bus error)
- No I2C devices responding on the bus at all
- Check SDA (GPIO8) and SCL (GPIO9) wiring
- Verify power supply to expansion boards
- Check for short circuits on the I2C bus

### Rain sensor not working
- Verify "Rain Sensor Enabled" is ON
- Check "Rain Sensor Type" matches your hardware (NC vs NO)
- Wiring: one wire to rain sensor GPIO pin, other wire to GND
- Check "Rain Sensor" binary sensor for raw hardware state

### Irrigation not resuming after rain
- Check if "Rain Delay Active" is ON (delay period may still be active)
- Reduce "Rain Delay Hours" or disable "Rain Delay Enabled" if not needed

---

## License

This project is open source. See LICENSE file for details.

---

## Support

- GitHub Issues: https://github.com/FluxOpenHome/IrrigationController/issues
- Documentation: See User Manual in releases

---

Made for ESPHome | Flux Open Home