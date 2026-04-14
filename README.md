# GE Appliances Card

A comprehensive set of custom Home Assistant Lovelace cards for monitoring GE Profile appliances connected via the SmartHQ integration. This bundle includes cards for ovens, washers, and dryers — all with a consistent dark theme, blue LCD displays, and appliance-specific visualizations.

![GE Appliances Card](https://raw.githubusercontent.com/ChrisCaho/ge-appliances-card/main/screenshot.png)

## Features

### GE Oven Card
- Dark-themed card styled to resemble a physical GE Profile oven control panel
- Blue LCD display showing current temperature in large digits, cooking mode, cook timer, kitchen timer, and probe temperature
- Animated oven window with heat element glow and pulsing orange light when the oven is active
- Oven door visualization with handle, window glass, and interior attribute grid
- Oven light indicator (illuminates when the oven light is on)
- Meat probe support — displays probe insertion status and live probe temperature
- Cook timer and kitchen timer display with formatted hours/minutes
- Attribute panel showing current temp, target temp, probe status, timers, and display state
- Three size variants: normal, medium, and small

### GE Washer Card
- Front-load washer visualization with animated circular drum door
- Water temperature color coding — cold cycles glow blue, warm cycles glow amber, hot cycles glow red/orange
- Drum animation: agitating paddles during wash cycles, full spin during spin cycles
- Water level visualization inside the drum glass during active non-spin phases
- Temperature glow ring that changes color based on the selected wash temperature
- LCD display showing active cycle, sub-cycle, time remaining, wash temperature, and pre-wash indicator
- Delay start countdown display when a delay is programmed
- Door status indicators for open and locked states
- Sensor grid: wash temperature, spin speed, soil level, rinse option, smart dispense tank status, and loads remaining

### GE Dryer Card
- Front-load dryer visualization with animated drum and wall-mounted lifter bars
- Heat level color coding — low heat glows blue/teal, medium glows amber, high glows orange/red
- Steam cycle detection with animated steam wisps inside the drum glass
- Blocked vent fault warning displayed prominently with a pulsing alert banner
- WasherLink status indicator (optional, replaces dryer sheets display)
- Dryer sheet inventory tracking (configurable)
- LCD display showing cycle, sub-cycle, time remaining, delay countdown, and heat setting
- Sensor grid: heat level, dryness level, dryer sheets or WasherLink, Eco Dry, Extended Tumble, and tumble/steam status

### Bundle
- All three cards in a single JavaScript file — one resource entry in Home Assistant
- Consistent visual design language across all three card types
- Dark background with blue LCD aesthetic matches GE Profile appliance styling

## Prerequisites

- Home Assistant 2023.1 or later
- HACS (Home Assistant Community Store)
- GE Profile appliances connected and reporting via the SmartHQ integration
- Relevant entities available in Home Assistant (see Entity Requirements below)

## Installation via HACS

1. Open HACS in your Home Assistant instance.
2. Click the three-dot menu in the top right and select "Custom repositories".
3. Add the repository URL: `https://github.com/ChrisCaho/ge-appliances-card`
4. Set the category to "Lovelace".
5. Click "Add".
6. Find "GE Appliances Card" in the HACS Frontend section and click "Download".
7. Reload your browser after installation.

The bundle registers a single Lovelace resource: `ge-appliances-card.js`. All three card types (`ge-oven-card`, `ge-washer-card`, `ge-dryer-card`) are available after loading this one file.

### Individual Cards

If you only need one or two card types, the cards are also available as separate HACS repositories:

- Oven only: `https://github.com/ChrisCaho/ge-oven-card`
- Washer only: `https://github.com/ChrisCaho/ge-washer-card`
- Dryer only: `https://github.com/ChrisCaho/ge-dryer-card`

Do not install both the bundle and an individual card — the custom element names will conflict.

## Configuration

### GE Oven Card

```yaml
type: custom:ge-oven-card
entity: water_heater.hasvr1_ge_top_oven
name: Top Oven
size: normal
```

| Option | Required | Default | Description |
|--------|----------|---------|-------------|
| `entity` | Yes | — | The `water_heater` entity ID for the oven |
| `name` | No | Friendly name from HA | Display name shown in the card header |
| `size` | No | `normal` | Card size variant: `normal`, `medium`, or `small` |

The card automatically derives related sensor entities from the `water_heater` entity ID by replacing the `water_heater.` domain prefix with `sensor.`. For example, `water_heater.hasvr1_ge_top_oven` maps to `sensor.hasvr1_ge_top_oven_cook_time_remaining`, `sensor.hasvr1_ge_top_oven_kitchen_timer`, and `sensor.hasvr1_ge_top_oven_probe_display_temp`. The oven light control is expected at `select.hasvr1_ge_top_oven_light`.

### GE Washer Card

```yaml
type: custom:ge-washer-card
prefix: sensor.hasvr1_ge_washer_laundry
name: Washer
```

| Option | Required | Default | Description |
|--------|----------|---------|-------------|
| `prefix` | Yes | — | The sensor entity prefix, without trailing slash (e.g. `sensor.hasvr1_ge_washer_laundry`) |
| `name` | No | `GE Washer` | Display name shown in the card header |

The card builds all entity IDs by appending suffixes to the prefix. Binary sensors are derived by replacing `sensor.` with `binary_sensor.` in the prefix.

### GE Dryer Card

```yaml
type: custom:ge-dryer-card
prefix: sensor.hasvr1_ge_dryer_laundry
name: Dryer
sheets: true
```

| Option | Required | Default | Description |
|--------|----------|---------|-------------|
| `prefix` | Yes | — | The sensor entity prefix (e.g. `sensor.hasvr1_ge_dryer_laundry`) |
| `name` | No | `GE Dryer` | Display name shown in the card header |
| `sheets` | No | `true` | Set to `false` to show WasherLink status instead of dryer sheet inventory |

## Complete Dashboard Example

The following example shows a full laundry and kitchen dashboard with three ovens, a washer, and a dryer arranged in a responsive grid.

```yaml
title: Appliances
views:
  - title: Kitchen & Laundry
    cards:
      - type: grid
        columns: 3
        square: false
        cards:
          - type: custom:ge-oven-card
            entity: water_heater.hasvr1_ge_top_oven
            name: Top Oven
            size: normal
          - type: custom:ge-oven-card
            entity: water_heater.hasvr1_ge_bottom_oven
            name: Bottom Oven
            size: normal
          - type: custom:ge-oven-card
            entity: water_heater.hasvr1_ge_advantium_oven
            name: Advantium
            size: normal
      - type: grid
        columns: 2
        square: false
        cards:
          - type: custom:ge-washer-card
            prefix: sensor.hasvr1_ge_washer_laundry
            name: Washer
          - type: custom:ge-dryer-card
            prefix: sensor.hasvr1_ge_dryer_laundry
            name: Dryer
            sheets: true
```

## Entity Requirements

### Oven

The oven card requires a `water_heater` entity. The following related entities are used automatically when present:

| Entity | Description |
|--------|-------------|
| `water_heater.<id>` | Main oven entity (required) |
| `sensor.<id>_cook_time_remaining` | Cook timer countdown in seconds |
| `sensor.<id>_kitchen_timer` | Kitchen timer countdown in seconds |
| `sensor.<id>_probe_display_temp` | Meat probe temperature in degrees F |
| `select.<id>_light` | Oven interior light control |

The oven card reads `operation_mode`, `current_temperature`, `temperature`, `display_temperature`, `display_state`, `probe_present`, `min_temp`, and `max_temp` from the `water_heater` entity attributes.

### Washer

All washer entities share the common prefix. The card reads the following suffixes:

| Suffix | Type | Description |
|--------|------|-------------|
| `_machine_state` | sensor | Overall machine state (Off, Running, etc.) |
| `_cycle` | sensor | Current cycle name |
| `_sub_cycle` | sensor | Current sub-cycle (Wash, Rinse, Spin, etc.) |
| `_time_remaining` | sensor | Time remaining in seconds |
| `_delay_time_remaining` | sensor | Delay start time remaining in seconds |
| `_washer_washtemp_level` | sensor | Wash temperature selection |
| `_washer_spintime_level` | sensor | Spin speed selection |
| `_washer_soil_level` | sensor | Soil level selection |
| `_washer_rinse_option` | sensor | Rinse option selection |
| `_washer_smart_dispense_loads_left` | sensor | Smart dispense loads remaining |
| `_washer_smart_dispense_tank_status` | sensor | Smart dispense tank status |
| `_door` | binary_sensor | Door open state |
| `_washer_door_lock` | binary_sensor | Door lock state |
| `_washer_prewash` | binary_sensor | Pre-wash active |
| `_remote_status` | binary_sensor | Remote start ready |

### Dryer

All dryer entities share the common prefix. The card reads the following suffixes:

| Suffix | Type | Description |
|--------|------|-------------|
| `_machine_state` | sensor | Overall machine state |
| `_cycle` | sensor | Current cycle name |
| `_sub_cycle` | sensor | Current sub-cycle |
| `_time_remaining` | sensor | Time remaining in seconds |
| `_delay_time_remaining` | sensor | Delay start time remaining in seconds |
| `_dryer_temperaturenew_option` | sensor | Heat level selection |
| `_dryer_drynessnew_level` | sensor | Dryness level selection |
| `_dryer_ecodry_option_selection` | sensor | Eco Dry setting |
| `_dryer_extended_tumble_option_selection` | sensor | Extended Tumble setting |
| `_dryer_sheet_inventory` | sensor | Dryer sheet count remaining |
| `_dryer_sheet_usage_configuration` | sensor | Sheet usage configuration |
| `_dryer_tumble_status` | sensor | Tumble status |
| `_door` | binary_sensor | Door open state |
| `_dryer_blocked_vent_fault` | binary_sensor | Blocked vent fault |
| `_dryer_washerlink_status` | binary_sensor | WasherLink paired status |

## Notification Automations

The following examples show how to set up completion and advance warning notifications for all three appliances. Adjust entity IDs to match your setup.

### Washer Finished

```yaml
alias: Washer Finished
trigger:
  - platform: state
    entity_id: sensor.hasvr1_ge_washer_laundry_machine_state
    to: "EndOfCycle"
action:
  - service: notify.mobile_app_your_phone
    data:
      title: Washer Done
      message: The washer has finished. Move clothes to the dryer.
```

### Dryer 5-Minute Warning

```yaml
alias: Dryer 5-Minute Warning
trigger:
  - platform: numeric_state
    entity_id: sensor.hasvr1_ge_dryer_laundry_time_remaining
    below: 301
    above: 0
condition:
  - condition: state
    entity_id: sensor.hasvr1_ge_dryer_laundry_machine_state
    state: "Running"
action:
  - service: notify.mobile_app_your_phone
    data:
      title: Dryer Almost Done
      message: The dryer has about 5 minutes remaining.
```

### Oven Preheat Complete

```yaml
alias: Oven Preheat Complete
trigger:
  - platform: state
    entity_id: water_heater.hasvr1_ge_top_oven
    attribute: display_state
    to: "PreheatComplete"
action:
  - service: notify.mobile_app_your_phone
    data:
      title: Oven Ready
      message: The top oven has reached its target temperature.
```

## Troubleshooting

**The card shows "Entity not found"**

Verify the entity ID you provided exists in Developer Tools > States. For the oven card, the entity must be in the `water_heater` domain. For washer and dryer cards, confirm the prefix matches the sensor entities visible in HA (check for a suffix like `_machine_state`).

**Sensor values show "--" even when the appliance is running**

The SmartHQ integration only reports values that the appliance actively sends. Some sensors may not update until a cycle starts or a specific feature is selected. Temperature sensors on ovens report 100 F as a floor value when idle — the card treats this as "no reading" and displays "--".

**The drum is not animating**

Animation requires `machine_state` to be anything other than "Off". Check that the machine state entity is updating correctly in Developer Tools > States.

**The oven window is dark even when the oven is on**

The window activates when the oven state is not "off" or "unavailable". If `water_heater` state is showing "on" but the window is dark, verify that `state.toLowerCase()` is not returning "off". Check Developer Tools > States for the exact state string.

**Washer temperature color is grey**

The temperature color map covers common SmartHQ temperature labels. If your appliance reports a temperature string not in the map (e.g. a localized or custom value), the fallback grey color will be used. The raw value will still display correctly in the sensor grid.

**Dryer vent warning is always showing**

The vent warning appears when `binary_sensor.<prefix>_dryer_blocked_vent_fault` is in the "on" state. If this sensor is stuck on, clean the lint screen, exhaust duct, and external vent, then check whether the fault clears in the SmartHQ app. If the entity is reporting incorrectly, you can ignore or suppress it in HA.

**I installed the bundle and an individual card and now I get a custom element conflict**

Each card type (ge-oven-card, ge-washer-card, ge-dryer-card) can only be registered once per browser session. Remove either the bundle resource or all individual card resources from Settings > Dashboards > Resources, then reload.

**The card is not picking up my entity prefix correctly**

Remove any trailing slash from the prefix. The card strips trailing slashes automatically, but double-check that the prefix matches the entity ID exactly up to (but not including) the first underscore-separated suffix.

## License

This project is released under the Unlicense. It is dedicated to the public domain. See the LICENSE file for details.
