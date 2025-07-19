# Solar EV Charging Blueprint for Home Assistant

A sophisticated Home Assistant blueprint that automatically manages EV charging based on available solar power. This blueprint dynamically adjusts charging rates to maximize solar usage while maintaining grid export buffers and includes advanced features like free power hour charging.

## Features

- **Dynamic Rate Adjustment**: Automatically adjusts charging rates based on real-time solar production
- **Smart Solar Calculation**: Handles both grid export and import scenarios intelligently
- **Free Power Hour**: Optional time-based charging at maximum rate (great for time-of-use tariffs)
- **Grace Periods**: Stability checking and import tolerance to prevent rapid switching
- **OCPP Integration**: Works with OCPP-compatible EV chargers
- **Comprehensive Logging**: Detailed debug information for troubleshooting
- **Fully Configurable**: All parameters can be customized without editing code

## Tested with
- Autel MaxichargerAC single phase 7.2kW
  - Charge Control Module V1.47.05
  - Power Control Module V1.19.00
  - Auto update turned off
  - No load balancing or power sharing set up
- Goodwe 6000N Inverter plus power meter
- OCPP (https://github.com/lbbrhzn/ocpp/blob/main/README.md)
  - 0.7.0

## How It Works

The automation continuously monitors:
- Solar power export/import levels
- Current EV charging rate
- Power stability (variance checking)
- Time-based conditions

It then:
1. **Starts charging** when sufficient solar power is available
2. **Adjusts rates** dynamically to match solar production
3. **Stops charging** when solar power drops below threshold
4. **Maintains buffer** to prevent grid import during charging
5. **Provides free power hour** for time-of-use optimization

## Prerequisites

### Required Integrations
- [OCPP Integration](https://github.com/lbbrhzn/ocpp) - For EV charger control
- [Solar/Energy Monitoring](https://www.home-assistant.io/integrations/energy/) - For power measurement

### Required Entities
You'll need the following entities in your Home Assistant setup:

1. **Charger Status Sensor** - Reports charger connector status
   - States: `Available`, `SuspendedEV`, `SuspendedEVSE`, `Charging`, `Preparing`, `Finishing`
   - Example: `sensor.charger_status_connector`

2. **Power Export/Import Sensor** - Reports current power flow
   - Positive values = exporting to grid
   - Negative values = importing from grid
   - Unit: Watts
   - Example: `sensor.active_power_l1`

3. **Charger Power Sensor** - Reports current charging power
   - Unit: kW (will be converted to Watts internally)
   - Example: `sensor.charger_power_active_import`

4. **Charge Control Switch** - Enable/disable automatic charging
   - Example: `switch.charger_charge_control`

5. **Transaction ID Sensor** - Current charging transaction ID
   - Example: `sensor.charger_transaction_id`

6. **Solar Elevation Sensor** - Sun elevation angle
   - Built-in: `sensor.sun_solar_elevation`

7. **Solar Variance Sensor** - Power export variance for stability
   - You may need to create this with a statistics sensor
   - Example: `sensor.solar_export_variance`

## Installation

### Method 1: Direct Import
[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://github.com/mavnz72/ha-blueprint-autel-ocpp/blob/main/Blueprint/EVSE_Solar_and_Free_Power_Charging_Controller.yaml)

### Method 2: Direct Download
1. Download the blueprint file: [`solar_ev_charging.yaml`](solar_ev_charging.yaml)
2. Copy it to your Home Assistant `blueprints/automation/` directory
3. Restart Home Assistant
4. Go to Settings → Automations & Scenes → Blueprints
5. The blueprint should appear in your list

### Method 3: Import URL
1. Go to Settings → Automations & Scenes → Blueprints
2. Click "Import Blueprint"
3. Paste this URL: `https://github.com/mavnz72/ha-blueprint-autel-ocpp/blob/main/Blueprint/EVSE_Solar_and_Free_Power_Charging_Controller.yaml`
4. Click "Preview" and then "Import"


## Configuration

### Creating the Automation
1. Go to Settings → Automations & Scenes
2. Click "Create Automation"
3. Select "Solar Excess EV Charging (Dynamic)" blueprint
4. Configure the required entities and parameters

### Entity Configuration
Map your actual entities to the blueprint inputs:

| Blueprint Input | Your Entity | Description |
|---|---|---|
| Charger Status Entity | `sensor.your_charger_status` | Charger connector status |
| Power Export Entity | `sensor.your_power_export` | Grid export/import power |
| Charger Power Entity | `sensor.your_charger_power` | Current charging power (kW) |
| Charge Control Switch | `switch.your_charge_control` | Enable/disable switch |
| Charger Transaction ID | `sensor.your_transaction_id` | Current transaction ID |
| Solar Elevation Entity | `sensor.sun_solar_elevation` | Sun elevation angle |
| Solar Variance Entity | `sensor.your_solar_variance` | Power export variance |

### Power Configuration
Adjust these values based on your setup:

| Parameter | Default | Description |
|---|---|---|
| Minimum EVSE Rate | 1400W | Minimum charging rate |
| Start Threshold | 1500W | Solar power needed to start charging |
| Stop Threshold | 300W | Stop charging below this solar level |
| Rate Increment | 100W | Charging rate adjustment steps |
| Target Buffer | 100W | Export buffer to maintain |
| Maximum EVSE Rate | 7400W | Maximum charging rate |

### Timing Configuration
| Parameter | Default | Description |
|---|---|---|
| Stability Grace Period | 120s | Wait for power stability |
| Import Grace Period | 360s | Tolerance for grid import |
| Variance Threshold | 500W | Power stability threshold |

### Free Power Hour
Configure time-based charging for time-of-use tariffs:
| Parameter | Default | Description |
|---|---|---|
| Enable Free Power Hour | Yes | Enable time-based charging |
| Start Time | 16:00 | Free power hour start |
| End Time | 17:00 | Free power hour end |
| Charging Rate | 32A | Charging rate during free hour |

## Creating Required Sensors

### Solar Export Variance Sensor
Add this to your `configuration.yaml`:

```yaml
sensor:
  - platform: statistics
    name: "Solar Export Variance"
    entity_id: sensor.active_power_l1  # Your power export sensor
    state_characteristic: variance
    max_age:
      minutes: 6
    sampling_size: 3
```


## Troubleshooting

### Common Issues

1. **Automation not triggering**
   - Check that all required entities exist and have valid states
   - Verify the charge control switch is enabled
   - Ensure sun elevation is above 0

2. **Charging not starting**
   - Check that available solar power exceeds start threshold
   - Verify charger status is in an available state
   - Check stability grace period hasn't expired

3. **Rapid rate changes**
   - Increase stability grace period
   - Adjust variance threshold
   - Check solar export variance sensor

4. **OCPP errors**
   - Verify OCPP integration is working
   - Check connection ID matches your charger
   - Ensure transaction ID sensor is valid

### Debug Information
The automation provides comprehensive logging. Check the logbook for entries from "Solar EV Charging" to see:
- Current power levels
- Charging decisions
- Rate adjustments
- Error conditions

### Log Examples
```
START CHARGING – Solar: 2500W, Starting at 1400W
ADJUST RATE – From 1400W to 2300W (Solar: 2500W, Target Buffer: 100W)
STOP CHARGING – Solar: 250W < 300W, Grace period expired
FREE POWER HOUR – Charging at 32A (16:00-17:00)
```

## Advanced Configuration

### Multiple Chargers
To use with multiple chargers, create separate automations with different:
- OCPP Connection IDs
- Entity mappings
- Rate limits

### Custom Charging Profiles
The blueprint uses OCPP custom profiles with:
- Profile ID: 1
- Stack Level: 2
- Purpose: TxProfile
- Kind: Relative

## Contributing

Found a bug or have a feature request? Please open an issue on GitHub!

### Development Setup
1. Fork the repository
2. Create a feature branch
3. Test with your Home Assistant setup
4. Submit a pull request

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- Home Assistant community for OCPP integration
- Solar EV charging enthusiasts for testing and feedback
- Contributors to the Home Assistant blueprints ecosystem

## Support

If you find this blueprint useful, please:
- ⭐ Star the repository
- 🐛 Report issues
- 💡 Suggest improvements
- 📢 Share with others

---

**Disclaimer**: This blueprint controls high-power electrical equipment. Use at your own risk and ensure your electrical installation meets local codes and safety standards.
