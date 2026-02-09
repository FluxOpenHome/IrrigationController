# Smart Irrigation Controller Blueprint

A Home Assistant automation blueprint for the **Flux Open Home Irrigation Controller**. It reads the controller's existing schedule times, day-of-week switches, and schedule enabled switch directly — no need to duplicate scheduling configuration.

All optional features gracefully degrade when not configured.

## Features

- **Weather-aware skip logic** — Rain and freeze protection using forecast data
- **Moisture-based zone skipping** — Skip individual zones or full cycles based on soil moisture readings
- **Seasonal duration adjustment** — Manual multiplier, automatic temperature-based scaling, or both
- **Persistent notifications** — Alerts for skipped runs, cycle starts, and completions
- **Up to 10 moisture probes** — Each mapped to specific zones
- **Up to 4 schedule start times** — Reads directly from the controller

## Requirements

- [Flux Open Home Irrigation Controller](https://github.com/FluxOpenHome/IrrigationController) configured in Home Assistant
- (Recommended) A weather integration for forecast data — the [National Weather Service (NWS)](https://www.home-assistant.io/integrations/nws/) integration works well
- (Optional) Soil moisture sensors with `humidity` device class

## Installation

1. Copy `smart_irrigation.yaml` into your Home Assistant `blueprints/automation/` directory (or import via the source URL).
2. In Home Assistant, go to **Settings > Automations & Scenes > Blueprints**.
3. Click **Create Automation** from the Smart Irrigation Controller blueprint.
4. Configure the required and optional inputs described below.

## Configuration

### Required Inputs

| Input | Description |
|---|---|
| **Irrigation Main Switch** | The Start/Stop/Resume switch entity for your controller |
| **Auto Advance Switch** | The Auto Advance switch entity for your controller |
| **Schedule Enabled Switch** | The Schedule Enabled switch entity on the controller |
| **Schedule Start Times 1-4** | The schedule start time text entities from the controller |
| **Day-of-Week Switches** | Monday through Sunday schedule switches from the controller |

### Pre-Check Timing

| Input | Default | Description |
|---|---|---|
| **Pre-Check Minutes** | 5 min | How many minutes before a scheduled start time to evaluate conditions. Range: 2-30 minutes. |

### Weather (Optional)

| Input | Default | Description |
|---|---|---|
| **Weather Entity** | *(empty)* | Weather integration entity for forecast data. Leave empty to disable weather features. |
| **Rain Forecast Skip** | Enabled | Skip irrigation when rain probability exceeds the threshold. |
| **Rain Probability Threshold** | 50% | Rain probability percentage above which watering is skipped. |
| **Forecast Lookahead** | 24 hours | How many hours ahead to check the forecast. Range: 1-48 hours. |
| **Freeze Protection Skip** | Enabled | Skip irrigation when forecast temperature drops below the freeze threshold. |
| **Freeze Temperature Threshold** | 35°F | Temperature below which watering is skipped. |

### Moisture Sensors (Optional)

| Input | Default | Description |
|---|---|---|
| **Moisture-Based Zone Skipping** | Disabled | Master toggle — enable to use moisture probe data for zone skipping. |
| **Moisture Threshold** | 60% | Zones mapped to a probe reading above this value are skipped. |
| **Moisture Probes 1-10** | *(empty)* | Humidity sensor entities. Leave empty if not used. |
| **Zones for Probe 1-10** | *(empty)* | Comma-separated zone numbers covered by each probe (e.g., `1,2,3`). |
| **Zone Enable Switches** | *(empty)* | All zone enable switch entities in order (Zone 1 first, Zone 2 second, etc.). Required for moisture-based zone skipping. |

### Seasonal Adjustment (Optional)

| Input | Default | Description |
|---|---|---|
| **Seasonal Adjustment Mode** | None | `none`, `manual`, `auto_temperature`, or `both`. |
| **Manual Watering Multiplier** | 100% | Scale all zone durations. 150% = 50% longer, 50% = half duration. |
| **Temperature Baseline** | 80°F | For auto mode: durations increase ~1% per degree (F) above this baseline. |
| **Zone Duration Entities** | *(empty)* | Zone duration number entities in order. Required for seasonal adjustment. |

### Notifications

| Input | Default | Description |
|---|---|---|
| **Enable Notifications** | Enabled | Send persistent notifications for skipped runs, cycle starts, and completions. |

## How It Works

The automation runs on a per-minute time trigger and evaluates whether the current time falls within the pre-check window of any configured schedule start time.

### Decision Flow

```
Trigger (every minute)
  │
  ├─ Is schedule enabled? ──(no)──> Stop
  │
  ├─ Is today's day-of-week enabled? ──(no)──> Stop
  │
  ├─ Is current time within pre-check window? ──(no)──> Stop
  │
  ▼
Step 1: Initialize variables
  │
  ▼
Step 2: Weather forecast check (if weather entity configured)
  ├─ Rain probability > threshold? ──> Flag full skip
  ├─ Forecast low < freeze threshold? ──> Flag full skip
  │
  ▼
Step 3: Moisture check (if moisture skipping enabled)
  ├─ Per-probe: reading > moisture threshold? ──> Add mapped zones to wet list
  ├─ ALL mapped zones wet? ──> Flag full skip
  │
  ▼
Step 4: Skip decision
  ├─ Full skip flagged?
  │   ├─ Disable schedule on controller
  │   ├─ Send notification with reason
  │   ├─ Wait for scheduled time to pass
  │   ├─ Re-enable schedule
  │   └─ Stop
  │
  ▼
Step 5: Disable individual wet zones (partial moisture skip)
  │
  ▼
Step 6: Apply seasonal duration adjustment
  ├─ Save original durations
  ├─ Calculate multiplier (manual, temperature-based, or both)
  └─ Set adjusted durations on controller
  │
  ▼
Step 7: Wait for controller schedule to fire irrigation
  │
  ▼
Step 8: Wait for irrigation cycle to complete
  │
  ▼
Step 9: Post-run cleanup
  ├─ Restore original zone durations
  ├─ Re-enable any zones disabled for moisture
  └─ Send completion notification
```

### Without Moisture Probes

When no moisture probes are configured (the default), the automation is purely **schedule-driven** with **weather-based skip logic**. The moisture check is bypassed entirely because the `moisture_skip_enabled` toggle defaults to `false`. Irrigation runs on the controller's schedule unless rain or freeze conditions trigger a skip.

### Without a Weather Entity

When no weather entity is selected, the weather forecast check is skipped. The automation relies on the controller's schedule and any configured moisture probes. Seasonal temperature-based adjustment will use its baseline value as the forecast high, resulting in no adjustment (only the manual multiplier applies if configured).

## Automation Mode

The blueprint runs in `single` mode with `max_exceeded: silent`, meaning only one instance can run at a time. Overlapping triggers are silently ignored.

## Author

**Flux Open Home**

## Source

[GitHub - smart_irrigation.yaml](https://github.com/FluxOpenHome/IrrigationController/blob/main/blueprints/smart_irrigation.yaml)
