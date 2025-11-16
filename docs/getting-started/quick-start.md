# Quick Start

Get LyreBirdAudio up and running in 5 minutes with the interactive orchestrator wizard.

---

## Prerequisites

Before starting, ensure you have:

- Linux system (Ubuntu, Debian, Raspberry Pi OS, or derivatives)
- USB microphone(s) connected
- Root/sudo access
- Internet connection for downloading MediaMTX

---

## Installation

### Step 1: Clone the Repository

```bash
git clone https://github.com/tomtom215/LyreBirdAudio.git
cd LyreBirdAudio
```

### Step 2: Run the Orchestrator

```bash
sudo ./lyrebird-orchestrator.sh
```

The orchestrator wizard will guide you through:

1. **MediaMTX Installation** - Automatic download and setup
2. **USB Device Mapping** - Create persistent device names
3. **Stream Configuration** - Setup RTSP streams
4. **Service Installation** - Enable auto-start on boot

---

## Interactive Setup

The orchestrator presents a menu-driven interface:

```text
========================================
LyreBirdAudio Orchestrator v2.1.0
========================================

1. Install/Update MediaMTX
2. Map USB Audio Devices
3. List Available Streams
4. Add New Stream
5. Remove Stream
6. Start All Streams
7. Stop All Streams
8. Install/Update Stream Manager Service
9. Check System Status
10. Version Management
11. Exit

Select an option:
```

---

## Quick Setup Path

For a standard single-microphone setup:

1. **Install MediaMTX** (Option 1)
   - Automatically detects your architecture
   - Downloads and installs latest version
   - Configures for LyreBirdAudio

2. **Map USB Device** (Option 2)
   - Detects connected USB microphones
   - Creates persistent device name via udev
   - Generates symlinks for consistent device access

3. **Add Stream** (Option 4)
   - Select mapped device
   - Auto-detects optimal audio settings
   - Creates RTSP stream configuration

4. **Install Service** (Option 8)
   - Installs systemd service
   - Enables auto-start on boot
   - Starts service immediately

---

## Verify Installation

### Check Stream Status

```bash
sudo ./mediamtx-stream-manager.sh status
```

Expected output:
```text
Stream Status:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Device_1    RUNNING    PID: 1234    Uptime: 5m
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Test the Stream

=== "VLC Player"

    ```bash
    # Stream name depends on your device naming
    vlc rtsp://localhost:8554/Device_1
    ```

=== "FFmpeg"

    ```bash
    # Stream name depends on your device naming
    ffmpeg -i rtsp://localhost:8554/Device_1 -t 10 -acodec copy test.aac
    ```

=== "cURL (Metadata)"

    ```bash
    # Stream name depends on your device naming
    curl -v rtsp://localhost:8554/Device_1
    ```

---

## Configuration Files

After setup, you'll have:

| File | Location | Purpose |
|------|----------|---------|
| MediaMTX Config | `/etc/mediamtx/mediamtx.yml` | Server configuration |
| Audio Device Config | `/etc/mediamtx/audio-devices.conf` | Stream settings and device parameters |
| udev Rules | `/etc/udev/rules.d/99-usb-soundcards.rules` | Device persistence |
| Service File | `/etc/systemd/system/mediamtx-audio.service` | Auto-start configuration |

---

## Next Steps

<div class="grid" markdown>

<div markdown>
###  Configuration
Learn about advanced audio settings and stream options

[Configuration Guide ->](../user-guide/configuration.md)
</div>

<div markdown>
###  Multiple Devices
Setup and manage multiple USB microphones

[USB Device Management ->](../user-guide/usb-device-management.md)
</div>

<div markdown>
###  Monitoring
Setup health checks and monitoring

[Diagnostics ->](../advanced/diagnostics-monitoring.md)
</div>

<div markdown>
###  Troubleshooting
Common issues and solutions

[Troubleshooting ->](../advanced/troubleshooting.md)
</div>

</div>

---

## Common Issues

!!! warning "MediaMTX Port Already in Use"
    If port 8554 is already in use, edit `/opt/mediamtx/mediamtx.yml` and change the `rtspAddress` port.

!!! warning "USB Device Not Detected"
    Run `arecord -l` to list all audio devices. If your microphone doesn't appear, check USB connection and driver support.

!!! tip "Testing Without VLC"
    Use `ffplay` for quick testing: `ffplay -nodisp rtsp://localhost:8554/lyrebird-mic-1`

---

## Getting Help

- **Detailed Installation Guide:** [Installation](installation.md)
- **System Requirements:** [Requirements](system-requirements.md)
- **Report Issues:** [GitHub Issues](https://github.com/tomtom215/LyreBirdAudio/issues)
