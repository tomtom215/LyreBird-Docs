# Environment Variables

Complete reference for all LyreBirdAudio environment variables for runtime configuration and customization.

---

## Overview

LyreBirdAudio scripts use environment variables to customize behavior without modifying configuration files. These variables control timing, resource thresholds, paths, and operational parameters.

**Variable Categories:**

- Timing Controls
- Resource Thresholds
- Path Configuration
- Operational Parameters
- Debug and Logging

---

## Timing Controls

### STREAM_STARTUP_DELAY

**Description:** Delay in seconds after starting streams before validation.

**Default:** `10`

**Valid Values:** `1` to `60`

**Usage:**

```bash
# Wait 15 seconds for stream stabilization
STREAM_STARTUP_DELAY=15 sudo ./mediamtx-stream-manager.sh start
```

**Purpose:**

- Allows streams time to initialize
- Prevents false negatives in health checks
- Adjust based on system performance

**Recommendations:**

| Platform | Recommended Value |
|----------|-------------------|
| Raspberry Pi | 15-20 seconds |
| Intel N100 | 10-15 seconds |
| Workstation | 5-10 seconds |

### USB_STABILIZATION_DELAY

**Description:** Delay in seconds after USB device connection before accessing.

**Default:** `5`

**Valid Values:** `1` to `30`

**Usage:**

```bash
# Wait 10 seconds for USB initialization
USB_STABILIZATION_DELAY=10 sudo ./usb-audio-mapper.sh
```

**Purpose:**

- Ensures USB device fully initialized
- Prevents device access errors
- Allows udev rules to apply

### MAX_RESTART_DELAY

**Description:** Maximum delay in seconds for exponential backoff recovery.

**Default:** `300` (5 minutes)

**Valid Values:** `60` to `1800`

**Usage:**

```bash
# Cap restart delay at 10 minutes
MAX_RESTART_DELAY=600 sudo ./mediamtx-stream-manager.sh start
```

**Purpose:**

- Limits maximum wait time between restart attempts
- Prevents excessively long delays
- Configures recovery aggressiveness

---

## Resource Thresholds

### MAX_CPU_WARNING

**Description:** CPU usage percentage threshold for warnings.

**Default:** `80`

**Valid Values:** `50` to `95`

**Usage:**

```bash
# Warn at 70% CPU usage
MAX_CPU_WARNING=70 sudo ./lyrebird-diagnostics.sh
```

**Purpose:**

- Triggers warnings when CPU usage high
- Helps prevent resource exhaustion
- Configurable based on system capacity

### MAX_MEMORY_WARNING

**Description:** Minimum free memory in MB before warning.

**Default:** `200`

**Valid Values:** `100` to `2000`

**Usage:**

```bash
# Warn when less than 500MB free
MAX_MEMORY_WARNING=500 sudo ./lyrebird-diagnostics.sh
```

**Purpose:**

- Triggers warnings when memory low
- Prevents OOM conditions
- Adjustable based on available RAM

### MAX_FD_WARNING

**Description:** File descriptor count threshold for warnings.

**Default:** `500`

**Valid Values:** `100` to `4000`

**Usage:**

```bash
# Warn at 800 file descriptors
MAX_FD_WARNING=800 sudo ./mediamtx-stream-manager.sh monitor
```

**Purpose:**

- Warns before file descriptor exhaustion
- Prevents "too many open files" errors
- Configurable based on ulimit settings

### MAX_DISK_WARNING

**Description:** Minimum free disk space in GB before warning.

**Default:** `1`

**Valid Values:** `0.5` to `100`

**Usage:**

```bash
# Warn when less than 5GB free
MAX_DISK_WARNING=5 sudo ./lyrebird-diagnostics.sh
```

**Purpose:**

- Alerts when log partition filling up
- Prevents disk space exhaustion
- Adjustable based on disk size

---

## Path Configuration

### AUDIO_CONFIG

**Description:** Path to audio devices configuration file.

**Default:** `/etc/mediamtx/audio-devices.conf`

**Usage:**

```bash
# Use alternate configuration file
AUDIO_CONFIG=/home/user/test-audio-devices.conf \
    sudo ./mediamtx-stream-manager.sh start
```

**Purpose:**

- Test configurations without modifying system files
- Support multiple configuration profiles
- Development and testing

### MEDIAMTX_CONFIG

**Description:** Path to MediaMTX configuration file.

**Default:** `/etc/mediamtx/mediamtx.yml`

**Usage:**

```bash
# Use test MediaMTX configuration
MEDIAMTX_CONFIG=/home/user/test-mediamtx.yml \
    sudo /usr/local/bin/mediamtx
```

**Purpose:**

- Test MediaMTX settings without system changes
- Support multiple server configurations
- Development environments

### LOG_DIR

**Description:** Directory for LyreBird log files.

**Default:** `/var/log/lyrebird`

**Usage:**

```bash
# Log to alternate directory
LOG_DIR=/home/user/logs sudo ./mediamtx-stream-manager.sh start
```

**Purpose:**

- Customize log location
- Testing without system permissions
- Separate logs per environment

### UDEV_RULES_FILE

**Description:** Path to USB persistence udev rules.

**Default:** `/etc/udev/rules.d/99-usb-soundcards.rules`

**Usage:**

```bash
# Generate rules to alternate location
UDEV_RULES_FILE=/tmp/test-udev.rules sudo ./usb-audio-mapper.sh
```

**Purpose:**

- Test udev rules before applying
- Generate rules without system modification
- Development and testing

---

## Operational Parameters

### DRY_RUN

**Description:** Preview actions without executing them. **Only supported by `install_mediamtx.sh`.**

**Default:** `false`

**Valid Values:** `true`, `false`, `1`, `0`

**Usage:**

```bash
# Preview MediaMTX installation steps without actually installing
DRY_RUN=true sudo ./install_mediamtx.sh
```

**Purpose:**

- Preview installation steps before executing
- Test installation script logic
- Validate system compatibility

!!! note "Limited Support"
    The DRY_RUN variable is only implemented in `install_mediamtx.sh`. It is not supported by other LyreBirdAudio scripts.

---

## Debug and Logging

### DIAGNOSTIC_DEBUG

**Description:** Enable detailed diagnostic tool debugging.

**Default:** `false`

**Valid Values:** `true`, `false`, `1`, `0`

**Usage:**

```bash
# Enable diagnostic debugging
DIAGNOSTIC_DEBUG=1 sudo ./lyrebird-diagnostics.sh --verbose
```

**Purpose:**

- Troubleshoot diagnostic tool itself
- Understand check logic
- Report diagnostic issues

### VERBOSE

**Description:** Enable verbose output in scripts.

**Default:** `false`

**Valid Values:** `true`, `false`, `1`, `0`

**Usage:**

```bash
# Verbose output for all operations
VERBOSE=1 sudo ./mediamtx-stream-manager.sh status
```

**Purpose:**

- Detailed operation logging
- Troubleshooting script issues
- Understanding script behavior

### QUIET

**Description:** Suppress non-error output.

**Default:** `false`

**Valid Values:** `true`, `false`, `1`, `0`

**Usage:**

```bash
# Only show errors
QUIET=1 sudo ./mediamtx-stream-manager.sh start
```

**Purpose:**

- Minimal output for automation
- Cron jobs
- Clean output for parsing

### NO_COLOR

**Description:** Disable colored output.

**Default:** `false`

**Valid Values:** `true`, `false`, `1`, `0`

**Usage:**

```bash
# Plain text output
NO_COLOR=1 sudo ./lyrebird-diagnostics.sh
```

**Purpose:**

- Log file clarity
- Terminal compatibility
- Automated parsing

---

## Quality Preset Overrides

### DEFAULT_QUALITY

**Description:** Default quality preset for device configuration generation.

**Default:** `normal`

**Valid Values:** `low`, `normal`, `high`

**Usage:**

```bash
# Generate with high quality defaults
DEFAULT_QUALITY=high sudo ./lyrebird-mic-check.sh -g
```

**Quality Presets:**

| Preset | Sample Rate | Codec | Bitrate |
|--------|-------------|-------|---------|
| low | 16 kHz | opus | 64 kbps |
| normal | 48 kHz | opus | 128 kbps |
| high | 48 kHz | aac | 192 kbps |

---

## Stream Management

### AUTO_RECOVERY_ENABLED

**Description:** Enable automatic stream recovery.

**Default:** `true`

**Valid Values:** `true`, `false`, `1`, `0`

**Usage:**

```bash
# Disable auto-recovery
AUTO_RECOVERY_ENABLED=false sudo ./mediamtx-stream-manager.sh monitor
```

**Purpose:**

- Disable self-healing in testing
- Manual recovery control
- Debugging stream issues

### MAX_RETRY_ATTEMPTS

**Description:** Maximum number of automatic restart attempts.

**Default:** `5`

**Valid Values:** `1` to `20`

**Usage:**

```bash
# Allow 10 restart attempts
MAX_RETRY_ATTEMPTS=10 sudo ./mediamtx-stream-manager.sh start
```

**Purpose:**

- Configure recovery persistence
- Adjust for environment stability
- Balance between recovery and alerting

---

## Environment Variable Combinations

### Production Configuration

```bash
# Production environment settings
export LOGLEVEL=INFO
export AUTO_RECOVERY_ENABLED=true
export MAX_RETRY_ATTEMPTS=5
export MAX_RESTART_DELAY=300
export STREAM_STARTUP_DELAY=10

sudo ./mediamtx-stream-manager.sh start
```

### Development/Testing Configuration

```bash
# Development environment settings
export LOGLEVEL=DEBUG
export DRY_RUN=false
export VERBOSE=1
export AUDIO_CONFIG=/home/user/test-audio-devices.conf
export LOG_DIR=/home/user/test-logs

sudo ./mediamtx-stream-manager.sh start
```

### Resource-Constrained Platform

```bash
# Raspberry Pi optimized settings
export STREAM_STARTUP_DELAY=20
export USB_STABILIZATION_DELAY=10
export MAX_CPU_WARNING=90
export MAX_MEMORY_WARNING=100
export MAX_RESTART_DELAY=600

sudo ./mediamtx-stream-manager.sh start
```

### High-Performance Platform

```bash
# Workstation optimized settings
export STREAM_STARTUP_DELAY=5
export USB_STABILIZATION_DELAY=3
export MAX_CPU_WARNING=70
export MAX_MEMORY_WARNING=500
export MAX_RESTART_DELAY=180

sudo ./mediamtx-stream-manager.sh start
```

---

## Persistent Configuration

### Using .env Files

Create environment file for persistent settings:

```bash
# /opt/lyrebird/.env

# Timing
STREAM_STARTUP_DELAY=15
USB_STABILIZATION_DELAY=8
MAX_RESTART_DELAY=300

# Resources
MAX_CPU_WARNING=80
MAX_MEMORY_WARNING=200
MAX_FD_WARNING=500

# Logging
LOGLEVEL=INFO
LOG_DIR=/var/log/lyrebird
```

**Load in scripts:**

```bash
#!/bin/bash
# Source environment variables
source /opt/lyrebird/.env

# Run command
sudo ./mediamtx-stream-manager.sh start
```

### System-Wide Configuration

Add to `/etc/environment` for system-wide settings:

```bash
# Edit /etc/environment
sudo nano /etc/environment

# Add LyreBird variables
STREAM_STARTUP_DELAY=15
MAX_RESTART_DELAY=300
LOGLEVEL=INFO
```

**Activate:**

```bash
# Reboot or source the file
source /etc/environment
```

### Systemd Service Configuration

Configure environment variables in systemd service files:

```ini
# /etc/systemd/system/mediamtx-audio.service

[Service]
Environment="STREAM_STARTUP_DELAY=15"
Environment="MAX_RESTART_DELAY=300"
Environment="LOG LEVEL=INFO"
Environment="AUTO_RECOVERY_ENABLED=true"

ExecStart=/opt/lyrebird/mediamtx-stream-manager.sh monitor
```

**Apply changes:**

```bash
sudo systemctl daemon-reload
sudo systemctl restart mediamtx-audio
```

---

## Validation and Testing

### Verify Environment Variables

```bash
# Check if variable is set
echo $STREAM_STARTUP_DELAY

# List all LyreBird-related variables
env | grep -E '(STREAM|USB|MAX_|LOG)'

# Test with specific variable
VERBOSE=1 sudo ./mediamtx-stream-manager.sh status
```

### Debugging Variable Issues

```bash
# Enable debug output to see variable usage
DIAGNOSTIC_DEBUG=1 VERBOSE=1 sudo ./lyrebird-diagnostics.sh

# Check variable precedence
# Command line > Environment file > System default
STREAM_STARTUP_DELAY=20 ./mediamtx-stream-manager.sh start
```

---

## Best Practices

### 1. Document Custom Settings

```bash
# Comment environment files
# /opt/lyrebird/.env

# Increased for Raspberry Pi stability
STREAM_STARTUP_DELAY=20

# Lower threshold for 4GB RAM system
MAX_MEMORY_WARNING=150
```

### 2. Use Configuration Files

Prefer `.env` files over inline variables for maintainability:

```bash
# Good: Using environment file
source /opt/lyrebird/.env
sudo ./mediamtx-stream-manager.sh start

# Avoid: Inline for multiple variables
STREAM_STARTUP_DELAY=15 USB_STABILIZATION_DELAY=10 \
  MAX_RESTART_DELAY=300 sudo ./mediamtx-stream-manager.sh start
```

### 3. Test Before Production

Always test environment variable changes:

```bash
# Test with DRY_RUN first
DRY_RUN=true STREAM_STARTUP_DELAY=20 \
  sudo ./mediamtx-stream-manager.sh start

# Then apply for real
STREAM_STARTUP_DELAY=20 sudo ./mediamtx-stream-manager.sh start
```

### 4. Platform-Specific Profiles

Create platform-specific environment files:

```bash
# /opt/lyrebird/env.rpi - Raspberry Pi profile
source /opt/lyrebird/env.common
export STREAM_STARTUP_DELAY=20
export MAX_CPU_WARNING=90

# /opt/lyrebird/env.workstation - Workstation profile
source /opt/lyrebird/env.common
export STREAM_STARTUP_DELAY=5
export MAX_CPU_WARNING=70
```

---

## Related Documentation

- [Configuration Files](configuration-files.md) - File-based configuration
- [Command Reference](command-reference.md) - Commands using variables
- [Performance Tuning](../advanced/performance.md) - Optimization settings
- [Troubleshooting](../advanced/troubleshooting.md) - Debugging

---

**Next Steps:** [Command Reference](command-reference.md) | [Configuration Files](configuration-files.md)
