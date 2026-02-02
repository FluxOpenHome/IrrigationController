**Flux Open Home**

**Irrigation Controller**

User Manual

Version 1.4.0

*Made for ESPHome*

# **Table of Contents**

1\. Overview

2\. Initial Setup

3\. Status LED Indicators

4\. Zone Configuration

5\. Zone 8 Special Modes

6\. Valve Overlap & Seamless Zone Transitions

7\. Manual Switches - Important Safety Information

8\. Operating Zones from Home Assistant

9\. Duration Settings

10\. Auto Advance Feature

11\. Repeat Cycles

12\. Scheduling

13\. Rain Sensor

14\. Expansion Boards

15\. Firmware Updates

16\. Troubleshooting

17\. Version History

# **1. Overview**

The Flux Open Home Irrigation Controller is an ESP32-S3 based smart
irrigation system that supports up to 32 zones:

-   Zones 1-8: Built-in on the main controller board

-   Zones 9-16: Optional Expansion Board 1 (I2C address 0x20)

-   Zones 17-24: Optional Expansion Board 2 (I2C address 0x21)

-   Zones 25-32: Optional Expansion Board 3 (I2C address 0x22)

The controller integrates with Home Assistant and provides features
including scheduled watering, auto-advance through zones, repeat cycles,
valve overlap for seamless transitions, and intelligent pump/master
valve management.

# **2. Initial Setup**

## **Wi-Fi Provisioning**

The controller supports multiple methods to connect to your Wi-Fi
network. On first boot (or if Wi-Fi credentials are lost), the
controller will automatically enter provisioning mode so you can
configure your network settings.

### **Bluetooth Provisioning (Recommended)**

This is the simplest method and works directly from the Home Assistant
mobile app:

1.  Power on the controller — it will begin advertising via Bluetooth
    automatically

2.  Open the Home Assistant Companion app on your phone or PC (with bluetooth)

3.  Go to **Settings → Devices & Services**

4.  The controller should appear under "Discovered" as a new ESPHome
    device

5.  Tap to configure and enter your Wi-Fi network name (SSID) and
    password when prompted

6.  The controller will connect to your Wi-Fi network and stop
    Bluetooth advertising

If the controller does not appear in discovered devices, ensure
Bluetooth is enabled on your phone and that you are within range
(approximately 10 meters / 30 feet).

### **Captive Portal (Fallback)**

If Bluetooth provisioning is not available (e.g., using a desktop
computer without Bluetooth), the controller automatically creates its
own Wi-Fi access point as a fallback:

1.  On your phone, tablet, or computer, open your Wi-Fi settings

2.  Look for a network named **irrigation-controller-XXXXXX** (where
    XXXXXX is a unique identifier derived from the device's MAC address)

3.  Connect to this network — no password is required

4.  A captive portal page should open automatically in your browser. If
    it does not, navigate to **http://192.168.4.1** in your browser

5.  Select your home Wi-Fi network from the list and enter your Wi-Fi
    password

6.  The controller will save the credentials, disconnect the access
    point, and connect to your home network

**Note:** The captive portal access point is only active when the
controller cannot connect to a previously configured Wi-Fi network.
During normal operation, the access point is not broadcast.

### **USB Serial Provisioning**

For advanced users or initial flashing, you can configure Wi-Fi over a
USB serial connection:

1.  Connect the controller to your computer using a USB-C cable

2.  Open the ESPHome Dashboard or use the ESPHome command-line tool

3.  The device will advertise itself via Improv Serial, allowing you to
    enter Wi-Fi credentials through the ESPHome interface

4.  Alternatively, you can use any serial terminal at 115200 baud to
    interact with the Improv Serial protocol

This method is particularly useful when first flashing the firmware onto
a brand-new ESP32-S3 board.

## **Adding to Home Assistant**

Once the controller is connected to your Wi-Fi network:

1.  Home Assistant will automatically discover the device within a few
    minutes

2.  A notification will appear, or you can find it under **Settings →
    Devices & Services → Discovered**

3.  Click **Configure** to add the device

4.  All entities (zones, sensors, controls) will be created
    automatically

5.  The device is now ready to use — no additional configuration is
    required in Home Assistant

If the device is not discovered automatically, verify that both Home
Assistant and the controller are on the same network and subnet.

# **3. Status LED Indicators**

The controller has an RGB status LED that indicates the current state.
The LED uses a priority system, showing only the highest-priority
condition:

  -----------------------------------------------------------------------
  **Priority**  **LED Color**     **Pattern**     **Meaning**
  ------------- ----------------- --------------- -----------------------
  1             Red               Pulsing         I2C bus error (no
                                                  devices responding)

  2             Orange            Pulsing         Expansion board mismatch
                                                  (expected board missing)

  3             Cyan              Fast Pulsing    OTA update in progress

  4             White             Pulsing         Unknown I2C device
                                                  detected

  5             Green             Solid (no       Zone actively running
                                  pulse)

  6             Blue              Pulsing         Wi-Fi disconnected

  7             Yellow            Pulsing         Home Assistant not
                                                  connected

  8             Magenta           Pulsing         Captive portal active

  9             Green             Pulsing         Normal operation (idle,
                                                  all connected)
  -----------------------------------------------------------------------

# **4. Zone Configuration**

## **Enabling/Disabling Zones**

Each zone has an \"Enable\" switch in Home Assistant. Only enabled zones
will:

-   Run during scheduled irrigation

-   Be included when using Auto Advance

-   Respond to the Start/Stop/Resume control

## **Zone Names**

Default zone names are \"Zone 1\" through \"Zone 32\". These can be
customized in the ESPHome configuration by modifying the substitutions.

# **5. Zone 8 Special Modes**

Zone 8 can operate in three different modes, configured via the \"Zone 8
Mode\" selector in Home Assistant:

### **Zone Mode (Normal irrigation zone)**

-   Zone 8 operates as a normal irrigation zone

-   Can be enabled, scheduled, and run like any other zone

-   No pump/master valve functionality

### **Master Valve Mode**

-   Zone 8 relay controls a master valve upstream of all zone valves

-   The master valve opens when the first zone starts

-   The master valve stays open throughout the entire irrigation cycle

-   The master valve closes only after the last zone completes

-   Zone 8 is automatically disabled as an irrigation zone in this mode

### **Pump Start Relay Mode**

-   Zone 8 relay controls a pump start relay

-   The pump starts 2 seconds after the first zone valve opens

-   The pump stays running throughout the entire irrigation cycle

-   The pump stops only after the last zone completes

-   Zone 8 is automatically disabled as an irrigation zone in this mode

**Important:** When Zone 8 is set to Master Valve or Pump Start Relay
mode, it cannot be enabled or run as an irrigation zone. The system
automatically prevents this.

# **6. Valve Overlap & Seamless Zone Transitions**

## **How It Works**

When running multiple zones in sequence (via Auto Advance or
scheduling), the controller uses valve overlap to ensure smooth
transitions between zones. This means:

1.  The next zone valve opens 3 seconds BEFORE the current zone valve
    > closes

2.  Both valves are briefly open simultaneously during the transition

3.  The current zone valve then closes while the next zone continues

## **Benefits**

-   Pump protection: The pump only starts once and stops once per
    > irrigation cycle

-   Reduced water hammer: Overlapping valves prevent sudden pressure
    > spikes

-   Smoother operation: No interruption in water flow between zones

-   Energy efficiency: Eliminates pump startup current spikes between
    > zones

## **Timing Summary**

  -----------------------------------------------------------------------
  **Event**                           **Timing**
  ----------------------------------- -----------------------------------
  Zone valve opens                    Immediate

  Pump/Master valve starts            2 seconds after first zone opens

  Next zone opens (overlap)           3 seconds before current zone
                                      closes

  Current zone closes                 After overlap period

  Pump/Master valve stops             When last zone closes
  -----------------------------------------------------------------------

# **7. Manual Switches - Important Safety Information**

The controller board includes physical manual switches for each zone
relay. These allow you to activate zones without using Home Assistant.

**⚠️ CRITICAL WARNING FOR ZONE 8 ⚠️**

**If Zone 8 is configured as \"Pump Start Relay\" or \"Master Valve\":**
NEVER activate the Zone 8 manual switch unless at least one other zone
switch (Zones 1-7 or expansion zones) is already activated.

### **Why this matters:**

-   In Pump Start Relay mode, Zone 8 controls your irrigation pump

-   Activating a pump without any zone valves open creates a \"dead
    > head\" condition

-   This can cause rapid pump overheating, motor damage, pipe pressure
    > buildup, and potential fitting damage

### **Safe operation of manual switches:**

4.  **First**, activate the zone(s) you want to water (Zones 1-7 or
    > expansion zones)

5.  **Wait** for the valve(s) to fully open (2-3 seconds)

6.  **Then** (and only if needed) activate Zone 8 manually

7.  When finished, **first** deactivate Zone 8

8.  **Then** deactivate the zone valves

**Note:** When operating through Home Assistant or the scheduled system,
the controller automatically handles all timing, valve overlap, and pump
safety logic.

# **8. Operating Zones from Home Assistant**

## **Starting a Single Zone**

9.  Find the zone switch (e.g., \"Zone 1\") in Home Assistant

10. Turn it on - the zone will run for its configured duration

11. If Zone 8 is configured as Pump Start Relay or Master Valve, it will
    > automatically activate

12. The \"Status\" text sensor shows which zone is active

13. The \"Time Remaining\" sensor shows countdown

## **Start/Stop/Resume Control**

The \"Start/Stop/Resume\" switch provides master control:

-   Turn ON: Starts the first enabled zone (or resumes if paused)

-   Turn OFF: Stops all irrigation

# **9. Duration Settings**

Each zone has a duration setting that determines how long it runs:

-   Entity name: \"Zone X Duration\"

-   Unit: Minutes

-   Default: 2 minutes

-   Icon: Timer icon

# **10. Auto Advance Feature**

The \"Auto Advance\" switch controls whether the system automatically
moves to the next enabled zone after each zone completes.

### **With Auto Advance ON:**

-   Start any zone or use Start/Stop/Resume

-   After the zone\'s duration completes, the next enabled zone starts
    > automatically

-   Valve overlap ensures smooth transitions - the next zone opens
    > before the current one closes

-   The pump/master valve stays on throughout the entire cycle

-   Stops after the last enabled zone completes

### **With Auto Advance OFF:**

-   Only the selected zone runs

-   System stops after that zone completes

-   You must manually start each zone

# **11. Repeat Cycles**

The \"Repeat\" number entity controls how many complete cycles run
through all enabled zones:

-   Default: 1 (single pass through all zones)

-   Setting it to 2: Runs through all enabled zones twice

-   Setting it to 3: Runs through all enabled zones three times

This is useful for sandy soils, slopes where water needs time to absorb,
and deep watering applications.

**Note:** The pump/master valve remains on throughout all repeat cycles,
only turning off when the final cycle completes.

# **12. Scheduling**

The controller supports automated scheduling with the following
controls:

### **Schedule Enabled**

Master switch to enable/disable all scheduled runs.

### **Schedule Days**

Seven switches to select which days the schedule runs: Monday through
Sunday.

### **Schedule Start Times**

Up to four start times can be configured. Enter times in HH:MM format
(24-hour). Leave blank to disable that time slot.

## **How Scheduling Works:**

14. Enable \"Schedule Enabled\"

15. Select the days you want irrigation to run

16. Set one or more start times

17. At the scheduled time, all enabled zones will run in sequence with
    > Auto Advance

18. Valve overlap ensures smooth transitions between zones

19. The pump/master valve (if configured) runs continuously throughout
    > the schedule

# **13. Rain Sensor**

The controller supports an optional wired rain sensor to automatically
suspend irrigation when rain is detected.

## **Wiring**

Most wired rain sensors are passive devices with two wires. Connect one
wire to the configured GPIO pin (default: GPIO4) and the other wire to
GND on the controller. The controller uses an internal pull-up resistor,
so no external resistor is needed.

To change the GPIO pin, modify the `rain_sensor_pin` substitution in
your ESPHome configuration.

## **Enabling the Rain Sensor**

In Home Assistant, turn on the "Rain Sensor Enabled" switch. When
disabled, the rain sensor hardware is still monitored but will not
affect irrigation operation.

## **Sensor Type (NO/NC)**

Rain sensors come in two wiring configurations:

-   **Normally Closed (NC)**: The circuit is closed when dry and opens
    when wet. This is the most common type.

-   **Normally Open (NO)**: The circuit is open when dry and closes
    when wet.

Use the "Rain Sensor Type" selector in Home Assistant to match your
sensor. The default is Normally Closed (NC).

## **Rain Detection Behavior**

When rain is detected and the rain sensor is enabled:

-   Any actively running irrigation is **immediately stopped**

-   The status displays "Rain Detected - Stopped"

-   Scheduled irrigation will not start while rain is actively detected

When the sensor clears (dries out), the status updates to "Rain
Cleared" and normal operation resumes (subject to rain delay, if
enabled).

## **Rain Delay**

The rain delay feature prevents irrigation from resuming too soon after
rain has stopped. This helps comply with local water conservation
regulations — for example, some jurisdictions require a 48-hour delay
after rainfall before automatic irrigation can resume.

### **Configuring Rain Delay:**

1.  Turn on the "Rain Delay Enabled" switch in Home Assistant

2.  Set the "Rain Delay Hours" value (default: 48 hours, range: 1-72
    hours)

When rain is detected, a delay timer is set for the configured number
of hours from the time of detection. During the delay period:

-   Scheduled irrigation is skipped with status "Schedule Skipped -
    Rain Delay"

-   The "Rain Delay Active" sensor shows ON in Home Assistant

-   The delay timer persists across reboots

### **Regulatory Compliance Note:**

Rain sensor and rain delay requirements vary by jurisdiction. Some US
states (including Florida, California, New Jersey, and Georgia, among
others) require rain sensors on automatic irrigation systems. California
specifically requires a 48-hour delay, which is the default setting.
Users are responsible for verifying and complying with their local
regulations. Adjust the rain delay hours as needed for your area.

## **Home Assistant Entities**

  -----------------------------------------------------------------------
  **Entity**                          **Type**
  ----------------------------------- -----------------------------------
  Rain Sensor Enabled                 Switch (toggle)

  Rain Delay Enabled                  Switch (toggle)

  Rain Sensor Type                    Select (NC / NO)

  Rain Delay Hours                    Number (1-72, default 48)

  Rain Sensor                         Binary Sensor (raw GPIO state)

  Rain Detected                       Binary Sensor (effective state)

  Rain Delay Active                   Binary Sensor
  -----------------------------------------------------------------------

# **14. Expansion Boards**

The controller supports up to three optional MCP23017 expansion boards,
adding 8 zones each. Expansion boards are sequential — Board 2 requires
Board 1, and Board 3 requires both Board 1 and Board 2.

## **Configuring Expansion Board Count**

In Home Assistant, find the \"Expansion Boards Installed\" selector under
the device's configuration entities. Set it to the number of expansion
boards you have physically installed:

-   **0** - No expansion boards (zones 1-8 only)

-   **1** - One expansion board (zones 1-16)

-   **2** - Two expansion boards (zones 1-24)

-   **3** - Three expansion boards (zones 1-32)

This setting is saved across reboots. Changing it immediately triggers a
re-scan of the I2C bus.

## **Viewing Detected Zones**

The \"Detected Zones\" text sensor shows the total number of available
zones and which expansion boards are connected.

## **Expansion Board Addresses**

  -----------------------------------------------------------------------
  **Board**                           **Address**
  ----------------------------------- -----------------------------------
  Board 1 (Zones 9-16)                I2C address 0x20

  Board 2 (Zones 17-24)               I2C address 0x21

  Board 3 (Zones 25-32)               I2C address 0x22
  -----------------------------------------------------------------------

## **Automatic Zone Disabling**

If an expansion board is not detected, zones on that board are
automatically disabled to prevent errors.

## **Error Detection**

The controller compares the number of boards you configured against what
is actually detected on the I2C bus:

-   **No error:** If you set \"0\" and no boards are found, this is
    expected and no error is flagged.

-   **Board mismatch (orange LED):** If you configured a board count but
    one or more expected boards are not detected, the status LED pulses
    orange. Check your I2C wiring and board addresses.

-   **I2C bus error (red LED):** If you expect boards but no devices at
    all respond on the I2C bus, this indicates a possible wiring fault
    or bus failure.

# **15. Firmware Updates**

The controller supports over-the-air (OTA) firmware updates directly
from Home Assistant.

## **Checking for Updates**

In Home Assistant, find the \"Firmware Update\" entity. It will show if
an update is available.

## **Installing Updates**

20. Click on the Firmware Update entity

21. Click \"Install\"

22. Wait for the download and installation to complete

23. The device will automatically reboot with the new firmware

# **16. Troubleshooting**

### **Device not connecting to Wi-Fi**

-   Check that the LED is pulsing blue (looking for connection)

-   Try the captive portal method — look for the
    "irrigation-controller-XXXXXX" network in your Wi-Fi settings and
    connect to it (no password required), then navigate to
    http://192.168.4.1 if the portal does not open automatically

-   Verify your Wi-Fi credentials are correct

-   Ensure your router supports 2.4GHz (5GHz is not supported)

### **Zones not running**

-   Verify the zone is enabled (Enable Zone X switch is ON)

-   Check the zone duration is greater than 0

-   Verify Zone 8 Mode if zones 1-7 aren\'t triggering the pump

### **Pump not starting (Zone 8 as Pump Start Relay)**

-   Verify Zone 8 Mode is set to \"Pump Start Relay\"

-   Remember there\'s a 2-second delay before the pump starts

-   Check that at least one zone valve is active

-   The pump won\'t start if Zone 8 Mode is set to \"Zone\"

### **Pump cycling on/off between zones**

This should NOT happen with firmware v1.3.0+. If you experience this:

-   Verify you have the latest firmware installed

-   Check that Zone 8 Mode is set correctly

-   Ensure Auto Advance is enabled for multi-zone runs

### **Schedule not running**

-   Verify \"Schedule Enabled\" is ON

-   Check that at least one day is selected

-   Verify at least one start time is configured

-   Ensure the time in Home Assistant is correct

-   Confirm zones are enabled

### **Expansion board zones not working**

-   Verify the \"Expansion Boards Installed\" selector matches the number
    of boards you have physically installed

-   Check the \"Detected Zones\" sensor

-   Verify I2C connections to expansion boards

-   Ensure correct I2C addresses (0x20, 0x21, 0x22)

### **Orange pulsing LED (board mismatch)**

-   One or more expected expansion boards were not detected

-   Check that the \"Expansion Boards Installed\" setting matches your
    physical hardware

-   Verify I2C wiring (SDA/SCL connections) to the missing board(s)

-   Ensure expansion board address jumpers are set correctly

-   If you removed an expansion board, update the selector to reflect
    the new count

### **Red pulsing LED (I2C bus error)**

-   No I2C devices are responding on the bus at all

-   Check SDA (GPIO8) and SCL (GPIO9) wiring

-   Verify power supply to expansion boards

-   Check for short circuits on the I2C bus

### **Rain sensor not detecting rain**

-   Verify "Rain Sensor Enabled" is ON

-   Check the "Rain Sensor Type" setting matches your sensor (NC vs NO)

-   Verify wiring: one wire to the configured GPIO pin, other wire to GND

-   Check the "Rain Sensor" binary sensor in Home Assistant to see the
    raw hardware state

### **Irrigation not resuming after rain**

-   Check if "Rain Delay Active" is still ON — the delay period may not
    have elapsed yet

-   Verify "Rain Delay Hours" is set to an appropriate value

-   If you want irrigation to resume immediately after rain clears,
    turn off "Rain Delay Enabled"

# **17. Version History**

### **v1.4.0**

-   NEW: Rain sensor support with configurable GPIO pin

-   NEW: Normally Open / Normally Closed sensor type selector

-   NEW: Rain Delay feature with configurable delay period (1-72 hours,
    default 48)

-   NEW: Immediate irrigation shutoff when rain is detected

-   NEW: Scheduled runs skipped during active rain or rain delay

-   NEW: Rain Detected and Rain Delay Active binary sensors for
    Home Assistant

### **v1.3.1**

-   NEW: \"Expansion Boards Installed\" selector to configure expected
    board count (0, 1, 2, or 3)

-   NEW: Board mismatch error detection — orange LED when expected
    boards are missing

-   IMPROVED: Full status LED priority table with 9 distinct states

-   IMPROVED: I2C bus error only flagged when expansion boards are
    expected but nothing responds

-   IMPROVED: Detection script re-runs automatically when board count
    setting is changed

### **v1.3.0**

-   NEW: Valve overlap (3 seconds) for seamless zone transitions

-   NEW: Integrated pump management via sprinkler component

-   IMPROVED: Pump/master valve no longer cycles between zones

-   IMPROVED: Simplified internal code for better reliability

### **v1.2.x**

-   Initial Made for ESPHome compliant release

-   OTA updates via GitHub

-   Auto-advance preference memory

-   Zone 8 mode selection

─────────────────────────────────────────────────────

Support: https://github.com/FluxOpenHome/IrrigationController

*Flux Open Home Irrigation Controller - Made for ESPHome*
