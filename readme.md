# Meshtastic to Traccar Bridge for Home Assistant

An AppDaemon application for Home Assistant that bridges Meshtastic MQTT data to Traccar tracking server using the OsmAnd protocol. This bridge intelligently identifies and forwards data only from GPS-enabled tracker devices in your Meshtastic mesh network.

## Overview

This application subscribes to MQTT topics containing Meshtastic data and forwards position information to a Traccar server. It intelligently identifies devices with GPS capabilities in your mesh network and only forwards location data from those devices.

## Features

- Intelligently identifies GPS-enabled devices in your Meshtastic mesh network
- Automatically detects trackers based on GPS capabilities and device names
- Subscribes to Meshtastic MQTT topics for position and device information
- Filters messages to only forward position data from actual tracking devices
- Multiple methods to identify trackers (GPS capabilities, names, explicit configuration)
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
  mqtt_topic: "meshtastic/+/json/#"  # Monitor all JSON messages
  position_topic_pattern: "position"  # Pattern for identifying position topics
  traccar_url: "http://your-traccar-server:5055"  # Update with your Traccar server URL
  tracker_nodes:  # Explicitly list node IDs to treat as trackers (optional)
    - "!abcd1234"
    - "!efgh5678" 
  forward_all_positions: false  # Set to true to forward all positions regardless of device type
  log_level: "INFO"  # Use "DEBUG" for more verbose logging
```

### Configuration Options

| Option | Default | Description |
|--------|---------|-------------|
| `mqtt_topic` | `meshtastic/+/json/#` | MQTT topic pattern to subscribe to for all Meshtastic JSON messages. |
| `position_topic_pattern` | `position` | String pattern to identify position-specific topics. |
| `traccar_url` | `http://your-traccar-server:5055` | URL of your Traccar server's OsmAnd endpoint. Typically port 5055. |
| `tracker_nodes` | `[]` | List of node IDs to explicitly treat as trackers, regardless of detected capabilities. |
| `forward_all_positions` | `false` | When set to true, forward all position data regardless of device type. |
| `log_level` | `INFO` | Logging level. Options: DEBUG, INFO, WARNING, ERROR. |

## How It Works

1. The app subscribes to the MQTT topic specified in the configuration, capturing all Meshtastic JSON messages.
2. It analyzes both position data and device information to identify devices with GPS capabilities.
3. The app identifies trackers through several methods:
   - Detecting GPS hardware capabilities in device information messages
   - Finding GPS configuration settings in device configuration
   - Recognizing tracker-related keywords in device names (like "tracker", "gps", etc.)
   - Checking the list of explicitly configured tracker nodes
4. When position data is received, it checks if it's from a known tracker device.
5. If it is a tracker, the position data is extracted and forwarded to Traccar.
6. Each tracker appears as a separate device in Traccar, identified by its node ID.

## Tracker Identification

The application uses several methods to identify tracker devices in your Meshtastic network:

1. **GPS capabilities detection**: Automatically detects devices that have GPS hardware or enabled GPS settings
2. **Name-based identification**: Identifies devices with names containing tracker-related terms
3. **Position data analysis**: Recognizes devices sending valid GPS coordinates
4. **Explicit configuration**: Uses devices listed in the `tracker_nodes` configuration
5. **Override option**: Setting `forward_all_positions` to true will forward all position data

This multi-layered approach ensures only relevant devices with actual GPS capabilities are forwarded to Traccar.

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
4. Ensure the tracker identification is working correctly. Check logs for "Identified node" messages.
5. Try setting `forward_all_positions: true` temporarily to see if any data is being received.
6. Ensure your Traccar server is accessible from Home Assistant.

Common issues:

- **No trackers identified**: Check if your devices have GPS capabilities or try adding them to `tracker_nodes`.
- **Invalid position data warnings**: Some messages may not include position data.
- **JSON decode errors**: Indicates malformed messages in your MQTT topic.
- **HTTP errors when forwarding to Traccar**: Check connectivity and Traccar server configuration.

## Advanced Usage

### Forwarding Specific Nodes Only

To only forward specific nodes regardless of their GPS capabilities:

```yaml
meshtraccar:
  module: meshtraccar
  class: MeshtasticToTraccar
  tracker_nodes:
    - "!abcd1234"  # Only this node will be forwarded
  forward_all_positions: false
```

### Forwarding All Positions

During testing or in smaller networks, you might want to forward all position data:

```yaml
meshtraccar:
  module: meshtraccar
  class: MeshtasticToTraccar
  forward_all_positions: true  # Forward all position data
```

### Custom Position Topic Pattern

If your Meshtastic setup uses a different naming pattern for position topics:

```yaml
meshtraccar:
  module: meshtraccar
  class: MeshtasticToTraccar
  position_topic_pattern: "gps"  # Your custom position topic pattern
```

## License

[MIT License](LICENSE)

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.
