# MediaMTX Integration

Comprehensive guide to MediaMTX RTSP server configuration, management, and integration with LyreBirdAudio.

---

## Overview

MediaMTX is a modern, high-performance RTSP server that forms the backbone of LyreBirdAudio's streaming infrastructure. It receives audio streams from FFmpeg and distributes them to multiple clients over the network using the RTSP protocol.

This guide covers:

- MediaMTX installation and updates
- Server configuration and management
- HTTP API integration
- RTSP stream access
- Authentication and security
- Performance tuning
- Troubleshooting

---

## What is MediaMTX?

### Purpose

MediaMTX is a ready-to-use RTSP server and proxy that:

- Publishes live audio/video streams via RTSP
- Supports multiple concurrent clients per stream
- Provides HTTP API for monitoring and control
- Handles authentication and access control
- Manages stream lifecycle and metadata
- Offers low-latency streaming performance

### Why MediaMTX for LyreBirdAudio?

**Advantages:**

- **Easy Deployment:** Single binary, minimal dependencies
- **Low Resource Usage:** Efficient handling of multiple streams
- **API Integration:** Comprehensive HTTP API for automation
- **Reliability:** Production-tested RTSP implementation
- **Active Development:** Regular updates and improvements
- **Cross-Platform:** Runs on Linux, Windows, macOS

---

## Installation

### Using the Install Script

LyreBirdAudio provides an automated installation script:

```bash
sudo ./install_mediamtx.sh
```

This script:

1. Downloads latest MediaMTX release from GitHub
2. Installs binary to `/usr/local/bin/mediamtx`
3. Creates configuration directory `/etc/mediamtx/`
4. Generates default configuration file
5. Installs systemd service
6. Starts MediaMTX service

**Example output:**

```
Installing MediaMTX...
Downloading MediaMTX v1.5.0...
Installing to /usr/local/bin/mediamtx...
Creating configuration directory...
Installing systemd service...
Starting MediaMTX...
MediaMTX installed successfully!

RTSP Server: rtsp://localhost:8554
HTTP API: http://localhost:9997
```

### Manual Installation

For manual installation or updates:

```bash
# Download latest release
wget https://github.com/bluenviron/mediamtx/releases/latest/download/mediamtx_linux_amd64.tar.gz

# Extract and install
tar -xzf mediamtx_linux_amd64.tar.gz
sudo mv mediamtx /usr/local/bin/

# Create configuration directory
sudo mkdir -p /etc/mediamtx

# Create default configuration
sudo /usr/local/bin/mediamtx --help > /dev/null
```

### Updating MediaMTX

To update to the latest version:

```bash
sudo ./install_mediamtx.sh --update
```

Or manually:

```bash
# Stop service
sudo systemctl stop mediamtx

# Download and replace binary
wget https://github.com/bluenviron/mediamtx/releases/latest/download/mediamtx_linux_amd64.tar.gz
tar -xzf mediamtx_linux_amd64.tar.gz
sudo mv mediamtx /usr/local/bin/

# Restart service
sudo systemctl start mediamtx
```

---

## Configuration

### Configuration File Location

**Path:** `/etc/mediamtx/mediamtx.yml`

This YAML file controls all MediaMTX server settings.

### Basic Configuration

**Minimal configuration for LyreBirdAudio:**

```yaml
# MediaMTX Configuration for LyreBirdAudio

# RTSP Server
rtspAddress: :8554

# HTTP API
api: yes
apiAddress: :9997

# Logging
logLevel: info
logDestinations: [stdout]

# Path defaults (applies to all streams)
paths:
  all:
    # Allow publishing from localhost
    publishUser: ""
    publishPass: ""
    publishIPs: [127.0.0.1]

    # Allow reading from any IP
    readUser: ""
    readPass: ""
    readIPs: []
```

### Full Configuration Example

**Production-ready configuration:**

```yaml
# MediaMTX Configuration
# Documentation: https://github.com/bluenviron/mediamtx

###############################################
# General
###############################################

# Log level: debug, info, warn, error
logLevel: info

# Log destinations: stdout, file, syslog
logDestinations: [stdout, file]
logFile: /var/log/mediamtx/mediamtx.log

###############################################
# RTSP Server
###############################################

# RTSP server port
rtspAddress: :8554

# Supported protocols: udp, tcp, auto
protocols: [tcp, udp]

# Encryption (RTSPS)
encryption: "no"
serverKey: ""
serverCert: ""

###############################################
# HTTP API
###############################################

# Enable HTTP API
api: yes

# API listen address
apiAddress: :9997

###############################################
# Performance
###############################################

# Read timeout for RTSP connections
readTimeout: 10s

# Write timeout for RTSP connections
writeTimeout: 10s

# Maximum number of simultaneous connections
maxConnections: 100

###############################################
# Path Configuration
###############################################

paths:
  # Default settings for all paths
  all:
    # Source settings
    source: publisher
    sourceOnDemand: no

    # Publishing authentication
    publishUser: ""
    publishPass: ""
    publishIPs: [127.0.0.1]  # Only localhost can publish

    # Reading authentication
    readUser: ""
    readPass: ""
    readIPs: []  # Any IP can read

    # Run commands on events
    runOnInit: ""
    runOnDemand: ""
    runOnReady: ""
    runOnRead: ""
```

### Configuration Parameters

#### Server Settings

| Parameter | Description | Default | Recommended |
|-----------|-------------|---------|-------------|
| `rtspAddress` | RTSP server listen address | :8554 | :8554 |
| `apiAddress` | HTTP API listen address | :9997 | :9997 |
| `logLevel` | Logging verbosity | info | info |
| `encryption` | Enable RTSPS (TLS) | no | no (use VPN) |

#### Path Settings

| Parameter | Description | Options |
|-----------|-------------|---------|
| `publishUser` | Username for publishing | Empty = no auth |
| `publishPass` | Password for publishing | Empty = no auth |
| `publishIPs` | Allowed publisher IPs | 127.0.0.1 for local |
| `readUser` | Username for reading | Empty = no auth |
| `readPass` | Password for reading | Empty = no auth |
| `readIPs` | Allowed reader IPs | Empty = all IPs |

---

## Service Management

### Systemd Service

MediaMTX runs as a systemd service for automatic startup and management.

**Service file location:** `/etc/systemd/system/mediamtx.service`

**Example service file:**

```ini
[Unit]
Description=MediaMTX RTSP Server
After=network.target

[Service]
Type=simple
User=root
ExecStart=/usr/local/bin/mediamtx /etc/mediamtx/mediamtx.yml
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

### Service Commands

**Enable automatic startup:**

```bash
sudo systemctl enable mediamtx
```

**Start service:**

```bash
sudo systemctl start mediamtx
```

**Stop service:**

```bash
sudo systemctl stop mediamtx
```

**Restart service:**

```bash
sudo systemctl restart mediamtx
```

**Check status:**

```bash
sudo systemctl status mediamtx
```

**Example status output:**

```
● mediamtx.service - MediaMTX RTSP Server
   Loaded: loaded (/etc/systemd/system/mediamtx.service; enabled)
   Active: active (running) since Mon 2025-11-15 10:00:00 UTC; 5h 23m ago
 Main PID: 1234 (mediamtx)
    Tasks: 8
   Memory: 45.2M
   CGroup: /system.slice/mediamtx.service
           └─1234 /usr/local/bin/mediamtx /etc/mediamtx/mediamtx.yml

Nov 15 10:00:00 server mediamtx[1234]: 2025/11/15 10:00:00 INF MediaMTX v1.5.0
Nov 15 10:00:00 server mediamtx[1234]: 2025/11/15 10:00:00 INF [RTSP] listener opened on :8554
Nov 15 10:00:00 server mediamtx[1234]: 2025/11/15 10:00:00 INF [API] listener opened on :9997
```

### View Logs

**Using journalctl:**

```bash
# View recent logs
sudo journalctl -u mediamtx -n 50

# Follow logs in real-time
sudo journalctl -u mediamtx -f

# View logs since boot
sudo journalctl -u mediamtx -b

# Filter for errors
sudo journalctl -u mediamtx -p err
```

**Using log file (if configured):**

```bash
tail -f /var/log/mediamtx/mediamtx.log
```

---

## HTTP API

### API Overview

MediaMTX provides a comprehensive HTTP API for monitoring and control.

**Base URL:** `http://localhost:9997`

**API Version:** v3

### Common API Endpoints

#### List All Streams

Get list of all active RTSP paths:

```bash
curl http://localhost:9997/v3/paths/list
```

**Example response:**

```json
{
  "items": [
    {
      "name": "Blue_Yeti",
      "confName": "all",
      "source": {
        "type": "rtspSession"
      },
      "ready": true,
      "readyTime": "2025-11-15T10:00:15.123456Z",
      "tracks": ["audio"],
      "bytesReceived": 1048576,
      "readers": 2
    },
    {
      "name": "USB_Microphone",
      "confName": "all",
      "source": {
        "type": "rtspSession"
      },
      "ready": true,
      "readyTime": "2025-11-15T10:00:16.234567Z",
      "tracks": ["audio"],
      "bytesReceived": 987654,
      "readers": 1
    }
  ]
}
```

#### Get Specific Stream

Get details for a specific stream:

```bash
curl http://localhost:9997/v3/paths/get/Blue_Yeti
```

**Example response:**

```json
{
  "name": "Blue_Yeti",
  "confName": "all",
  "source": {
    "type": "rtspSession",
    "id": "abcd1234"
  },
  "ready": true,
  "readyTime": "2025-11-15T10:00:15.123456Z",
  "tracks": ["audio"],
  "bytesReceived": 1048576,
  "bytesSent": 524288,
  "readers": 2
}
```

#### Server Configuration

Get current server configuration:

```bash
curl http://localhost:9997/v3/config/get
```

#### Server Metrics

Get server performance metrics:

```bash
curl http://localhost:9997/v3/metrics/get
```

### Using the API in Scripts

**Check if stream is active:**

```bash
#!/bin/bash
STREAM_NAME="Blue_Yeti"
RESPONSE=$(curl -s http://localhost:9997/v3/paths/get/$STREAM_NAME)

if echo "$RESPONSE" | grep -q '"ready":true'; then
    echo "Stream $STREAM_NAME is active"
else
    echo "Stream $STREAM_NAME is not active"
fi
```

**Count active streams:**

```bash
#!/bin/bash
STREAM_COUNT=$(curl -s http://localhost:9997/v3/paths/list | grep -o '"name"' | wc -l)
echo "Active streams: $STREAM_COUNT"
```

**Monitor stream readers:**

```bash
#!/bin/bash
STREAM_NAME="Blue_Yeti"
READERS=$(curl -s http://localhost:9997/v3/paths/get/$STREAM_NAME | grep -o '"readers":[0-9]*' | cut -d: -f2)
echo "Stream $STREAM_NAME has $READERS active clients"
```

---

## RTSP Stream Access

### Stream URL Format

Streams are accessible via RTSP URLs:

```
rtsp://<hostname>:<port>/<stream-name>
```

**Examples:**

```bash
# Local access
rtsp://localhost:8554/Blue_Yeti

# Network access
rtsp://192.168.1.100:8554/Blue_Yeti

# Hostname access
rtsp://audio-server.local:8554/Blue_Yeti
```

### Accessing Streams

#### VLC Media Player

**Command line:**

```bash
vlc rtsp://localhost:8554/Blue_Yeti
```

**GUI:**

1. Open VLC
2. Media -> Open Network Stream
3. Enter URL: `rtsp://localhost:8554/Blue_Yeti`
4. Click Play

#### FFplay

```bash
ffplay rtsp://localhost:8554/Blue_Yeti
```

#### FFmpeg (Recording)

```bash
# Record to file
ffmpeg -i rtsp://localhost:8554/Blue_Yeti -c copy output.mkv

# Record with re-encoding
ffmpeg -i rtsp://localhost:8554/Blue_Yeti \
  -c:a libmp3lame -b:a 192k recording.mp3
```

#### GStreamer

```bash
gst-launch-1.0 rtspsrc location=rtsp://localhost:8554/Blue_Yeti ! \
  rtpopusdepay ! opusdec ! audioconvert ! autoaudiosink
```

### Transport Protocols

MediaMTX supports multiple RTSP transport protocols:

**TCP (Default - Recommended):**

```bash
ffplay -rtsp_transport tcp rtsp://localhost:8554/Blue_Yeti
```

- Reliable delivery
- Works through firewalls
- Slightly higher latency

**UDP:**

```bash
ffplay -rtsp_transport udp rtsp://localhost:8554/Blue_Yeti
```

- Lower latency
- May experience packet loss
- May require firewall configuration

---

## Authentication and Security

### Basic Authentication

Enable username/password authentication for streams:

**In `/etc/mediamtx/mediamtx.yml`:**

```yaml
paths:
  all:
    # Require authentication for publishing
    publishUser: "publisher"
    publishPass: "secret123"

    # Require authentication for reading
    readUser: "viewer"
    readPass: "viewer123"
```

**Accessing authenticated streams:**

```bash
# VLC
vlc rtsp://viewer:viewer123@localhost:8554/Blue_Yeti

# FFplay
ffplay rtsp://viewer:viewer123@localhost:8554/Blue_Yeti

# FFmpeg publishing (from stream manager)
ffmpeg -f alsa -i hw:CARD=Blue_Yeti \
  -rtsp_transport tcp \
  -f rtsp rtsp://publisher:secret123@localhost:8554/Blue_Yeti
```

### IP-Based Access Control

Restrict access by IP address:

**Allow specific IPs to publish:**

```yaml
paths:
  all:
    publishIPs: [127.0.0.1, 192.168.1.50]
```

**Allow specific IPs to read:**

```yaml
paths:
  all:
    readIPs: [192.168.1.0/24]  # Entire subnet
```

**Block all external access (local only):**

```yaml
paths:
  all:
    publishIPs: [127.0.0.1]
    readIPs: [127.0.0.1]
```

### RTSPS (Encrypted RTSP)

For encrypted streams, enable RTSPS:

```yaml
encryption: "yes"
serverCert: "/etc/mediamtx/cert.pem"
serverKey: "/etc/mediamtx/key.pem"
```

**Generate self-signed certificate:**

```bash
openssl req -x509 -newkey rsa:4096 -nodes \
  -keyout /etc/mediamtx/key.pem \
  -out /etc/mediamtx/cert.pem \
  -days 365 -subj "/CN=localhost"
```

**Access encrypted streams:**

```bash
vlc rtsps://localhost:8322/Blue_Yeti
```

!!! note "Production Security"
    For production deployments, consider using a VPN (WireGuard, OpenVPN) instead of RTSPS for better security and performance.

---

## Performance Tuning

### Connection Limits

Adjust maximum concurrent connections:

```yaml
maxConnections: 1000
```

**Calculation:**

```
Max Connections = (Streams × Clients per Stream) + Overhead
Example: (10 streams × 5 clients) + 50 = 100 connections
```

### Timeout Settings

Configure client timeout values:

```yaml
readTimeout: 30s   # How long to wait for client data
writeTimeout: 30s  # How long to wait for client writes
```

**Recommendations:**

- **Stable networks:** 10-30 seconds
- **Unstable networks:** 60-120 seconds
- **Mobile clients:** 120+ seconds

### Buffer Settings

Optimize for latency vs. reliability:

**Low latency (real-time monitoring):**

```yaml
paths:
  all:
    readBufferCount: 1
```

**High reliability (recording):**

```yaml
paths:
  all:
    readBufferCount: 512
```

### Resource Usage

**Typical resource consumption:**

- **CPU:** 1-5% per stream
- **Memory:** 10-20 MB per active stream
- **Bandwidth:** Depends on codec/bitrate

**Monitor resource usage:**

```bash
# CPU and memory
top -p $(pidof mediamtx)

# Network bandwidth
iftop -i eth0
```

---

## Troubleshooting

### MediaMTX Won't Start

**Symptom:** Service fails to start

**Solutions:**

1. **Check port conflicts:**
   ```bash
   sudo netstat -tulpn | grep -E '8554|9997'
   ```

2. **Verify configuration syntax:**
   ```bash
   /usr/local/bin/mediamtx --check /etc/mediamtx/mediamtx.yml
   ```

3. **Check permissions:**
   ```bash
   ls -la /etc/mediamtx/mediamtx.yml
   sudo chown root:root /etc/mediamtx/mediamtx.yml
   ```

4. **Review logs:**
   ```bash
   sudo journalctl -u mediamtx -n 50
   ```

### Cannot Access Streams

**Symptom:** RTSP URLs don't work

**Solutions:**

1. **Verify MediaMTX is running:**
   ```bash
   sudo systemctl status mediamtx
   ```

2. **Check stream exists:**
   ```bash
   curl http://localhost:9997/v3/paths/list
   ```

3. **Test API access:**
   ```bash
   curl http://localhost:9997/v3/paths/list
   ```

4. **Check firewall:**
   ```bash
   sudo ufw allow 8554/tcp
   sudo ufw allow 9997/tcp
   ```

5. **Verify network connectivity:**
   ```bash
   telnet localhost 8554
   ```

### Streams Appear but Won't Play

**Symptom:** Stream listed in API but clients can't connect

**Solutions:**

1. **Check authentication:**
   - Ensure credentials match configuration
   - Try without authentication first

2. **Try different transport:**
   ```bash
   ffplay -rtsp_transport tcp rtsp://localhost:8554/Blue_Yeti
   ```

3. **Verify stream is ready:**
   ```bash
   curl http://localhost:9997/v3/paths/get/Blue_Yeti | grep ready
   ```

4. **Check FFmpeg is publishing:**
   ```bash
   ps aux | grep ffmpeg
   ```

### High CPU Usage

**Symptom:** MediaMTX consuming excessive CPU

**Solutions:**

1. **Check number of clients:**
   ```bash
   curl http://localhost:9997/v3/paths/list | grep readers
   ```

2. **Reduce buffer sizes:**
   ```yaml
   readBufferCount: 128
   ```

3. **Limit connections:**
   ```yaml
   maxConnections: 100
   ```

4. **Monitor per-stream usage:**
   ```bash
   top -p $(pidof mediamtx)
   ```

### API Not Responding

**Symptom:** HTTP API returns errors or timeouts

**Solutions:**

1. **Verify API is enabled:**
   ```yaml
   api: yes
   ```

2. **Check API port:**
   ```bash
   sudo netstat -tulpn | grep 9997
   ```

3. **Test with curl:**
   ```bash
   curl -v http://localhost:9997/v3/paths/list
   ```

4. **Check firewall rules:**
   ```bash
   sudo ufw status
   ```

---

## Best Practices

### Production Deployments

1. **Use systemd service** - Enable automatic startup and recovery
2. **Configure logging** - Use file logging for long-term storage
3. **Set resource limits** - Configure appropriate connection limits
4. **Implement monitoring** - Use API to monitor server health
5. **Plan for scale** - Calculate resource requirements before deployment

### Security

1. **Restrict publishing** - Use `publishIPs: [127.0.0.1]` for local-only
2. **Use authentication** - Enable username/password for network access
3. **Deploy behind firewall** - Don't expose RTSP directly to internet
4. **Consider VPN** - Use VPN for remote access instead of public exposure
5. **Regular updates** - Keep MediaMTX updated to latest version

### Performance

1. **Monitor resources** - Track CPU, memory, and bandwidth usage
2. **Optimize buffers** - Adjust based on use case (latency vs reliability)
3. **Limit connections** - Set reasonable maximum connection limits
4. **Use TCP transport** - More reliable for most use cases
5. **Regular maintenance** - Restart service periodically to clear state

### Reliability

1. **Enable auto-restart** - Configure systemd to restart on failure
2. **Monitor logs** - Check for errors and warnings regularly
3. **Test failover** - Verify automatic recovery works
4. **Backup configuration** - Version control mediamtx.yml
5. **Document setup** - Keep records of configuration choices

---

## Related Documentation

- [Stream Management](stream-management.md) - Managing FFmpeg streams
- [Configuration](configuration.md) - Audio encoding settings
- [Multiplex Streaming](multiplex-streaming.md) - Combining multiple audio sources
- [Installation](../getting-started/installation.md) - Initial setup guide
- [Troubleshooting](../advanced/troubleshooting.md) - Common problems and solutions

---

**Next Steps:** [Multiplex Streaming](multiplex-streaming.md) | [Stream Management](stream-management.md)
