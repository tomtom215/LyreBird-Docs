# Code Standards

Coding standards and best practices for LyreBirdAudio development.

---

## Overview

LyreBirdAudio follows strict coding standards to ensure maintainability, reliability, and consistency across all scripts. This guide covers Bash scripting conventions, error handling, logging, configuration management, and security practices.

**Core Principles:**

- **Clarity:** Code should be self-explanatory
- **Consistency:** Follow established patterns
- **Robustness:** Handle errors gracefully
- **Security:** Validate all inputs
- **Maintainability:** Document complex logic

---

## Bash Scripting Standards

## Bash Version Requirements

**Minimum Version: Bash 4.0+**

```bash
# Required for associative arrays
declare -A DEVICE_CONFIG
DEVICE_CONFIG["sample_rate"]="48000"
DEVICE_CONFIG["bitrate"]="128k"

# Verify Bash version at script start
if (( BASH_VERSINFO[0] < 4 )); then
    echo "Error: Bash 4.0+ required" >&2
    exit 1
fi
```

## Shebang and Script Header

**Required Header:**

```bash
#!/bin/bash
#
# Script Name: lyrebird-example-script.sh
# Description: Brief description of script purpose
# Author: LyreBirdAudio Contributors
# License: Apache 2.0
#
# Usage: ./lyrebird-example-script.sh [options]
#
# Dependencies:
#   - ffmpeg
#   - alsa-utils
#   - udev
#
```

**Strict Mode:**

```bash
# Enable strict error handling
set -euo pipefail

# Explanation:
# -e: Exit on error
# -u: Exit on undefined variable
# -o pipefail: Exit if any command in pipeline fails
```

!!! warning "Strict Mode Exceptions"
    Use `set +e` temporarily when errors are expected and handled:
    ```bash
    set +e
    optional_command_that_might_fail
    EXIT_CODE=$?
    set -e
    ```

## Variable Naming Conventions

**Constants (Uppercase):**

```bash
# Global constants
readonly MEDIAMTX_INSTALL_DIR="/opt/mediamtx"
readonly CONFIG_DIR="/etc/mediamtx"
readonly DEFAULT_SAMPLE_RATE=48000
readonly VERSION="2.1.0"

# Environment variables
LYREBIRD_HOME="${LYREBIRD_HOME:-/opt/LyreBirdAudio}"
LOG_LEVEL="${LOG_LEVEL:-INFO}"
```

**Variables (Lowercase with underscores):**

```bash
# Local variables
local device_name="mic-front-door"
local sample_rate=48000
local bitrate="128k"

# Function parameters
function configure_device() {
    local device_path="$1"
    local config_file="$2"
    # ...
}
```

**Associative Arrays (Uppercase):**

```bash
# Declare associative arrays
declare -A DEVICE_CONFIG
declare -A STREAM_STATUS

# Populate arrays
DEVICE_CONFIG["device"]="/dev/lyrebird-mic-1"
DEVICE_CONFIG["sample_rate"]="48000"
DEVICE_CONFIG["codec"]="aac"
```

## Quoting Rules

**Always Quote Variables:**

```bash
# Good: Variables quoted
if [[ -f "$CONFIG_FILE" ]]; then
    cp "$CONFIG_FILE" "$BACKUP_DIR/"
fi

# Bad: Unquoted variables (word splitting issues)
if [[ -f $CONFIG_FILE ]]; then
    cp $CONFIG_FILE $BACKUP_DIR/
fi
```

**Quote Command Substitution:**

```bash
# Good: Command substitution quoted
device_name="$(basename "$device_path")"
timestamp="$(date +%Y%m%d-%H%M%S)"

# Bad: Unquoted command substitution
device_name=$(basename $device_path)
```

**Array Expansion:**

```bash
# Proper array handling
files=("/etc/mediamtx/mediamtx.yml" "/etc/mediamtx/audio-devices.conf")

# Good: Quoted array expansion
for file in "${files[@]}"; do
    echo "$file"
done

# Bad: Unquoted array expansion
for file in ${files[@]}; do
    echo $file
done
```

---

## Function Standards

## Function Declaration

**Preferred Style:**

```bash
# Use 'function' keyword for clarity
function validate_device() {
    local device_path="$1"

    # Validate input
    if [[ -z "$device_path" ]]; then
        echo "Error: Device path required" >&2
        return 1
    fi

    # Function logic
    if [[ ! -e "$device_path" ]]; then
        echo "Error: Device not found: $device_path" >&2
        return 1
    fi

    return 0
}
```

## Function Documentation

**Standard Format:**

```bash
#
# Configures audio device encoding parameters
#
# Detects device capabilities and generates optimal FFmpeg
# encoding configuration based on hardware support.
#
# Arguments:
#   $1 - device_path: Full path to device (e.g., /dev/lyrebird-mic-1)
#   $2 - output_file: Configuration file path (optional)
#
# Returns:
#   0 - Success
#   1 - Invalid device path
#   2 - Configuration generation failed
#
# Environment:
#   LYREBIRD_HOME - Installation directory
#   LOG_LEVEL     - Logging verbosity (DEBUG, INFO, WARN, ERROR)
#
# Example:
#   configure_device "/dev/lyrebird-mic-1" "/tmp/config.env"
#
function configure_device() {
    local device_path="$1"
    local output_file="${2:-/tmp/device-config.env}"

    # Function implementation
}
```

## Parameter Handling

**Check Argument Count:**

```bash
function process_stream() {
    # Validate argument count
    if [[ $# -lt 2 ]]; then
        echo "Usage: process_stream <device> <stream_name>" >&2
        return 1
    fi

    local device_path="$1"
    local stream_name="$2"
    local codec="${3:-aac}"  # Optional with default

    # Function logic
}
```

**Named Parameters:**

```bash
# For complex functions, use named parameters
function create_stream() {
    local device=""
    local stream_name=""
    local sample_rate=48000
    local bitrate="128k"

    # Parse named parameters
    while [[ $# -gt 0 ]]; do
        case "$1" in
            --device)
                device="$2"
                shift 2
                ;;
            --name)
                stream_name="$2"
                shift 2
                ;;
            --sample-rate)
                sample_rate="$2"
                shift 2
                ;;
            --bitrate)
                bitrate="$2"
                shift 2
                ;;
            *)
                echo "Unknown option: $1" >&2
                return 1
                ;;
        esac
    done

    # Validate required parameters
    if [[ -z "$device" ]] || [[ -z "$stream_name" ]]; then
        echo "Error: --device and --name required" >&2
        return 1
    fi

    # Function logic
}

# Usage
create_stream --device "/dev/lyrebird-mic-1" --name "front-door" --bitrate "192k"
```

---

## Error Handling

## Exit Codes

**Standard Exit Codes:**

| Code | Meaning | Usage |
|------|---------|-------|
| 0 | Success | Normal completion |
| 1 | General error | Default error condition |
| 2 | Usage error | Invalid arguments |
| 3 | Configuration error | Config file issues |
| 4 | Permission denied | Insufficient privileges |
| 5 | Device error | Hardware not found |
| 6 | Service error | Systemd/service issues |
| 7 | Network error | Connection failures |
| 126 | Command not executable | Permission issues |
| 127 | Command not found | Missing dependency |
| 130 | Script terminated | Ctrl+C (SIGINT) |

**Implementation:**

```bash
# Define exit codes as constants
readonly EXIT_SUCCESS=0
readonly EXIT_ERROR=1
readonly EXIT_USAGE=2
readonly EXIT_CONFIG=3
readonly EXIT_PERMISSION=4
readonly EXIT_DEVICE=5

function main() {
    # Check for root privileges
    if [[ $EUID -ne 0 ]]; then
        echo "Error: Root privileges required" >&2
        exit $EXIT_PERMISSION
    fi

    # Validate configuration
    if [[ ! -f "$CONFIG_FILE" ]]; then
        echo "Error: Configuration not found: $CONFIG_FILE" >&2
        exit $EXIT_CONFIG
    fi

    # Normal execution
    process_devices || exit $EXIT_ERROR

    exit $EXIT_SUCCESS
}

main "$@"
```

## Error Messages

**Format:**

```bash
# Error messages to stderr
echo "Error: Failed to start stream: $stream_name" >&2

# Warning messages to stderr
echo "Warning: Device $device may not support 192k bitrate" >&2

# Info messages to stdout
echo "Stream started successfully: $stream_name"

# Debug messages (conditional)
[[ "$DEBUG" == "1" ]] && echo "Debug: FFmpeg command: $ffmpeg_cmd" >&2
```

**Structured Error Handling:**

```bash
function handle_error() {
    local error_message="$1"
    local exit_code="${2:-1}"

    echo "ERROR: $error_message" >&2
    echo "ERROR: Script: ${BASH_SOURCE[1]}" >&2
    echo "ERROR: Function: ${FUNCNAME[1]}" >&2
    echo "ERROR: Line: ${BASH_LINENO[0]}" >&2

    exit "$exit_code"
}

# Usage
[[ -f "$CONFIG_FILE" ]] || handle_error "Configuration not found: $CONFIG_FILE" $EXIT_CONFIG
```

## Trap Handlers

**Cleanup on Exit:**

```bash
# Temporary file cleanup
TEMP_FILES=()

function cleanup() {
    local exit_code=$?

    # Remove temporary files
    for temp_file in "${TEMP_FILES[@]}"; do
        [[ -f "$temp_file" ]] && rm -f "$temp_file"
    done

    # Log exit
    echo "Script exiting with code: $exit_code" >&2

    exit "$exit_code"
}

# Register cleanup handler
trap cleanup EXIT INT TERM

# Create temp file and register for cleanup
TEMP_CONFIG=$(mktemp)
TEMP_FILES+=("$TEMP_CONFIG")
```

---

## Logging Standards

## Log Levels

**Implementation:**

```bash
# Log level constants
readonly LOG_LEVEL_DEBUG=0
readonly LOG_LEVEL_INFO=1
readonly LOG_LEVEL_WARN=2
readonly LOG_LEVEL_ERROR=3

# Current log level (from environment or default)
CURRENT_LOG_LEVEL="${LOG_LEVEL:-$LOG_LEVEL_INFO}"

# Convert string to numeric level
case "$CURRENT_LOG_LEVEL" in
    DEBUG) CURRENT_LOG_LEVEL=$LOG_LEVEL_DEBUG ;;
    INFO)  CURRENT_LOG_LEVEL=$LOG_LEVEL_INFO ;;
    WARN)  CURRENT_LOG_LEVEL=$LOG_LEVEL_WARN ;;
    ERROR) CURRENT_LOG_LEVEL=$LOG_LEVEL_ERROR ;;
esac

# Logging functions
function log_debug() {
    [[ $CURRENT_LOG_LEVEL -le $LOG_LEVEL_DEBUG ]] && \
        echo "[DEBUG] $(date +'%Y-%m-%d %H:%M:%S') - $*" >&2
}

function log_info() {
    [[ $CURRENT_LOG_LEVEL -le $LOG_LEVEL_INFO ]] && \
        echo "[INFO]  $(date +'%Y-%m-%d %H:%M:%S') - $*"
}

function log_warn() {
    [[ $CURRENT_LOG_LEVEL -le $LOG_LEVEL_WARN ]] && \
        echo "[WARN]  $(date +'%Y-%m-%d %H:%M:%S') - $*" >&2
}

function log_error() {
    [[ $CURRENT_LOG_LEVEL -le $LOG_LEVEL_ERROR ]] && \
        echo "[ERROR] $(date +'%Y-%m-%d %H:%M:%S') - $*" >&2
}

# Usage
log_debug "Checking device: $device_path"
log_info "Stream started: $stream_name"
log_warn "High CPU usage detected: ${cpu_usage}%"
log_error "Failed to connect to MediaMTX API"
```

## Log File Management

**Rotating Logs:**

```bash
# Log file configuration
readonly LOG_DIR="/var/log/lyrebird"
readonly LOG_FILE="$LOG_DIR/lyrebird.log"
readonly LOG_MAX_SIZE=$((10 * 1024 * 1024))  # 10 MB

function rotate_log() {
    if [[ -f "$LOG_FILE" ]]; then
        local log_size=$(stat -f%z "$LOG_FILE" 2>/dev/null || stat -c%s "$LOG_FILE")

        if [[ $log_size -gt $LOG_MAX_SIZE ]]; then
            local timestamp=$(date +%Y%m%d-%H%M%S)
            mv "$LOG_FILE" "${LOG_FILE}.${timestamp}"
            gzip "${LOG_FILE}.${timestamp}"

            # Keep only last 10 rotated logs
            ls -t "${LOG_FILE}".*.gz | tail -n +11 | xargs rm -f
        fi
    fi
}

function log_to_file() {
    local message="$1"

    # Ensure log directory exists
    mkdir -p "$LOG_DIR"

    # Rotate if needed
    rotate_log

    # Write to log
    echo "$message" >> "$LOG_FILE"
}
```

---

## Configuration File Standards

## YAML Configuration

**MediaMTX Configuration (mediamtx.yml):**

```yaml
# Server configuration
logLevel: info
logDestinations: [stdout, file]
logFile: /var/log/mediamtx.log

# RTSP server
rtspAddress: :8554
protocols: [tcp]

# API configuration
api: yes
apiAddress: :9997

# Path configuration
paths:
  all:
    source: publisher
```

**Validation:**

```bash
function validate_yaml() {
    local yaml_file="$1"

    # Check file exists
    if [[ ! -f "$yaml_file" ]]; then
        log_error "YAML file not found: $yaml_file"
        return 1
    fi

    # Validate syntax (requires yq or python)
    if command -v yq &>/dev/null; then
        yq eval "$yaml_file" > /dev/null || return 1
    elif command -v python3 &>/dev/null; then
        python3 -c "import yaml; yaml.safe_load(open('$yaml_file'))" || return 1
    fi

    log_info "YAML validation passed: $yaml_file"
    return 0
}
```

## Bash Configuration Files

**Format (audio-devices.conf):**

```bash
# Audio Device Configuration
# Generated: 2025-01-15 14:30:00

# Device: Blue Yeti
DEVICE_Blue_Yeti_PATH="/dev/lyrebird-blue-yeti"
DEVICE_Blue_Yeti_SAMPLE_RATE=48000
DEVICE_Blue_Yeti_FORMAT="S16_LE"
DEVICE_Blue_Yeti_CHANNELS=2
DEVICE_Blue_Yeti_CODEC="aac"
DEVICE_Blue_Yeti_BITRATE="192k"

# Device: Front Door Mic
DEVICE_Front_Door_PATH="/dev/lyrebird-front-door"
DEVICE_Front_Door_SAMPLE_RATE=44100
DEVICE_Front_Door_FORMAT="S16_LE"
DEVICE_Front_Door_CHANNELS=1
DEVICE_Front_Door_CODEC="aac"
DEVICE_Front_Door_BITRATE="128k"
```

**Loading Configuration:**

```bash
function load_device_config() {
    local config_file="/etc/mediamtx/audio-devices.conf"

    # Validate file exists
    if [[ ! -f "$config_file" ]]; then
        log_error "Config file not found: $config_file"
        return 1
    fi

    # Source configuration
    # shellcheck source=/dev/null
    source "$config_file" || {
        log_error "Failed to load configuration: $config_file"
        return 1
    }

    log_info "Loaded device configuration: $config_file"
    return 0
}
```

---

## Security Best Practices

## Input Validation

**Sanitize User Input:**

```bash
function sanitize_device_name() {
    local input="$1"

    # Remove dangerous characters, allow only alphanumeric, hyphen, underscore
    local sanitized=$(echo "$input" | tr -cd '[:alnum:]-_')

    # Limit length
    sanitized="${sanitized:0:64}"

    # Ensure not empty after sanitization
    if [[ -z "$sanitized" ]]; then
        log_error "Invalid device name: $input"
        return 1
    fi

    echo "$sanitized"
}

# Usage
user_input="my-device; rm -rf /"
device_name=$(sanitize_device_name "$user_input") || exit 1
# Result: "my-device"
```

**Path Validation:**

```bash
function validate_device_path() {
    local device_path="$1"

    # Check path format
    if [[ ! "$device_path" =~ ^/dev/lyrebird-[a-zA-Z0-9_-]+$ ]]; then
        log_error "Invalid device path format: $device_path"
        return 1
    fi

    # Check path traversal
    if [[ "$device_path" == *".."* ]]; then
        log_error "Path traversal detected: $device_path"
        return 1
    fi

    # Verify device exists
    if [[ ! -e "$device_path" ]]; then
        log_error "Device not found: $device_path"
        return 1
    fi

    return 0
}
```

## Privilege Management

**Check Root Requirements:**

```bash
function require_root() {
    if [[ $EUID -ne 0 ]]; then
        echo "Error: This script must be run as root" >&2
        echo "Usage: sudo $0 $*" >&2
        exit $EXIT_PERMISSION
    fi
}

function drop_privileges() {
    local target_user="$1"

    if [[ $EUID -eq 0 ]]; then
        sudo -u "$target_user" "$@"
    else
        "$@"
    fi
}
```

**File Permissions:**

```bash
# Secure file creation
function create_config_file() {
    local config_file="$1"
    local config_content="$2"

    # Create with restricted permissions
    (umask 077 && echo "$config_content" > "$config_file")

    # Set explicit permissions
    chmod 644 "$config_file"
    chown root:root "$config_file"

    log_info "Created config file: $config_file"
}

# Secure directory creation
function create_secure_directory() {
    local dir_path="$1"

    mkdir -p "$dir_path"
    chmod 755 "$dir_path"
    chown root:root "$dir_path"
}
```

## Secure Temporary Files

**Temp File Handling:**

```bash
# Create secure temporary file
TEMP_FILE=$(mktemp /tmp/lyrebird.XXXXXX) || {
    log_error "Failed to create temporary file"
    exit 1
}

# Set restrictive permissions
chmod 600 "$TEMP_FILE"

# Ensure cleanup
trap 'rm -f "$TEMP_FILE"' EXIT

# Use temporary file
echo "sensitive data" > "$TEMP_FILE"
process_file "$TEMP_FILE"
```

---

## Code Review Checklist

Before submitting code for review:

## Shellcheck Validation

- [ ] All scripts pass `shellcheck` with zero warnings
- [ ] SC2086 (Quote variables) addressed
- [ ] SC2164 (Check cd return value) addressed
- [ ] SC2155 (Separate declaration and assignment) addressed

## Documentation

- [ ] Function headers complete with parameters and returns
- [ ] Inline comments explain complex logic
- [ ] README updated if new features added
- [ ] Configuration examples provided

## Error Handling

- [ ] All commands checked for success
- [ ] Appropriate exit codes used
- [ ] Error messages go to stderr
- [ ] Cleanup handlers registered

## Security

- [ ] User input validated
- [ ] File paths sanitized
- [ ] Appropriate permissions set
- [ ] No hardcoded credentials

## Testing

- [ ] Tested with multiple USB devices
- [ ] Edge cases handled
- [ ] Error conditions tested
- [ ] Performance acceptable

## Code Style

- [ ] Consistent naming conventions
- [ ] Proper variable quoting
- [ ] Functions documented
- [ ] Modular design

---

## Related Documentation

- [Development Setup](development-setup.md) - Development environment configuration
- [Testing](testing.md) - Testing procedures and guidelines
- [Architecture](../advanced/architecture.md) - System architecture details
- [Configuration Files](../reference/configuration-files.md) - Configuration reference

---

## Next Steps

<div class="grid" markdown>

<div markdown>
## Testing Guidelines
Comprehensive testing procedures

[Testing Guidelines →](testing.md)
</div>

<div markdown>
## Contributing Overview
Contributing guidelines and workflows

[Contributing Overview →](index.md)
</div>

</div>
