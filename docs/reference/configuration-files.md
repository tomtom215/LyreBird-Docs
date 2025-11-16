# Configuration Files

Complete reference for all LyreBirdAudio configuration files, including formats, options, and examples.

---

## Overview

LyreBirdAudio uses multiple configuration files to manage audio devices, streaming settings, USB persistence, and service configuration. This reference provides comprehensive documentation for each file.

**Configuration Files:**

| File | Purpose | Location |
|------|---------|----------|
| `audio-devices.conf` | Device audio settings | `/etc/mediamtx/audio-devices.conf` |
| `mediamtx.yml` | MediaMTX server config | `/etc/mediamtx/mediamtx.yml` |
| `99-usb-soundcards.rules` | USB persistence rules | `/etc/udev/rules.d/99-usb-soundcards.rules` |
| `mediamtx.service` | MediaMTX systemd service | `/etc/systemd/system/mediamtx.service` |
| `mediamtx-audio.service` | Stream manager service | `/etc/systemd/system/mediamtx-audio.service` |

---

## Audio Devices Configuration

## File Location

`/etc/mediamtx/audio-devices.conf`

## Purpose

Defines audio encoding settings for each USB audio device, including codec, sample rate, bitrate, and channel configuration.

## Format

Shell script format with environment variables:

```bash
# Device name must match ALSA card name from /proc/asound/cards

DEVICE_<DEVICE_NAME>_CODEC=<codec>
DEVICE_<DEVICE_NAME>_SAMPLE_RATE=<rate>
DEVICE_<DEVICE_NAME>_BITRATE=<bitrate>
DEVICE_<DEVICE_NAME>_CHANNELS=<channels>
DEVICE_<DEVICE_NAME>_THREAD_QUEUE=<size>
```

## Configuration Variables

**DEVICE_<NAME>_CODEC**

Audio codec for encoding.

- **Type:** String
- **Required:** Yes
- **Valid Values:** `opus`, `aac`, `mp3`, `pcm_s16le`, `pcm_s24le`
- **Default:** `opus`

**Examples:**

```bash
DEVICE_Blue_Yeti_CODEC=opus
DEVICE_Studio_Mic_CODEC=aac
DEVICE_USB_Microphone_CODEC=mp3
```

**DEVICE_<NAME>_SAMPLE_RATE**

Audio sampling rate in Hz.

- **Type:** Integer
- **Required:** Yes
- **Valid Values:** `8000`, `12000`, `16000`, `22050`, `24000`, `44100`, `48000`, `96000`
- **Default:** `48000`

**Examples:**

```bash
DEVICE_Blue_Yeti_SAMPLE_RATE=48000      # Standard quality
DEVICE_Voice_Mic_SAMPLE_RATE=16000      # Voice optimized
DEVICE_HiFi_Mic_SAMPLE_RATE=96000       # High fidelity
```

**DEVICE_<NAME>_BITRATE**

Target encoding bitrate.

- **Type:** String (with 'k' suffix)
- **Required:** Yes
- **Valid Values:** `24k` to `320k` (depends on codec)
- **Default:** `128k`

**Examples:**

```bash
DEVICE_Blue_Yeti_BITRATE=128k      # Normal quality
DEVICE_Voice_Mic_BITRATE=64k       # Low bandwidth
DEVICE_Studio_Mic_BITRATE=192k     # High quality
```

**DEVICE_<NAME>_CHANNELS**

Number of audio channels.

- **Type:** Integer
- **Required:** Yes
- **Valid Values:** `1` (mono), `2` (stereo)
- **Default:** `1`

**Examples:**

```bash
DEVICE_Blue_Yeti_CHANNELS=1      # Mono
DEVICE_Studio_Mic_CHANNELS=2     # Stereo
```

**DEVICE_<NAME>_THREAD_QUEUE**

FFmpeg thread queue size for buffering.

- **Type:** Integer
- **Required:** No
- **Valid Values:** `512` to `32768`
- **Default:** `8192`

**Examples:**

```bash
DEVICE_Blue_Yeti_THREAD_QUEUE=8192      # Default
DEVICE_Unreliable_USB_THREAD_QUEUE=16384 # Increased stability
DEVICE_Low_Latency_THREAD_QUEUE=1024     # Reduced buffer size
```

## Complete Example

```bash
# /etc/mediamtx/audio-devices.conf

# Blue Yeti - High quality mono streaming
DEVICE_Blue_Yeti_CODEC=opus
DEVICE_Blue_Yeti_SAMPLE_RATE=48000
DEVICE_Blue_Yeti_BITRATE=128k
DEVICE_Blue_Yeti_CHANNELS=1
# THREAD_QUEUE defaults to 8192 if not specified

# Studio Microphone - Stereo AAC for music
DEVICE_Studio_Mic_CODEC=aac
DEVICE_Studio_Mic_SAMPLE_RATE=48000
DEVICE_Studio_Mic_BITRATE=192k
DEVICE_Studio_Mic_CHANNELS=2
DEVICE_Studio_Mic_THREAD_QUEUE=16384  # Increased for high-quality stereo

# USB Microphone - Low bandwidth voice
DEVICE_USB_Microphone_CODEC=opus
DEVICE_USB_Microphone_SAMPLE_RATE=16000
DEVICE_USB_Microphone_BITRATE=64k
DEVICE_USB_Microphone_CHANNELS=1
# THREAD_QUEUE defaults to 8192 if not specified

# Conference Room Mic - Standard quality
DEVICE_Conference_Room_CODEC=opus
DEVICE_Conference_Room_SAMPLE_RATE=24000
DEVICE_Conference_Room_BITRATE=96k
DEVICE_Conference_Room_CHANNELS=1
# THREAD_QUEUE defaults to 8192 if not specified
```

## Codec-Specific Guidelines

**Opus (Recommended for most use cases):**

```bash
DEVICE_<NAME>_CODEC=opus
DEVICE_<NAME>_SAMPLE_RATE=48000  # Best quality
DEVICE_<NAME>_BITRATE=128k       # Good quality
DEVICE_<NAME>_CHANNELS=1         # Mono or 2 for stereo
```

- Best CPU efficiency
- Excellent quality at low bitrates
- Ideal for voice: 24-64 kbps
- Ideal for music: 96-128 kbps

**AAC (Universal compatibility, high quality):**

```bash
DEVICE_<NAME>_CODEC=aac
DEVICE_<NAME>_SAMPLE_RATE=48000  # Standard
DEVICE_<NAME>_BITRATE=192k       # High quality
DEVICE_<NAME>_CHANNELS=2         # Stereo
```

- Widely compatible
- Excellent for music
- Recommended: 128-256 kbps

**MP3 (Legacy compatibility):**

```bash
DEVICE_<NAME>_CODEC=mp3
DEVICE_<NAME>_SAMPLE_RATE=44100  # CD quality
DEVICE_<NAME>_BITRATE=160k       # Good quality
DEVICE_<NAME>_CHANNELS=2         # Stereo
```

- Universal compatibility
- Higher CPU usage than Opus
- Recommended: 128-192 kbps

**PCM (Uncompressed, maximum quality):**

```bash
DEVICE_<NAME>_CODEC=pcm_s16le
DEVICE_<NAME>_SAMPLE_RATE=48000  # Standard
# No bitrate setting (uncompressed)
DEVICE_<NAME>_CHANNELS=2         # Stereo
```

- Perfect quality
- Very high bandwidth (~1.5 Mbps)
- Minimal CPU usage
- For local networks only

---

## MediaMTX Configuration

## File Location

`/etc/mediamtx/mediamtx.yml`

## Purpose

Configures the MediaMTX RTSP server, including ports, protocols, authentication, and path configuration.

## Format

YAML configuration file.

## Key Configuration Sections

**General Settings:**

```yaml
# Logging level (debug, info, warn, error)
logLevel: info

# Log destinations (stdout, file, syslog)
logDestinations: [stdout]

# Log file path (if using file destination)
logFile: /var/log/mediamtx.out

# Read timeout for connections
readTimeout: 10s

# Write timeout for connections
writeTimeout: 10s

# UDP read buffer size
readBufferCount: 512
```

**API Configuration:**

```yaml
# Enable HTTP API
api: yes

# API listening address
apiAddress: :9997
```

**RTSP Configuration:**

```yaml
# Enable RTSP protocol
rtsp: yes

# RTSP listening address (:8554 for all interfaces)
rtspAddress: :8554

# RTSP protocols (udp, tcp, both)
protocols: [tcp]

# Encryption (no, optional, strict)
encryption: no

# RTSP transport protocol
rtspTransport: automatic
```

**Path Configuration:**

```yaml
# Default path configuration (applies to all streams)
paths:
  all:
    # Source: publisher or redirect
    # Empty = accept from any publisher
    source:

    # Run on init (command to run when path is initialized)
    runOnInit:

    # Run on demand (command to run when first reader connects)
    runOnDemand:

    # Run on ready (command to run when source is ready)
    runOnReady:

    # Run on read (command to run when client starts reading)
    runOnRead:

    # Run on disconnect (command to run when client disconnects)
    runOnDisconnect:
```

## Complete Example

```yaml
# /etc/mediamtx/mediamtx.yml

# General settings
logLevel: info
logDestinations: [stdout, file]
logFile: /var/log/mediamtx.out
readTimeout: 10s
writeTimeout: 10s
readBufferCount: 512

# API configuration
api: yes
apiAddress: :9997

# RTSP configuration
rtsp: yes
rtspAddress: :8554
protocols: [tcp, udp]
encryption: no
rtspTransport: automatic

# HLS configuration (optional)
hls: yes
hlsAddress: :8888
hlsAlwaysRemux: no

# WebRTC configuration (optional)
webrtc: yes
webrtcAddress: :8889

# Paths configuration
paths:
  # Default configuration for all paths
  all:
    # Record streams (optional)
    # record: yes
    # recordPath: /var/recordings/%path/%Y-%m-%d_%H-%M-%S.mp4

    # Run commands on events (optional)
    # runOnReady: /usr/local/bin/on-stream-ready.sh %path
    # runOnRead: /usr/local/bin/on-client-connect.sh %path
    # runOnDisconnect: /usr/local/bin/on-client-disconnect.sh %path

  # Specific path configuration example
  Blue_Yeti:
    # Override settings for this specific stream
    # readTimeout: 20s

  # Multiplex stream example
  Office_Audio:
    # Custom configuration for multiplex stream
    # runOnReady: /path/to/multiplex-handler.sh
```

## Path Variables

When using `runOn*` commands, these variables are available:

| Variable | Description |
|----------|-------------|
| `%path` | Stream path name |
| `%query` | URL query parameters |
| `%sourceType` | Source type (rtsp, rtmp, etc.) |
| `%port` | Server port |

---

## USB Persistence Rules

## File Location

`/etc/udev/rules.d/99-usb-soundcards.rules`

## Purpose

Creates persistent device naming for USB audio devices to prevent card number changes across reboots.

## Format

udev rules format.

## Rule Syntax

```text
SUBSYSTEM=="sound", \
  ACTION=="add", \
  ATTRS{idVendor}=="<vendor_id>", \
  ATTRS{idProduct}=="<product_id>", \
  ATTR{number}="<card_number>"
```

**Or for symbolic links:**

```text
SUBSYSTEM=="sound", \
  ATTRS{idVendor}=="<vendor_id>", \
  ATTRS{idProduct}=="<product_id>", \
  ATTR{id}="<device_name>", \
  SYMLINK+="sound/by-id/<device_name>"
```

## Complete Example

```bash
# /etc/udev/rules.d/99-usb-soundcards.rules

# Blue Yeti - Always assign to card 0
SUBSYSTEM=="sound", \
  ACTION=="add", \
  ATTRS{idVendor}=="046d", \
  ATTRS{idProduct}=="0a44", \
  ATTR{number}="0"

# USB Microphone - Always assign to card 1
SUBSYSTEM=="sound", \
  ACTION=="add", \
  ATTRS{idVendor}=="0d8c", \
  ATTRS{idProduct}=="0014", \
  ATTR{number}="1"

# Studio Mic - Symbolic link approach
SUBSYSTEM=="sound", \
  ATTRS{idVendor}=="0763", \
  ATTRS{idProduct}=="2030", \
  ATTR{id}="Studio_Mic", \
  SYMLINK+="sound/by-id/Studio_Mic"

# Conference Room Mic - USB topology based
SUBSYSTEM=="sound", \
  KERNELS=="1-1.2", \
  ATTR{number}="2"
```

## Finding Device IDs

```bash
# List USB devices with vendor/product IDs
lsusb

# Example output:
# Bus 001 Device 004: ID 046d:0a44 Logitech, Inc. Blue Yeti

# Get detailed device information
udevadm info -a -p $(udevadm info -q path -n /dev/snd/controlC0)

# Show USB topology
lsusb -t
```

## Applying Rules

```bash
# Reload udev rules
sudo udevadm control --reload-rules

# Trigger rules for connected devices
sudo udevadm trigger

# Test rule application
sudo udevadm test /sys/class/sound/card0
```

---

## Systemd Service Files

## MediaMTX Service

**File Location:** `/etc/systemd/system/mediamtx.service`

**Purpose:** Manages MediaMTX RTSP server as a systemd service.

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
StandardOutput=journal
StandardError=journal

# Resource limits (optional)
LimitNOFILE=4096
LimitNPROC=512

[Install]
WantedBy=multi-user.target
```

**Management:**

```bash
# Enable automatic startup
sudo systemctl enable mediamtx

# Start service
sudo systemctl start mediamtx

# Stop service
sudo systemctl stop mediamtx

# Restart service
sudo systemctl restart mediamtx

# Check status
sudo systemctl status mediamtx

# View logs
sudo journalctl -u mediamtx -f
```

## Stream Manager Service

**File Location:** `/etc/systemd/system/mediamtx-audio.service`

**Purpose:** Manages audio stream lifecycle with automatic recovery.

```ini
[Unit]
Description=LyreBird Audio Stream Manager
After=network.target mediamtx.service
Requires=mediamtx.service

[Service]
Type=simple
User=root
WorkingDirectory=/opt/lyrebird
ExecStart=/opt/lyrebird/mediamtx-stream-manager.sh monitor
Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal

# Environment variables (optional)
Environment="STREAM_STARTUP_DELAY=10"
Environment="MAX_RESTART_DELAY=300"

# Resource limits
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
```

**Management:**

```bash
# Enable automatic startup
sudo systemctl enable mediamtx-audio

# Start service
sudo systemctl start mediamtx-audio

# Stop service
sudo systemctl stop mediamtx-audio

# Restart service
sudo systemctl restart mediamtx-audio

# Check status
sudo systemctl status mediamtx-audio

# View logs
sudo journalctl -u mediamtx-audio -f
```

---

## Configuration Best Practices

## Backup Configurations

Always backup before making changes:

```bash
# Backup all configuration files
sudo mkdir -p /etc/mediamtx/backups/$(date +%Y%m%d)
sudo cp /etc/mediamtx/audio-devices.conf \
       /etc/mediamtx/backups/$(date +%Y%m%d)/
sudo cp /etc/mediamtx/mediamtx.yml \
       /etc/mediamtx/backups/$(date +%Y%m%d)/
sudo cp /etc/udev/rules.d/99-usb-soundcards.rules \
       /etc/mediamtx/backups/$(date +%Y%m%d)/
```

## Version Control

Use git for configuration tracking:

```bash
# Initialize git repository
cd /etc/mediamtx
sudo git init
sudo git add audio-devices.conf mediamtx.yml
sudo git commit -m "Initial configuration"

# Track changes
sudo git diff audio-devices.conf
sudo git commit -am "Update Blue Yeti bitrate"
```

## Validation

Validate configurations before applying:

```bash
# Validate audio device configuration
sudo ./lyrebird-mic-check.sh -V

# Test udev rules
sudo udevadm test /sys/class/sound/card0
```

## Documentation

Document custom configurations:

```bash
# Add comments to audio-devices.conf
# Blue Yeti - Conference room microphone
# Using high quality settings for Zoom calls
DEVICE_Blue_Yeti_CODEC=opus
DEVICE_Blue_Yeti_SAMPLE_RATE=48000
DEVICE_Blue_Yeti_BITRATE=128k
DEVICE_Blue_Yeti_CHANNELS=1
DEVICE_Blue_Yeti_THREAD_QUEUE=2048  # Increased for USB stability
```

---

## Related Documentation

- [Environment Variables](environment-variables.md) - Runtime configuration
- [Command Reference](command-reference.md) - Configuration tools
- [Configuration Guide](../user-guide/configuration.md) - User guide
- [USB Device Management](../user-guide/usb-device-management.md) - Device setup
- [Performance Tuning](../advanced/performance.md) - Optimization

---

## Next Steps

<div class="grid" markdown>

<div markdown>
## Environment Variables
Runtime configuration and customization

[Environment Variables →](environment-variables.md)
</div>

<div markdown>
## Command Reference
All available commands and utilities

[Command Reference →](command-reference.md)
</div>

</div>
