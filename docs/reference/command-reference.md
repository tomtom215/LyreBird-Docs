# Command Reference

Complete reference for all LyreBirdAudio commands, scripts, and utilities.

---

## Overview

LyreBirdAudio provides several command-line tools for managing USB audio devices, streams, and system health.

**Available Commands:**

| Command | Purpose |
|---------|---------|
| `lyrebird-orchestrator.sh` | Main menu system |
| `mediamtx-stream-manager.sh` | Stream lifecycle management |
| `usb-audio-mapper.sh` | USB device mapping |
| `lyrebird-mic-check.sh` | Device capability detection |
| `lyrebird-diagnostics.sh` | System health checks |
| `lyrebird-updater.sh` | Version management |
| `install_mediamtx.sh` | MediaMTX installation |

---

## lyrebird-orchestrator.sh

**Purpose:** Interactive menu system for LyreBirdAudio

**Syntax:**

```bash
sudo ./lyrebird-orchestrator.sh
```

**Description:**

Main entry point providing user-friendly menu interface for all LyreBirdAudio operations.

**Features:**

- Device detection and mapping
- Stream management
- System diagnostics
- Configuration assistance
- Version management

**Usage:**

```bash
# Launch interactive menu
sudo ./lyrebird-orchestrator.sh
```

---

## mediamtx-stream-manager.sh

**Purpose:** Manage RTSP stream lifecycle with automatic recovery

**Syntax:**

```bash
sudo ./mediamtx-stream-manager.sh <command> [options]
```

**Commands:**

### start

Start all configured audio streams.

```bash
sudo ./mediamtx-stream-manager.sh start
```

**Behavior:**

- Reads `/etc/mediamtx/audio-devices.conf`
- Launches FFmpeg for each device
- Waits for stream stabilization
- Validates via MediaMTX API
- Reports status

### stop

Stop all running streams.

```bash
sudo ./mediamtx-stream-manager.sh stop
```

**Behavior:**

- Sends SIGTERM to FFmpeg processes
- Waits for graceful shutdown
- Forces SIGKILL if needed
- Cleans up PID files

### restart

Restart all streams (stop then start).

```bash
sudo ./mediamtx-stream-manager.sh restart
```

### status

Display current stream status.

```bash
sudo ./mediamtx-stream-manager.sh status
```

**Output:**

- Stream name and status
- PID and uptime
- RTSP URL
- Codec and settings
- Client connections

### list

List all available streams from MediaMTX.

```bash
sudo ./mediamtx-stream-manager.sh list
```

### monitor

Continuous monitoring with automatic recovery.

```bash
sudo ./mediamtx-stream-manager.sh monitor
```

**Features:**

- Real-time health checks
- Automatic stream restart
- Exponential backoff
- Resource tracking
- Press Ctrl+C to exit

**Environment Variables:**

- `STREAM_STARTUP_DELAY` - Stream stabilization wait time
- `MAX_RESTART_DELAY` - Maximum exponential backoff
- `AUTO_RECOVERY_ENABLED` - Enable/disable auto-recovery

---

## usb-audio-mapper.sh

**Purpose:** Generate persistent USB device mapping rules

**Syntax:**

```bash
sudo ./usb-audio-mapper.sh [--test]
```

**Options:**

### --test

Test mode - preview without modifying files.

```bash
sudo ./usb-audio-mapper.sh --test
```

**Behavior:**

- Detects all USB audio devices
- Queries vendor/product IDs
- Generates udev rules
- Writes to `/etc/udev/rules.d/99-usb-soundcards.rules`
- Reloads udev
- Triggers rule application

**Example Output:**

```text
Detected USB Audio Devices:
===========================
Card 0: Blue_Yeti
  Vendor: 046d
  Product: 0a44
  USB Path: 1-1.2

Generated rules: /etc/udev/rules.d/99-usb-soundcards.rules
Reloaded udev rules
```

---

## lyrebird-mic-check.sh

**Purpose:** Detect device capabilities and generate configuration

**Syntax:**

```bash
sudo ./lyrebird-mic-check.sh [options]
```

**Options:**

### -g

Generate device configuration.

```bash
sudo ./lyrebird-mic-check.sh -g
```

Creates `/etc/mediamtx/audio-devices.conf` with detected settings.

### --quality=<preset>

Set quality preset (low, normal, high).

```bash
sudo ./lyrebird-mic-check.sh -g --quality=high
```

**Quality Presets:**

| Preset | Sample Rate | Codec | Bitrate |
|--------|-------------|-------|---------|
| low | 16 kHz | opus | 64 kbps |
| normal | 48 kHz | opus | 128 kbps |
| high | 48 kHz | aac | 192 kbps |

### -f

Force overwrite existing configuration.

```bash
sudo ./lyrebird-mic-check.sh -g -f
```

## -V

Validate existing configuration.

```bash
sudo ./lyrebird-mic-check.sh -V
```

Checks `/etc/mediamtx/audio-devices.conf` syntax and values.

**Example:**

```bash
# Generate normal quality configuration
sudo ./lyrebird-mic-check.sh -g --quality=normal

# Validate configuration
sudo ./lyrebird-mic-check.sh -V
```

---

## lyrebird-diagnostics.sh

**Purpose:** Comprehensive system health checks

**Syntax:**

```bash
sudo ./lyrebird-diagnostics.sh [options]
```

**Modes:**

### Default (Quick Check)

```bash
sudo ./lyrebird-diagnostics.sh
```

Runs all critical checks with summary output.

### --verbose

Detailed diagnostic output.

```bash
sudo ./lyrebird-diagnostics.sh --verbose
```

Includes extended device information, logs, and configuration validation.

**Exit Codes:**

| Code | Status | Description |
|------|--------|-------------|
| 0 | Success | All checks passed |
| 1 | Warnings | System operational with warnings |
| 2 | Failures | Critical issues detected |
| 127 | Error | Diagnostic tool error |

**Checks Performed:**

- Operating system compatibility
- Required packages installed
- USB audio devices detected
- udev rules configured
- MediaMTX service running
- Stream health
- Resource usage (CPU, memory, FDs)
- Network connectivity
- Log file access

---

## lyrebird-updater.sh

**Purpose:** Version management and updates

**Syntax:**

```bash
sudo ./lyrebird-updater.sh [options]
```

**Options:**

### --status

Check current version and update availability.

```bash
sudo ./lyrebird-updater.sh --status
```

**Output:**

```text
Current Version: 1.2.0
Latest Version: 1.3.0
Update Available: Yes
```

### --list

List all available versions.

```bash
sudo ./lyrebird-updater.sh --list
```

### --update

Update to latest version.

```bash
sudo ./lyrebird-updater.sh --update
```

**Behavior:**

- Backs up current configuration
- Downloads latest version
- Applies updates
- Preserves custom settings
- Restarts services if needed

---

## install_mediamtx.sh

**Purpose:** Install or update MediaMTX RTSP server

**Syntax:**

```bash
sudo ./install_mediamtx.sh
```

**Behavior:**

- Detects system architecture
- Downloads latest MediaMTX binary
- Installs to `/usr/local/bin/mediamtx`
- Creates configuration directory
- Generates default configuration
- Sets up systemd service
- Enables and starts service

**Example:**

```bash
# Install MediaMTX
sudo ./install_mediamtx.sh

# Verify installation
/usr/local/bin/mediamtx --version

# Check service status
sudo systemctl status mediamtx
```

---

## Common Usage Patterns

## Initial Setup

```bash
# 1. Install MediaMTX
sudo ./install_mediamtx.sh

# 2. Map USB devices
sudo ./usb-audio-mapper.sh

# 3. Generate configuration
sudo ./lyrebird-mic-check.sh -g --quality=normal

# 4. Start streams
sudo ./mediamtx-stream-manager.sh start

# 5. Verify health
sudo ./lyrebird-diagnostics.sh
```

## Daily Operations

```bash
# Check stream status
sudo ./mediamtx-stream-manager.sh status

# Monitor streams
sudo ./mediamtx-stream-manager.sh monitor

# Run health check
sudo ./lyrebird-diagnostics.sh
```

## Troubleshooting

```bash
# Detailed diagnostics
sudo ./lyrebird-diagnostics.sh --verbose

# Restart streams
sudo ./mediamtx-stream-manager.sh restart

# Regenerate USB mapping
sudo ./usb-audio-mapper.sh

# Validate configuration
sudo ./lyrebird-mic-check.sh -V
```

## Production Deployment

```bash
# Enable automatic startup
sudo systemctl enable mediamtx
sudo systemctl enable mediamtx-audio

# Start services
sudo systemctl start mediamtx
sudo systemctl start mediamtx-audio

# Verify status
sudo systemctl status mediamtx
sudo systemctl status mediamtx-audio
```

---

## Related Documentation

- [Environment Variables](environment-variables.md) - Runtime configuration
- [Configuration Files](configuration-files.md) - File-based settings
- [Exit Codes](exit-codes.md) - Command return values
- [Getting Started](../getting-started/quick-start.md) - First-time setup

---

## Next Steps

<div class="grid" markdown>

<div markdown>
## Exit Codes
Command return values and status codes

[Exit Codes →](exit-codes.md)
</div>

<div markdown>
## Log Files
Log locations, formats, and analysis

[Log Files →](log-files.md)
</div>

</div>
