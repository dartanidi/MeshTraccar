# Meshtastic to Traccar Bridge

An AppDaemon application for Home Assistant that bridges Meshtastic MQTT data to Traccar tracking server using the OsmAnd protocol.

## Overview

This application subscribes to MQTT topics containing Meshtastic position data and forwards it to a Traccar server. It's designed to work with multiple trackers simultaneously, extracting position information from various Meshtastic message formats.

## Features

- Subscribes to all Meshtastic MQTT topics automatically
- Handles multiple trackers concurrently
- Extracts device IDs from MQTT topics and message payloads
- Parses various Meshtastic message formats
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
  mqtt_topic: "meshtastic/#"  # This wildcard catches all Meshtastic topics
  traccar_url: "http://your-traccar-server:5055"  # Update with your Traccar server URL
  log_level: "INFO"  # Use "DEBUG" for more verbose logging
```

### Configuration Options

| Option | Default | Description |
|--------|---------|-------------|
| `mqtt_topic` | `meshtastic/#` | MQTT topic pattern to subscribe to. The # wildcard captures all subtopics. |
| `traccar_url` | `http://your-traccar-server:5055` | URL of your Traccar server's OsmAnd endpoint. Typically port 5055. |
| `log_level` | `INFO` | Logging level. Options: DEBUG, INFO, WARNING, ERROR. |

## How It Works

1. The app subscribes to the MQTT topics specified in the configuration.
2. When a message is received, it attempts to extract position data (latitude, longitude, altitude, speed).
3. It identifies the device using:
   - First, the node ID from the MQTT topic (e.g., from `meshtastic/node123/json/position`)
   - If that fails, it looks for device identifiers in the message payload
4. The position data is forwarded to the Traccar server using the OsmAnd protocol.
5. Each tracker appears as a separate device in Traccar.

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
4. Ensure your Traccar server is accessible from Home Assistant.
5. Check that the device IDs in Traccar match the node IDs being extracted.

Common issues:

- **Invalid position data warnings**: The MQTT topic may contain messages that don't include position data (like telemetry-only messages).
- **JSON decode errors**: Indicates malformed messages in your MQTT topic.
- **HTTP errors when forwarding to Traccar**: Check connectivity and Traccar server configuration.

## License

[MIT License](LICENSE)

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.
