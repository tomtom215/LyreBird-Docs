# Custom Integration

Comprehensive guide to integrating LyreBirdAudio with external systems, automation workflows, and custom applications.

---

## Overview

LyreBirdAudio provides multiple integration points for extending functionality and connecting with other systems. Whether you're building custom dashboards, integrating with automation platforms, or creating specialized workflows, LyreBirdAudio's open architecture enables seamless integration.

This guide covers:

- MediaMTX HTTP API integration
- Custom FFmpeg pipeline creation
- External monitoring system integration
- Scripting and automation
- Webhook and event handling
- Third-party tool integration

---

## MediaMTX HTTP API Integration

### API Overview

MediaMTX provides a comprehensive HTTP API for stream management and monitoring:

**Base URL:** `http://localhost:9997/v3/`

**Authentication:** None (local access), can be configured for authentication

**Format:** JSON request/response

### Available Endpoints

**List All Streams:**

```bash
GET http://localhost:9997/v3/paths/list
```

**Response:**

```json
{
  "itemCount": 2,
  "pageCount": 1,
  "items": [
    {
      "name": "Blue_Yeti",
      "confName": "all",
      "source": {
        "type": "rtspSession"
      },
      "ready": true,
      "readyTime": "2025-11-15T10:00:00Z",
      "tracks": ["G722"],
      "bytesReceived": 1048576,
      "bytesSent": 2097152,
      "readers": [
        {
          "type": "rtspConn",
          "id": "abc123"
        }
      ]
    },
    {
      "name": "USB_Microphone",
      "confName": "all",
      "source": {
        "type": "rtspSession"
      },
      "ready": true,
      "readyTime": "2025-11-15T10:00:01Z",
      "tracks": ["opus"],
      "bytesReceived": 987654,
      "bytesSent": 1975308,
      "readers": []
    }
  ]
}
```

**Get Specific Stream:**

```bash
GET http://localhost:9997/v3/paths/get/Blue_Yeti
```

**Response:**

```json
{
  "name": "Blue_Yeti",
  "confName": "all",
  "source": {
    "type": "rtspSession",
    "id": "session-id-123"
  },
  "ready": true,
  "readyTime": "2025-11-15T10:00:00Z",
  "tracks": ["opus"],
  "bytesReceived": 1048576,
  "bytesSent": 2097152,
  "readers": [
    {
      "type": "rtspConn",
      "id": "client-123",
      "created": "2025-11-15T10:05:00Z"
    }
  ]
}
```

### Python Integration Example

```python
#!/usr/bin/env python3
import requests
import json

class LyreBirdAPI:
    def __init__(self, base_url="http://localhost:9997/v3"):
        self.base_url = base_url

    def list_streams(self):
        """Get list of all active streams"""
        response = requests.get(f"{self.base_url}/paths/list")
        return response.json()

    def get_stream(self, stream_name):
        """Get details for specific stream"""
        response = requests.get(f"{self.base_url}/paths/get/{stream_name}")
        return response.json()

    def get_stream_stats(self, stream_name):
        """Get statistics for a stream"""
        stream = self.get_stream(stream_name)
        return {
            "name": stream["name"],
            "ready": stream["ready"],
            "bytes_received": stream["bytesReceived"],
            "bytes_sent": stream["bytesSent"],
            "num_readers": len(stream.get("readers", []))
        }

    def monitor_all_streams(self):
        """Monitor all streams and return health status"""
        streams = self.list_streams()
        results = []

        for stream in streams["items"]:
            results.append({
                "name": stream["name"],
                "status": "HEALTHY" if stream["ready"] else "FAILED",
                "clients": len(stream.get("readers", []))
            })

        return results

# Usage example
if __name__ == "__main__":
    api = LyreBirdAPI()

    # List all streams
    streams = api.list_streams()
    print(f"Active streams: {streams['itemCount']}")

    # Get stream stats
    stats = api.get_stream_stats("Blue_Yeti")
    print(f"Stream: {stats['name']}")
    print(f"Status: {'READY' if stats['ready'] else 'NOT READY'}")
    print(f"Clients: {stats['num_readers']}")

    # Monitor all streams
    health = api.monitor_all_streams()
    for stream in health:
        print(f"{stream['name']}: {stream['status']} ({stream['clients']} clients)")
```

### Node.js Integration Example

```javascript
// lyrebird-api.js
const axios = require('axios');

class LyreBirdAPI {
    constructor(baseUrl = 'http://localhost:9997/v3') {
        this.baseUrl = baseUrl;
    }

    async listStreams() {
        const response = await axios.get(`${this.baseUrl}/paths/list`);
        return response.data;
    }

    async getStream(streamName) {
        const response = await axios.get(`${this.baseUrl}/paths/get/${streamName}`);
        return response.data;
    }

    async getStreamHealth(streamName) {
        try {
            const stream = await this.getStream(streamName);
            return {
                name: stream.name,
                healthy: stream.ready,
                uptime: this.calculateUptime(stream.readyTime),
                clients: stream.readers ? stream.readers.length : 0
            };
        } catch (error) {
            return {
                name: streamName,
                healthy: false,
                error: error.message
            };
        }
    }

    calculateUptime(readyTime) {
        const start = new Date(readyTime);
        const now = new Date();
        const uptimeMs = now - start;
        const uptimeMinutes = Math.floor(uptimeMs / 60000);
        return `${uptimeMinutes} minutes`;
    }

    async monitorAllStreams() {
        const streams = await this.listStreams();
        const healthChecks = streams.items.map(stream =>
            this.getStreamHealth(stream.name)
        );
        return Promise.all(healthChecks);
    }
}

// Usage
async function main() {
    const api = new LyreBirdAPI();

    // Monitor all streams
    const health = await api.monitorAllStreams();
    console.log('Stream Health:');
    health.forEach(stream => {
        console.log(`${stream.name}: ${stream.healthy ? 'HEALTHY' : 'FAILED'}`);
        console.log(`  Clients: ${stream.clients}`);
        console.log(`  Uptime: ${stream.uptime}`);
    });
}

main();
```

---

## Custom FFmpeg Pipelines

### Advanced FFmpeg Configuration

Create custom audio processing pipelines beyond standard configuration:

**Audio Filtering Example:**

```bash
#!/bin/bash
# Custom FFmpeg pipeline with audio processing

DEVICE="Blue_Yeti"
RTSP_URL="rtsp://localhost:8554/Blue_Yeti_Processed"

ffmpeg \
    -f alsa -i hw:CARD=$DEVICE \
    -af "highpass=f=80,lowpass=f=18000,compand,volume=1.5" \
    -c:a libopus \
    -b:a 128k \
    -ar 48000 \
    -ac 1 \
    -f rtsp \
    $RTSP_URL
```

**Noise Reduction:**

```bash
#!/bin/bash
# Apply noise reduction to audio stream

DEVICE="USB_Microphone"
RTSP_URL="rtsp://localhost:8554/USB_Microphone_Clean"

ffmpeg \
    -f alsa -i hw:CARD=$DEVICE \
    -af "arnndn=m=/path/to/noise-model.rnnn" \
    -c:a libopus \
    -b:a 128k \
    -ar 48000 \
    -f rtsp \
    $RTSP_URL
```

**Multi-Band Compression:**

```bash
#!/bin/bash
# Professional audio compression

DEVICE="Studio_Mic"
RTSP_URL="rtsp://localhost:8554/Studio_Mic_Mastered"

ffmpeg \
    -f alsa -i hw:CARD=$DEVICE \
    -af "acompressor=threshold=-20dB:ratio=4:attack=5:release=50,\
         equalizer=f=100:width_type=o:width=1:g=-2,\
         equalizer=f=5000:width_type=o:width=1:g=2" \
    -c:a aac \
    -b:a 192k \
    -ar 48000 \
    -ac 2 \
    -f rtsp \
    $RTSP_URL
```

### Stream Recording Integration

Record streams automatically:

```bash
#!/bin/bash
# /usr/local/bin/lyrebird-record-stream.sh

STREAM_NAME="$1"
OUTPUT_DIR="/var/recordings/lyrebird"
TIMESTAMP=$(date '+%Y%m%d_%H%M%S')
OUTPUT_FILE="$OUTPUT_DIR/${STREAM_NAME}_${TIMESTAMP}.opus"

mkdir -p "$OUTPUT_DIR"

# Record stream to file
ffmpeg \
    -i "rtsp://localhost:8554/$STREAM_NAME" \
    -c:a copy \
    -f opus \
    "$OUTPUT_FILE"
```

**Scheduled Recording:**

```bash
# Cron job to record streams during business hours
# /etc/cron.d/lyrebird-recording

# Record Blue_Yeti from 9 AM to 5 PM weekdays
0 9 * * 1-5 /usr/local/bin/lyrebird-record-stream.sh Blue_Yeti
0 17 * * 1-5 pkill -f "ffmpeg.*Blue_Yeti"
```

---

## External Monitoring Integration

### Prometheus Metrics Exporter

Complete metrics exporter for Prometheus:

```python
#!/usr/bin/env python3
# /usr/local/bin/lyrebird-prometheus-exporter.py

from prometheus_client import start_http_server, Gauge, Info
import requests
import time
import psutil

# Define metrics
streams_active = Gauge('lyrebird_streams_active', 'Number of active streams')
stream_ready = Gauge('lyrebird_stream_ready', 'Stream ready status', ['stream'])
stream_clients = Gauge('lyrebird_stream_clients', 'Number of connected clients', ['stream'])
stream_bytes_rx = Gauge('lyrebird_stream_bytes_received', 'Bytes received', ['stream'])
stream_bytes_tx = Gauge('lyrebird_stream_bytes_sent', 'Bytes sent', ['stream'])
cpu_usage = Gauge('lyrebird_cpu_usage_percent', 'CPU usage percentage')
memory_usage = Gauge('lyrebird_memory_usage_bytes', 'Memory usage in bytes')
system_info = Info('lyrebird_system', 'System information')

def collect_metrics():
    """Collect metrics from MediaMTX API and system"""

    try:
        # Get stream list
        response = requests.get('http://localhost:9997/v3/paths/list', timeout=5)
        data = response.json()

        # Update stream count
        streams_active.set(data['itemCount'])

        # Update per-stream metrics
        for stream in data['items']:
            name = stream['name']
            stream_ready.labels(stream=name).set(1 if stream['ready'] else 0)
            stream_clients.labels(stream=name).set(len(stream.get('readers', [])))
            stream_bytes_rx.labels(stream=name).set(stream.get('bytesReceived', 0))
            stream_bytes_tx.labels(stream=name).set(stream.get('bytesSent', 0))

        # System metrics
        cpu_usage.set(psutil.cpu_percent(interval=1))
        memory_usage.set(psutil.virtual_memory().used)

        # System info
        system_info.info({
            'hostname': psutil.os.uname().nodename,
            'platform': psutil.os.uname().system,
            'version': '1.0.0'
        })

    except Exception as e:
        print(f"Error collecting metrics: {e}")

if __name__ == '__main__':
    # Start HTTP server on port 9100
    start_http_server(9100)
    print("Prometheus exporter running on :9100")

    # Collect metrics every 15 seconds
    while True:
        collect_metrics()
        time.sleep(15)
```

**Systemd Service:**

```ini
# /etc/systemd/system/lyrebird-prometheus-exporter.service

[Unit]
Description=LyreBird Prometheus Exporter
After=network.target mediamtx.service

[Service]
Type=simple
User=lyrebird
ExecStart=/usr/bin/python3 /usr/local/bin/lyrebird-prometheus-exporter.py
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

### InfluxDB Integration

Send metrics to InfluxDB:

```python
#!/usr/bin/env python3
# /usr/local/bin/lyrebird-influxdb-reporter.py

from influxdb_client import InfluxDBClient, Point
from influxdb_client.client.write_api import SYNCHRONOUS
import requests
import time

# InfluxDB configuration
INFLUX_URL = "http://localhost:8086"
INFLUX_TOKEN = "your-token"
INFLUX_ORG = "lyrebird"
INFLUX_BUCKET = "audio-streams"

def collect_and_send():
    """Collect metrics and send to InfluxDB"""

    client = InfluxDBClient(url=INFLUX_URL, token=INFLUX_TOKEN, org=INFLUX_ORG)
    write_api = client.write_api(write_options=SYNCHRONOUS)

    try:
        # Get stream data
        response = requests.get('http://localhost:9997/v3/paths/list')
        data = response.json()

        # Write metrics
        for stream in data['items']:
            point = Point("stream_metrics") \
                .tag("stream", stream['name']) \
                .field("ready", 1 if stream['ready'] else 0) \
                .field("clients", len(stream.get('readers', []))) \
                .field("bytes_received", stream.get('bytesReceived', 0)) \
                .field("bytes_sent", stream.get('bytesSent', 0))

            write_api.write(bucket=INFLUX_BUCKET, record=point)

        print(f"Sent metrics for {len(data['items'])} streams")

    except Exception as e:
        print(f"Error: {e}")
    finally:
        client.close()

if __name__ == '__main__':
    while True:
        collect_and_send()
        time.sleep(60)  # Every minute
```

---

## Scripting and Automation

### Automated Stream Rotation

Rotate between multiple devices:

```bash
#!/bin/bash
# /usr/local/bin/lyrebird-stream-rotation.sh

DEVICES=("Mic_Room1" "Mic_Room2" "Mic_Room3")
ACTIVE_STREAM="Conference_Room"
ROTATION_INTERVAL=300  # 5 minutes

while true; do
    for DEVICE in "${DEVICES[@]}"; do
        echo "Switching to $DEVICE"

        # Stop current stream
        sudo pkill -f "ffmpeg.*$ACTIVE_STREAM"

        # Start new stream from device
        ffmpeg -f alsa -i hw:CARD=$DEVICE \
            -c:a libopus -b:a 128k -ar 48000 \
            -f rtsp rtsp://localhost:8554/$ACTIVE_STREAM &

        # Wait for rotation interval
        sleep $ROTATION_INTERVAL
    done
done
```

### Event-Driven Automation

Trigger actions based on events:

```bash
#!/bin/bash
# /usr/local/bin/lyrebird-event-handler.sh

# Monitor MediaMTX logs for events
tail -F /var/log/mediamtx.out | while read line; do

    # New client connected
    if echo "$line" | grep -q "reader opened"; then
        STREAM=$(echo "$line" | grep -oP '(?<=path )\w+')
        echo "Client connected to $STREAM"

        # Trigger action (e.g., send notification)
        curl -X POST https://hooks.slack.com/services/YOUR/WEBHOOK/URL \
            -H 'Content-Type: application/json' \
            -d "{\"text\": \"New client connected to stream: $STREAM\"}"
    fi

    # Stream failed
    if echo "$line" | grep -q "error"; then
        STREAM=$(echo "$line" | grep -oP '(?<=path )\w+')
        echo "Stream $STREAM encountered error"

        # Send alert
        mail -s "LyreBird Alert: Stream Error" admin@example.com <<< \
            "Stream $STREAM encountered an error. Check logs for details."
    fi
done
```

### Health Check Integration

Create comprehensive health check script:

```bash
#!/bin/bash
# /usr/local/bin/lyrebird-health-check.sh

# Returns: 0 = healthy, 1 = warning, 2 = critical

CRITICAL=0
WARNINGS=0

# Check MediaMTX service
if ! systemctl is-active --quiet mediamtx; then
    echo "CRITICAL: MediaMTX service not running"
    CRITICAL=1
fi

# Check stream count
EXPECTED_STREAMS=2
ACTUAL_STREAMS=$(curl -s http://localhost:9997/v3/paths/list | jq '.itemCount')

if [ "$ACTUAL_STREAMS" -lt "$EXPECTED_STREAMS" ]; then
    echo "WARNING: Expected $EXPECTED_STREAMS streams, found $ACTUAL_STREAMS"
    WARNINGS=1
fi

# Check CPU usage
CPU_USAGE=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)
if (( $(echo "$CPU_USAGE > 80" | bc -l) )); then
    echo "WARNING: High CPU usage: $CPU_USAGE%"
    WARNINGS=1
fi

# Check memory usage
MEM_FREE=$(free -m | awk 'NR==2{print $7}')
if [ "$MEM_FREE" -lt 200 ]; then
    echo "WARNING: Low memory: ${MEM_FREE}MB free"
    WARNINGS=1
fi

# Return appropriate exit code
if [ $CRITICAL -eq 1 ]; then
    exit 2
elif [ $WARNINGS -eq 1 ]; then
    exit 1
else
    echo "OK: All checks passed"
    exit 0
fi
```

---

## Webhook Integration

### Stream Event Webhooks

Send webhooks on stream events:

```python
#!/usr/bin/env python3
# /usr/local/bin/lyrebird-webhook-sender.py

import requests
import json
import time
from datetime import datetime

WEBHOOK_URL = "https://your-webhook-endpoint.com/lyrebird"

class StreamMonitor:
    def __init__(self):
        self.known_streams = {}

    def check_streams(self):
        """Check current streams and detect changes"""
        try:
            response = requests.get('http://localhost:9997/v3/paths/list')
            data = response.json()

            current_streams = {s['name']: s for s in data['items']}

            # Detect new streams
            for name, stream in current_streams.items():
                if name not in self.known_streams:
                    self.send_webhook('stream_started', {
                        'stream': name,
                        'timestamp': datetime.utcnow().isoformat()
                    })

            # Detect removed streams
            for name in self.known_streams:
                if name not in current_streams:
                    self.send_webhook('stream_stopped', {
                        'stream': name,
                        'timestamp': datetime.utcnow().isoformat()
                    })

            # Detect client connections
            for name, stream in current_streams.items():
                old_clients = len(self.known_streams.get(name, {}).get('readers', []))
                new_clients = len(stream.get('readers', []))

                if new_clients > old_clients:
                    self.send_webhook('client_connected', {
                        'stream': name,
                        'clients': new_clients,
                        'timestamp': datetime.utcnow().isoformat()
                    })
                elif new_clients < old_clients:
                    self.send_webhook('client_disconnected', {
                        'stream': name,
                        'clients': new_clients,
                        'timestamp': datetime.utcnow().isoformat()
                    })

            self.known_streams = current_streams

        except Exception as e:
            print(f"Error checking streams: {e}")

    def send_webhook(self, event_type, data):
        """Send webhook notification"""
        payload = {
            'event': event_type,
            'data': data,
            'source': 'lyrebird-audio'
        }

        try:
            response = requests.post(
                WEBHOOK_URL,
                json=payload,
                headers={'Content-Type': 'application/json'},
                timeout=5
            )
            print(f"Webhook sent: {event_type} - {response.status_code}")
        except Exception as e:
            print(f"Error sending webhook: {e}")

if __name__ == '__main__':
    monitor = StreamMonitor()

    while True:
        monitor.check_streams()
        time.sleep(10)  # Check every 10 seconds
```

---

## Third-Party Tool Integration

### OBS Studio Integration

Integrate with OBS for streaming/recording:

```bash
#!/bin/bash
# /usr/local/bin/lyrebird-obs-setup.sh

# Add LyreBird streams as OBS media sources

STREAM_URL="rtsp://localhost:8554/Blue_Yeti"
OBS_CONFIG="$HOME/.config/obs-studio/basic/scenes/"

# Create OBS scene with LyreBird audio
cat > /tmp/lyrebird-scene.json <<EOF
{
  "sources": [
    {
      "name": "LyreBird_Audio",
      "type": "ffmpeg_source",
      "settings": {
        "is_local_file": false,
        "input": "$STREAM_URL",
        "input_format": "rtsp"
      }
    }
  ]
}
EOF

echo "Import /tmp/lyrebird-scene.json into OBS Studio"
```

### VLC Integration Script

Create VLC playlists from active streams:

```bash
#!/bin/bash
# /usr/local/bin/lyrebird-vlc-playlist.sh

OUTPUT_FILE="$HOME/lyrebird-streams.m3u"

# Get active streams
STREAMS=$(curl -s http://localhost:9997/v3/paths/list | \
          jq -r '.items[].name')

# Create M3U playlist
echo "#EXTM3U" > "$OUTPUT_FILE"

for STREAM in $STREAMS; do
    echo "#EXTINF:-1,$STREAM" >> "$OUTPUT_FILE"
    echo "rtsp://localhost:8554/$STREAM" >> "$OUTPUT_FILE"
done

echo "Playlist created: $OUTPUT_FILE"
vlc "$OUTPUT_FILE"
```

### Home Assistant Integration

YAML configuration for Home Assistant:

```yaml
# configuration.yaml

sensor:
  - platform: rest
    name: LyreBird Streams
    resource: http://localhost:9997/v3/paths/list
    value_template: '{{ value_json.itemCount }}'
    json_attributes:
      - items
    scan_interval: 30

  - platform: template
    sensors:
      lyrebird_blue_yeti_status:
        friendly_name: "Blue Yeti Status"
        value_template: >
          {% set streams = state_attr('sensor.lyrebird_streams', 'items') %}
          {% set blue_yeti = streams | selectattr('name', 'equalto', 'Blue_Yeti') | first %}
          {{ 'Online' if blue_yeti.ready else 'Offline' }}

      lyrebird_blue_yeti_clients:
        friendly_name: "Blue Yeti Clients"
        value_template: >
          {% set streams = state_attr('sensor.lyrebird_streams', 'items') %}
          {% set blue_yeti = streams | selectattr('name', 'equalto', 'Blue_Yeti') | first %}
          {{ blue_yeti.readers | length }}

automation:
  - alias: "LyreBird Stream Alert"
    trigger:
      - platform: numeric_state
        entity_id: sensor.lyrebird_streams
        below: 2
    action:
      - service: notify.mobile_app
        data:
          message: "LyreBird stream count low"
```

---

## Best Practices

### API Integration

1. **Implement Error Handling**
   - Always catch API timeouts
   - Handle connection failures gracefully
   - Implement retry logic with backoff

2. **Rate Limiting**
   - Don't poll API too frequently (max 1/second)
   - Cache responses when appropriate
   - Use webhooks instead of polling when possible

3. **Security**
   - Use authentication when exposing externally
   - Restrict API access to localhost when possible
   - Validate all input data

### Custom Pipelines

1. **Test Thoroughly**
   - Validate FFmpeg pipelines before production
   - Monitor resource usage
   - Verify audio quality

2. **Document Custom Configurations**
   - Comment complex FFmpeg filters
   - Document why custom pipelines were created
   - Version control configurations

3. **Monitor Performance**
   - Track CPU usage of custom pipelines
   - Watch for memory leaks
   - Log errors and warnings

---

## Related Documentation

- [Architecture](architecture.md) - System components
- [MediaMTX Integration](../user-guide/mediamtx-integration.md) - RTSP server
- [Diagnostics & Monitoring](diagnostics-monitoring.md) - Health checks
- [API Reference](https://github.com/bluenviron/mediamtx#api) - MediaMTX API docs

---

## Next Steps

<div class="grid" markdown>

<div markdown>
### Diagnostics & Monitoring
System health checks and monitoring

[Diagnostics & Monitoring →](diagnostics-monitoring.md)
</div>

<div markdown>
### Architecture
System design and component overview

[Architecture →](architecture.md)
</div>

</div>
