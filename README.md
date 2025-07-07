# EVSE Solar and Free Power Charging Controller For Autel MAxicharger - Complete Setup Guide

## Overview

This blueprint creates an intelligent EVSE (Electric Vehicle Supply Equipment) charging system that automatically:
- **Tracks charging sessions** and logs start/stop events
- **Optimizes solar charging** using excess solar power with configurable thresholds
- **Maximizes charging during free power hours** (configurable time period)
- **Prevents grid import** during solar charging with adjustable power buffer
- **Provides mobile notifications** for important events

## Prerequisites

### Required Hardware/Integrations
- **EVSE Charger** with OCPP (Open Charge Point Protocol) support
- **Home Assistant** with OCPP integration configured
- **Solar inverter** or energy monitoring system providing power data
- **Mobile device** with Home Assistant app for notifications

### Required Entities
Before using this blueprint, you need the following entities in your Home Assistant:

#### From Your EVSE/OCPP Integration:
- `sensor.charger_status_connector` - Reports connector status (Available, Charging, etc.)
- `sensor.charger_transaction_id` - Current charging transaction ID
- `sensor.charger_power_active_import` - Power consumption of the charger
- `switch.your_evse_switch` - Switch to turn EVSE on/off

#### From Your Energy System:
- `sensor.grid_power` - Grid import/export power (negative values = export)
- `sensor.sun_solar_elevation` - Sun elevation angle (usually available by default)

#### Helper Entities (Create These):
You must create these helper entities before using the blueprint:

1. **Input Boolean for Session Tracking:**
   ```yaml
   input_boolean:
     evse_charging_session_active:
       name: "EVSE Charging Session Active"
       icon: mdi:ev-station
   ```

2. **Input Boolean for Timed Charging:**
   ```yaml
   input_boolean:
     activate_evse_charge_timer:
       name: "Activate EVSE Charge Timer"
       icon: mdi:timer
   ```

3. **Input Number for Timer Hours:**
   ```yaml
   input_number:
     ev_timer:
       name: "EV Charge Timer Hours"
       min: 0.5
       max: 12
       step: 0.5
       unit_of_measurement: "hours"
       icon: mdi:timer
   ```

## Installation Instructions

### Step 1: Add Helper Entities

1. **Navigate to:** Settings → Devices & Services → Helpers
2. **Click:** "Create Helper"
3. **Create each helper** as specified above, or add them to your `configuration.yaml`

### Step 2: Install the Blueprint

1. **Create the blueprint file:**
   - Navigate to `/config/blueprints/automation/` (create the folder if it doesn't exist)
   - Create a new file named `evse_solar_free_power.yaml`
   - Copy the blueprint YAML content into this file

2. **Alternative method:**
   - Go to Settings → Automations & Scenes → Blueprints
   - Click "Import Blueprint"
   - Paste the blueprint URL or content

### Step 3: Create the Automation

1. **Go to:** Settings → Automations & Scenes → Automations
2. **Click:** "Create Automation"
3. **Select:** "Use Blueprint"
4. **Choose:** "EVSE Solar and Free Power Charging Controller"
5. **Configure all required fields** (see Configuration Guide below)

## Configuration Guide

### Device and Entity Configuration

| Field | Description | Example |
|-------|-------------|---------|
| **EVSE Device** | Your EVSE switch device | Select your EVSE device |
| **EVSE Switch Entity** | The switch entity for your EVSE | `switch.evse_charger` |
| **Charger Status Entity** | Connector status sensor | `sensor.charger_status_connector` |
| **Charger Transaction ID Entity** | Transaction ID sensor | `sensor.charger_transaction_id` |
| **Charger Power Entity** | Power consumption sensor | `sensor.charger_power_active_import` |
| **Grid Power Entity** | Grid import/export sensor | `sensor.grid_power_l1` |
| **Sun Elevation Entity** | Sun elevation sensor | `sensor.sun_solar_elevation` |
| **Charging Session Active Boolean** | Helper for session tracking | `input_boolean.evse_charging_session_active` |
| **Charge Timer Activation Boolean** | Helper for timed charging | `input_boolean.activate_evse_charge_timer` |
| **Charge Timer Hours** | Helper for timer duration | `input_number.ev_timer` |

### Notification Configuration

| Field | Description | Example |
|-------|-------------|---------|
| **Mobile Notification Service** | Your mobile app service | `mobile_app_your_phone` |

*Find your mobile app service name in Developer Tools → Services, search for "mobile_app"*

### Timing Configuration

| Field | Description | Default | Notes |
|-------|-------------|---------|-------|
| **Free Power Start Time** | When free power period begins | 16:00:00 | Adjust for your utility's free power hours |
| **Free Power End Time** | When free power period ends | 17:00:00 | System will notify and turn off EVSE |

### Solar Charging Configuration

| Field | Description | Default | Recommended Range |
|-------|-------------|---------|-------------------|
| **Solar Start Threshold (W)** | Minimum export to start charging | 1500W | 1200-2000W |
| **Solar Stop Threshold (W)** | Minimum export to continue charging | 1000W | 800-1500W |
| **Solar Buffer (W)** | Power buffer to prevent grid import | 100W | 50-200W |

**Important:** Start threshold should be higher than stop threshold to prevent rapid on/off cycling.

### Current and Power Settings

| Field | Description | Default | Notes |
|-------|-------------|---------|-------|
| **Maximum Charging Amps** | Maximum charging current | 32A | Don't exceed your EVSE's rating |
| **Minimum Charging Amps** | Minimum stable charging current | 6A | Most EVSEs require 6A minimum |
| **Voltage** | Nominal voltage for calculations | 230V | 230V (EU/AU) or 240V (US) |

## How It Works

### Solar Charging Logic

1. **Monitoring:** System checks grid power every 30 seconds during daylight hours
2. **Hysteresis:** Uses different thresholds for starting vs. stopping to prevent cycling
3. **Power Calculation:** 
   - Available power = Grid export + Current charger consumption
   - Usable power = Available power - Buffer
   - Target amps = Usable power ÷ Voltage
4. **Current Limiting:** Ensures current stays between minimum and maximum limits

### Free Power Hour Operation

1. **Auto Start:** At configured start time, turns on EVSE and sets maximum charging
2. **Session Tracking:** Activates session tracking
3. **Auto Stop:** At configured end time, sends notification and turns off EVSE

### Session Tracking

- **Automatic Detection:** Monitors charger status changes
- **Logging:** Records all session start/stop events
- **Integration:** Updates helper boolean for use in other automations

## Testing and Validation

### Initial Testing

1. **Verify Entity Names:** Check that all your entity names match the configuration
2. **Test Free Power Hour:** 
   - Set start time to 1-2 minutes in the future
   - Observe automation behavior
   - Check logs for any errors

3. **Test Solar Charging:**
   - Ensure you have solar export above your start threshold
   - Check that charging starts automatically
   - Verify current adjustments as solar conditions change

### Monitoring

**Check these locations for system activity:**
- **Logbook:** Settings → System → Logs → Logbook (search for "EVSE")
- **Automation Traces:** Settings → Automations & Scenes → [Your Automation] → Traces
- **Developer Tools:** Developer Tools → States (check your helper entities)

## Troubleshooting

### Common Issues

**Automation not triggering:**
- Check that all entity names are correct
- Verify your EVSE supports OCPP commands
- Ensure helper entities are created

**Solar charging not working:**
- Verify grid power sensor returns negative values for export
- Check sun elevation sensor is available
- Confirm charging session is active

**Free power hour not starting:**
- Verify EVSE is not already charging
- Check mobile notification service name
- Ensure times are in 24-hour format

**Current not adjusting properly:**
- Verify OCPP integration is working
- Check that transaction ID sensor has valid values
- Ensure your EVSE supports dynamic current limiting

### Debug Steps

1. **Enable automation traces:** Settings → Automations & Scenes → [Your Automation] → Enable Traces
2. **Check entity states:** Developer Tools → States
3. **Monitor logs:** Settings → System → Logs
4. **Test OCPP commands:** Developer Tools → Services → ocpp.set_charge_rate

## Advanced Customization

### Adjusting for Different Utility Rates

If you have multiple free power periods or different rates:
1. **Duplicate the automation** with different time settings
2. **Modify conditions** to handle multiple time periods
3. **Add additional time-based triggers** as needed

### Integration with Other Systems

**Home Energy Management:**
- Use the session active boolean in other automations
- Integrate with battery storage systems
- Coordinate with heat pump or other high-power loads

**Smart Home Integration:**
- Add voice announcements for charging events
- Integrate with presence detection
- Create dashboard cards showing charging status

### Performance Optimization

**Reduce Update Frequency:**
- Change time_pattern from 30 seconds to 1-2 minutes for less frequent updates
- Add conditions to only run when solar conditions are favorable

**Battery Protection:**
- Add conditions to pause charging if battery is full
- Integrate with vehicle API if available

## Maintenance

### Regular Checks

- **Monthly:** Review logbook entries for any errors
- **Seasonally:** Adjust solar thresholds based on changing conditions
- **Annually:** Review and update notification settings

### Updates

When updating the blueprint:
1. **Backup your configuration** before making changes
2. **Test in a safe environment** if possible
3. **Monitor carefully** after updates for any issues

## Support and Community

- **Home Assistant Community:** Share experiences and get help
- **GitHub Issues:** Report bugs or request features
- **Documentation:** Keep this guide updated with your customizations

## Legal and Safety Notes

- **Electrical Safety:** Ensure all installations comply with local electrical codes
- **EVSE Compatibility:** Verify your EVSE supports OCPP and dynamic current limiting
- **Grid Compliance:** Check local regulations regarding grid interaction and solar export
- **Insurance:** Verify that automated charging doesn't affect your insurance coverage

---

*This blueprint and documentation are provided as-is. Test thoroughly in your environment and ensure compliance with all local regulations and safety requirements.*
