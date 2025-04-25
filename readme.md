# Meshtastic To Traccar for Home Assistant

An AppDaemon integration for Home Assistant that bridges Meshtastic mesh network devices with Traccar GPS tracking server.

## Overview

MeshTraccar subscribes to MQTT messages from Meshtastic devices and forwards position data to a Traccar server using the OsmAnd protocol. This integration enables you to visualize and track Meshtastic nodes on your Traccar server's map interface.

Key features:
- Connects Meshtastic mesh network devices to Traccar GPS tracking platforms
- Configurable tracking for specific Meshtastic node IDs
- Processes position data via MQTT and forwards to Traccar
- Simple setup within Home Assistant's AppDaemon environment

## Requirements

- Home Assistant with AppDaemon installed
- MQTT broker configured in Home Assistant
- Meshtastic devices publishing data to MQTT
- Traccar server with OsmAnd protocol enabled

## Installation

1. Add this file to your AppDaemon apps directory (typically `/config/appdaemon/apps/`)
2. Configure the app in your `apps.yaml` file as shown below
3. Restart AppDaemon

## Configuration

Add the following to your AppDaemon `apps.yaml` file:

```yaml
meshtraccar:
  module: meshtraccar
  class: MeshtasticToTraccar
  mqtt_topic: "meshtastic/+/json/#"
  position_topic_pattern: "position"
  traccar_url: "http://your-traccar-server:5055"
  log_level: "INFO"
  tracker_nodes:
    - "!abcd1234"  # Node ID of your first Meshtastic device
    - "!efgh5678"  # Node ID of your second Meshtastic device
```

### Configuration Options

| Option | Description | Default |
| ------ | ----------- | ------- |
| `mqtt_topic` | MQTT topic to subscribe to for Meshtastic messages | `meshtastic/+/json/#` |
| `position_topic_pattern` | Pattern to identify position topics | `position` |
| `traccar_url` | URL of your Traccar server (required) | None |
| `log_level` | Logging level (DEBUG, INFO, WARNING, ERROR) | `INFO` |
| `tracker_nodes` | List of Meshtastic node IDs to track (required) | None |

## How It Works

1. The app subscribes to the configured MQTT topic where Meshtastic devices publish their data
2. When a position message is received from a configured tracker node, the app:
   - Validates that the message contains valid position data
   - Extracts coordinates and other relevant information
   - Formats the data according to the OsmAnd protocol
   - Forwards the data to the configured Traccar server

## Limitations

- Only processes position data in standard formats (top-level lat/lon or latitude/longitude fields)
- Does not automatically detect which devices have GPS capabilities; you must explicitly configure which nodes to track
- Requires proper Meshtastic configuration to publish position data to MQTT

## Troubleshooting

If you're not seeing data in Traccar:

1. Check that your Meshtastic devices are publishing position data to MQTT
2. Verify that the node IDs in your configuration match your Meshtastic devices
3. Ensure your Traccar server is properly configured to accept OsmAnd protocol data
4. Check AppDaemon logs for any errors or warnings from this integration

## License

This project is licensed under the MIT License - see the LICENSE file for details.
