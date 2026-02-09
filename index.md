---
title: Flux Open Home Irrigation Controller
date-published: 2025-01-01
type: misc
standard: global
board: esp32
project-url: https://github.com/FluxOpenHome/IrrigationController
difficulty: 3
made-for-esphome: true
---

## Description

The Flux Open Home Irrigation Controller is an ESP32-S3 based smart irrigation system designed for ESPHome and Home Assistant. It supports up to 32 zones (8 built-in + 24 via I2C expansion boards), built-in scheduling, rain sensor support, and seamless zone transitions with valve overlap.

### Key Features

- **Up to 32 Zones**: 8 built-in relay-driven zones + 24 expansion zones via MCP23017 I2C boards
- **Built-in Scheduling**: 4 programmable start times with 7-day scheduling
- **Rain Sensor Support**: Wired rain sensor input with configurable NO/NC type, automatic shutoff, and rain delay timer (1-72 hours)
- **Valve Overlap**: Next zone opens before the current zone closes, preventing pump cycling
- **Pump/Master Valve Control**: Zone 8 configurable as a normal zone, master valve, or pump start relay
- **Auto Advance**: Automatically cycles through enabled zones
- **Repeat Cycles**: Run multiple passes through all zones (0-5 cycles)
- **OTA Updates**: Firmware updates delivered via Home Assistant
- **Expansion Board Auto-Detection**: Automatically detects connected MCP23017 I2C boards on boot
- **Status LED**: WS2812 RGB LED indicates Wi-Fi, rain, and system state
- **Wi-Fi Provisioning**: Bluetooth (Improv), captive portal, or USB serial

## Hardware Requirements

- ESP32-S3 DevKitC-1 or compatible
- 8-channel relay module
- WS2812 RGB LED (optional, for status indication)
- Rain sensor (optional, wired NO or NC type)
- MCP23017 I2C I/O expander boards (optional, up to 3 for zones 9-32)

## GPIO Pinout

| Pin    | Function              |
| ------ | --------------------- |
| GPIO45 | Zone 1 Relay          |
| GPIO35 | Zone 2 Relay          |
| GPIO36 | Zone 3 Relay          |
| GPIO37 | Zone 4 Relay          |
| GPIO38 | Zone 5 Relay          |
| GPIO39 | Zone 6 Relay          |
| GPIO40 | Zone 7 Relay          |
| GPIO41 | Zone 8 Relay          |
| GPIO42 | Status LED (WS2812)   |
| GPIO4  | Rain Sensor (default) |
| GPIO8  | I2C SDA               |
| GPIO9  | I2C SCL               |

### Expansion Boards (MCP23017)

| Board   | Zones  | I2C Address |
| ------- | ------ | ----------- |
| Board 1 | 9-16   | 0x20        |
| Board 2 | 17-24  | 0x21        |
| Board 3 | 25-32  | 0x22        |

Boards are sequential: Board 2 requires Board 1, Board 3 requires Boards 1 and 2.

## Setup

1. Flash the ESP32-S3 with the Irrigation Controller firmware via ESPHome Dashboard, USB, or the web installer.
2. Connect to the device using one of the provisioning methods:
   - **Bluetooth (Recommended)**: Open the Home Assistant Companion app, go to Settings > Devices & Services, and add the discovered device.
   - **Captive Portal**: Connect to the `irrigation-controller-XXXXXX` Wi-Fi AP (no password), then enter credentials at `192.168.4.1`.
   - **USB Serial**: Use the ESPHome Dashboard or CLI with the Improv Serial protocol.
3. Once connected to Wi-Fi, Home Assistant will automatically discover the device. Accept the integration to add all entities.

## Basic Configuration

```yaml
substitutions:
  name: "irrigation-controller"
  friendly_name: "Irrigation Controller"
  zone_1_name: "Front Lawn"
  zone_2_name: "Back Lawn"
  zone_3_name: "Side Yard"
  zone_4_name: "Garden"
  zone_5_name: "Flower Beds"
  zone_6_name: "Driveway Strip"
  zone_7_name: "Backyard"
  zone_8_name: "Zone 8"
  rain_sensor_pin: "GPIO4"

dashboard_import:
  package_import_url: github://FluxOpenHome/IrrigationController/IrrigationControllerMain.yaml@main
  import_full_config: true
```

## Home Assistant Entities

### Controls

- **Start/Stop/Resume**: Main irrigation switch
- **Auto Advance**: Automatically cycle through enabled zones
- **Repeat Cycles**: Number of passes (0-5)
- **Zone 1-32 Switches**: Direct zone control
- **Enable Zone 1-32**: Enable/disable zone participation
- **Zone 1-32 Duration**: Run time per zone in minutes (0-999)
- **Zone 8 Mode**: Selector for Zone, Master Valve, or Pump Start Relay
- **Schedule Enabled**: Master schedule toggle
- **Schedule Days**: Monday through Sunday toggles
- **Schedule Start Times 1-4**: Configurable start times (HH:MM)
- **Rain Sensor Enabled**: Master rain sensor toggle
- **Rain Sensor Type**: Normally Open (NO) or Normally Closed (NC)
- **Rain Delay Enabled**: Enable rain delay timer
- **Rain Delay Hours**: Delay period (1-72 hours, default 48)

### Sensors

- **Status**: Current activity description
- **Time Remaining**: Minutes and seconds remaining on current zone
- **Progress**: Percentage progress of current zone
- **Detected Zones**: Total zones and detected expansion boards
- **Rain Sensor**: Raw GPIO pin state
- **Rain Delay Active**: ON when delay timer is counting down
- **IP Address**, **SSID**, **WiFi Signal**, **ESPHome Version**, **Uptime**

### Updates

- **Firmware Update**: OTA updates from GitHub Releases

## Zone 8 Modes

Zone 8 can be configured via a Home Assistant selector:

- **Zone**: Normal irrigation zone
- **Master Valve**: Opens immediately when any zone activates, closes when all zones complete
- **Pump Start Relay**: Activates 2 seconds after the first zone opens, stays on until all zones complete

## Rain Sensor

Connect a wired rain sensor to GPIO4 (default) and GND. The controller uses an internal pull-up resistor.

- Configurable as NO or NC via Home Assistant
- Immediately stops active irrigation on rain detection
- Scheduled runs are skipped during active rain or rain delay
- Rain delay timer (1-72 hours) prevents premature schedule resumption
- Manual operation remains available after initial shutoff

## Valve Overlap

Starting in v1.3.0, the controller uses valve overlap for seamless transitions:

| Event               | Timing                            |
| ------------------- | --------------------------------- |
| Zone valve opens    | Immediate                         |
| Pump starts         | 2 seconds after first zone        |
| Next zone opens     | 3 seconds before current closes   |
| Pump stops          | When last zone closes             |

## Status LED

| Color             | Pattern      | Meaning                          |
| ----------------- | ------------ | -------------------------------- |
| Green             | Pulsing      | Wi-Fi connected, no rain         |
| Green / Yellow    | Alternating  | Wi-Fi connected, rain detected   |
| Blue              | Pulsing      | Wi-Fi disconnected, no rain      |
| Green / Blue      | Alternating  | Wi-Fi disconnected, rain detected|

## Links

- [GitHub](https://github.com/FluxOpenHome/IrrigationController)
- [Issues](https://github.com/FluxOpenHome/IrrigationController/issues)
