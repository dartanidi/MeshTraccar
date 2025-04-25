# Meshtastic to Traccar Bridge for Home Assistant

An AppDaemon application for Home Assistant that bridges Meshtastic MQTT data to Traccar tracking server using the OsmAnd protocol. This bridge specifically focuses on filtering and forwarding only tracker devices from your Meshtastic mesh network.

## Overview

This application subscribes to MQTT topics containing Meshtastic position data and forwards it to a Traccar server. It intelligently identifies and processes only tracker devices, filtering out other nodes in your mesh network that aren't meant for location tracking.

## Features

- Intelligently identifies tracker devices in your Meshtastic mesh network
- Subscribes to position and device info MQTT topics
- Filters messages to only forward position data from tracker devices
- Multiple methods to identify trackers (role, name, explicit configuration)
- Handles various Meshtastic message formats
- Forwards position data to Traccar using the OsmAnd protocol
- Includes altitude, speed, and battery information when available
- Detailed logging for debugging

## Requirements

- Home Assistant with AppDaemon
- MQTT integration configured in Home Assistant
- Meshtastic network publishing data to MQTT
- Traccar server accessible from Home Assistant

## Installation

1. Clone this repository or download the `meshtraccar.py` file.
2. Copy the file to your AppDaemon apps directory (typically `/config/appdaemon/apps/`).
3. Add the configuration to your `apps.yaml` file as shown below.
4. Restart AppDaemon.

## Configuration

Add the following to your `apps.yaml` file:

```yaml
meshtraccar:
  module: meshtraccar
  class: MeshtasticToTraccar
  mqtt_topic: "meshtastic/+/json/position"  # Focus on position messages
  device_info_topic: "meshtastic/+/json/nodeinfo"  # Topic for node info messages
  traccar_url: "http://your-traccar-server:5055"  # Update with your Traccar server URL
  tracker_role: "TRACKER"  # Role that identifies tracker devices
  tracker_nodes:  # Explicitly list node IDs to treat as trackers
    - "!abcd1234"
    - "!efgh5678" 
  forward_all_positions: false  # Set to true to forward all devices' positions
  log_level: "INFO"  # Use "DEBUG" for more verbose logging
```

### Configuration Options

| Option | Default | Description |
|--------|---------|-------------|
| `mqtt_topic` | `meshtastic/+/json/position` | MQTT topic pattern to subscribe to for position data. |
| `device_info_topic` | `meshtastic/+/json/nodeinfo` | MQTT topic pattern to subscribe to for device information. |
| `traccar_url` | `http://your-traccar-server:5055` | URL of your Traccar server's OsmAnd endpoint. Typically port 5055. |
| `tracker_role` | `TRACKER` | The role value that identifies tracker devices in the mesh. |
| `tracker_nodes` | `[]` | List of node IDs to explicitly treat as trackers, regardless of their role. |
| `forward_all_positions` | `false` | When set to true, forward all position data regardless of device role. |
| `log_level` | `INFO` | Logging level. Options: DEBUG, INFO, WARNING, ERROR. |

## How It Works

1. The app subscribes to the position MQTT topic specified in the configuration.
2. It also subscribes to the device info topic to identify which nodes are trackers.
3. The app identifies trackers through three methods:
   - Devices with the specified `tracker_role` in their node info
   - Devices explicitly listed in the `tracker_nodes` configuration
   - Devices with names containing "tracker", "gps", or "location" indicators
4. When position data is received, it checks if it's from a known tracker device.
5. If it is a tracker, the position data is extracted and forwarded to Traccar.
6. Each tracker appears as a separate device in Traccar, identified by its node ID.

## Tracker Identification

The application uses several methods to identify tracker devices:

1. **Role-based identification**: Devices with the role matching `tracker_role` configuration
2. **Name-based identification**: Devices with names containing tracker-related terms
3. **Explicit configuration**: Devices listed in the `tracker_nodes` configuration
4. **Override option**: Setting `forward_all_positions` to true will forward all position data

This multi-layered approach ensures only relevant devices are forwarded to Traccar.

## Supported Meshtastic Message Formats

The app can parse position data from various message structures including:

- Direct lat/lon fields in the message
- Nested position objects
- Decoded payload structures
- Position data in various field naming conventions

## Traccar Configuration

In your Traccar server:

1. Make sure the OsmAnd protocol is enabled (it is by default).
2. Create devices with device identifiers matching your Meshtastic node IDs.
3. Check that port 5055 (default for OsmAnd) is accessible from your Home Assistant instance.

## Troubleshooting

If you're not seeing data in Traccar:

1. Set `log_level: "DEBUG"` in your configuration.
2. Check the AppDaemon logs for detailed information about received messages and forwarding attempts.
3. Verify that your Meshtastic nodes are publishing position data to MQTT.
4. Ensure the tracker identification is working correctly. Check logs for "Identified tracker node" messages.
5. Try setting `forward_all_positions: true` temporarily to see if any data is being received.
6. Ensure your Traccar server is accessible from Home Assistant.

Common issues:

- **No trackers identified**: Check if your devices have the correct role or try adding them to `tracker_nodes`.
- **Invalid position data warnings**: Some messages may not include position data.
- **JSON decode errors**: Indicates malformed messages in your MQTT topic.
- **HTTP errors when forwarding to Traccar**: Check connectivity and Traccar server configuration.

## Advanced Usage

### Using Custom Roles

If your Meshtastic network uses different roles than the default "TRACKER":

```yaml
meshtastic_to_traccar:
  module: meshtastic_to_traccar
  class: MeshtasticToTraccar
  tracker_role: "GPS_TRACKER"  # Your custom role
```

### Forwarding Specific Nodes Only

To only forward specific nodes regardless of their role:

```yaml
meshtastic_to_traccar:
  module: meshtastic_to_traccar
  class: MeshtasticToTraccar
  tracker_nodes:
    - "!abcd1234"  # Only this node will be forwarded
  forward_all_positions: false
```

### Forwarding All Positions

During testing or in smaller networks, you might want to forward all position data:

```yaml
meshtastic_to_traccar:
  module: meshtastic_to_traccar
  class: MeshtasticToTraccar
  forward_all_positions: true  # Forward all position data
```

## License

[MIT License](LICENSE)

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

Contributions are welcome! Please feel free to submit a Pull Request.
