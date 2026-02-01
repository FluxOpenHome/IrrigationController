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

### Expansion Boards (Optional)
- MCP23017 I2C I/O Expanders
- Board 1 (Zones 9-16): Address 0x20
- Board 2 (Zones 17-24): Address 0x21
- Board 3 (Zones 25-32): Address 0x22
- I2C: SDA on GPIO8, SCL on GPIO9

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

### Sensors
- Status (current activity)
- Time Remaining
- Detected Zones
- IP Address
- Connected SSID
- ESPHome Version

### Updates
- Firmware Update entity for OTA updates

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
- Try captive portal: connect to "irrigation-controller-XXXXXX" AP
- Default AP password: 12345678

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

---

## License

This project is open source. See LICENSE file for details.

---

## Support

- GitHub Issues: https://github.com/FluxOpenHome/IrrigationController/issues
- Documentation: See User Manual in releases

---

Made for ESPHome | Flux Open Home