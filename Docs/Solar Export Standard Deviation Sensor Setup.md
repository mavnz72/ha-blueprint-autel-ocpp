# Home Assistant Solar Export Standard Deviation Sensor Setup

This guide will help you create a statistics sensor in Home Assistant that monitors the standard deviation of your solar export power readings for use in blueprints.

## Configuration

### Step 1: Add the Sensor Configuration

Add the following configuration to your `configuration.yaml` file under the `sensor:` section:

```yaml
sensor:
  - platform: statistics
    name: Solar Export Standard Deviation
    entity_id: [your excess solar sensor]
    sampling_size: 3
    max_age:
      minutes: 6
    state_characteristic: standard_deviation
```

### Step 2: Configuration Parameters

- **`platform: statistics`** - Uses the statistics integration to calculate statistical values
- **`name: Solar Export Standard Deviation`** - The friendly name that will appear in Home Assistant
- **`entity_id: [your excess solar sensor]`** - The source sensor to monitor (your solar power sensor)
- **`sampling_size: 3`** - Number of recent samples to include in the calculation
- **`max_age: minutes: 6`** - Only consider samples from the last 6 minutes
- **`state_characteristic: standard_deviation`** - Specifies that the sensor should display the standard deviation value

### Step 3: Verify the Sensor

1. Navigate to **Settings** → **Devices & Services** → **Entities**
2. Search for "Solar Export Standard Deviation"
3. The entity ID will be: `sensor.solar_export_standard_deviation`

The sensor is now ready to be used in your blueprint configurations.
