# Frequently Asked Questions

## General

**What appliances are supported?**

The bundle supports GE Profile ovens (via `water_heater` entities), GE Profile front-load washers, and GE Profile dryers — all connected through the SmartHQ integration in Home Assistant. Top-load washers are not explicitly designed for, though they may work if they expose the same sensor entities.

**Do I need all three cards?**

No. You can install the bundle and only use the card types that match your appliances. Unused card types do not consume any resources at runtime — they are simply not instantiated.

**Can I use this with a non-GE washer or dryer?**

The card reads specific entity suffixes that the SmartHQ integration creates for GE appliances. If another integration exposes entities with the same naming convention, the card may work, but this is untested.

**What version of Home Assistant is required?**

Home Assistant 2023.1 or later is recommended. The cards use standard Lovelace custom element APIs that have been stable since 2022. Earlier versions may work but are not tested.

---

## Installation

**Do I need to add a resource manually?**

No. HACS automatically registers the JavaScript file as a Lovelace resource when you install through HACS. If you install manually, add the resource at Settings > Dashboards > Resources with type "JavaScript module" pointing to `/hacsfiles/ge-appliances-card/ge-appliances-card.js`.

**Can I install the bundle and individual cards at the same time?**

No. Each card type registers a global custom element name (`ge-oven-card`, `ge-washer-card`, `ge-dryer-card`). Custom element names can only be defined once per page session. Installing both will cause a conflict. Use either the bundle or the individual cards, not both.

**The card does not appear in the card picker after installation.**

Clear your browser cache and perform a hard reload (Ctrl+Shift+R or Cmd+Shift+R). If the card still does not appear, go to Settings > Dashboards > Resources and verify that `ge-appliances-card.js` is listed. If not, the HACS resource registration may have failed — add it manually.

---

## Oven Card

**Why does the oven window stay dark when the oven is on?**

The window activates based on the `state` field of the `water_heater` entity. If the state is "off" or "unavailable", the window stays dark. Check Developer Tools > States to verify what state your oven entity is reporting.

**The temperature shows "--" even though the oven is preheating.**

The oven card treats 100 F as a sensor floor value (not a real reading) and displays "--" in its place. GE ovens commonly report 100 F when the temperature sensor is not yet at a meaningful level. Once the oven reaches roughly 110 F, the real temperature will display.

**The probe section always shows "No" even though a probe is inserted.**

The probe detection relies on the `probe_present` attribute of the `water_heater` entity. Verify this attribute exists and is set to `true` in Developer Tools > States when the probe is physically inserted. This is a SmartHQ integration concern and not something the card can override.

**Can I use the oven card with a range or cooktop?**

The card is designed for wall ovens and ranges that expose a `water_heater` entity. Cooktop burners do not use the `water_heater` domain and are not supported.

**What does the light bulb icon at the top of the card mean?**

It represents the oven interior light. It brightens and glows yellow when the light is on. It is controlled by the `select.<id>_light` entity. The card displays the state but does not provide a button to toggle it — use a separate entities card or button card for control.

**Can I get a smaller card to fit a narrow column?**

Yes. Use the `size` option. Set it to `medium` for a shorter oven window, or `small` for an even more compact view. All three sizes show the same information in the LCD and attribute panel; only the oven window height changes.

---

## Washer Card

**How do I find my washer prefix?**

In Developer Tools > States, search for your washer. Look for entities that end in `_machine_state`. The prefix is everything before `_machine_state`. For example, if the entity is `sensor.hasvr1_ge_washer_laundry_machine_state`, the prefix is `sensor.hasvr1_ge_washer_laundry`.

**The drum is not spinning or agitating during a cycle.**

Drum animation requires the `machine_state` sensor to be something other than "Off". It also checks `sub_cycle` to determine spin vs. agitate. Verify both sensors are updating in Developer Tools > States during a cycle.

**The water level inside the drum never appears.**

The water level visualization shows during active non-spin phases. It relies on the machine being active (`machine_state` not "Off") and the sub-cycle not containing the word "spin". If your appliance reports a different state or sub-cycle string, the water level may not trigger.

**The dispenser tank shows the wrong status.**

The dispenser status is read directly from `sensor.<prefix>_washer_smart_dispense_tank_status`. The value is displayed as-is from the SmartHQ integration. If the value looks wrong, check the raw entity state in Developer Tools > States and confirm the SmartHQ integration is reporting it correctly.

**What does the prewash indicator "PRE" mean in the LCD?**

It means the washer is running a pre-wash cycle before the main wash. The indicator appears when `binary_sensor.<prefix>_washer_prewash` is "on".

**There is no spin animation even during the spin cycle.**

The animation checks whether `sub_cycle` contains the substring "spin" (case-insensitive). If your appliance reports the spin sub-cycle under a different name, the agitate animation will be used instead. This is a SmartHQ entity naming variation.

---

## Dryer Card

**How do I find my dryer prefix?**

In Developer Tools > States, search for your dryer. Look for entities ending in `_machine_state`. The prefix is everything before `_machine_state`. For example, `sensor.hasvr1_ge_dryer_laundry_machine_state` gives a prefix of `sensor.hasvr1_ge_dryer_laundry`.

**The dryer sheet count is not showing up.**

The sheet inventory reads `sensor.<prefix>_dryer_sheet_inventory`. Verify this entity exists and is updating. Also confirm that the `sheets` option is not set to `false` in your card config. If you set `sheets: false`, the WasherLink status is shown in that slot instead.

**What is WasherLink?**

WasherLink is a GE feature that allows a paired GE washer to automatically signal the dryer when a wash cycle ends. The card displays the linked/unlinked status of this feature. To show WasherLink status instead of dryer sheet inventory, set `sheets: false` in the card config.

**The vent warning is showing but I have already cleaned the vent.**

The blocked vent fault is reported by `binary_sensor.<prefix>_dryer_blocked_vent_fault`. After cleaning the vent, the fault may need to be cleared in the SmartHQ app or by running a short cycle. If the binary sensor remains "on" after clearing, this is a SmartHQ integration or appliance firmware issue.

**The steam animation is not appearing during a steam cycle.**

The steam effect activates when the cycle name or sub-cycle name contains the word "steam" (case-insensitive). If your appliance uses a different label for steam cycles, the animation will not trigger. The cycle and heat level will still display correctly.

**How do I show WasherLink instead of sheets?**

Add `sheets: false` to your card configuration:

```yaml
type: custom:ge-dryer-card
prefix: sensor.hasvr1_ge_dryer_laundry
name: Dryer
sheets: false
```

---

## Visual and Styling

**Can I change the card colors?**

The cards use hardcoded CSS within Shadow DOM and do not currently support theme overrides or custom color options. This is a known limitation of the current version.

**The card looks different on mobile.**

The cards use a fixed-width drum visualization (200 x 200 px) which may appear differently on small screens depending on your dashboard column layout. Using a single-column layout or setting the view to "panel" mode can improve appearance on phones.

**Can I remove the entity ID shown in the footer?**

The footer entity ID is part of the card's built-in layout and cannot be hidden through configuration options in the current version.

**The card background is white instead of dark.**

This can happen if the ha-card custom element is not being styled by Home Assistant's theme. Try a hard browser refresh. If the issue persists, check that your HA theme is loaded correctly.

---

## Updates

**How do I update the card?**

If installed via HACS, updates appear in the HACS Frontend section when a new release is published. Click "Update" and reload your browser.

**Will updates break my dashboard configuration?**

Card configuration options are kept stable between minor versions. Breaking changes, if any, will be noted in the release notes.
