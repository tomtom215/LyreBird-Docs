# Exit Codes

Complete reference for LyreBirdAudio command exit codes and return values.

---

## Overview

LyreBirdAudio commands use standardized exit codes for automation and scripting. Exit codes indicate success, warnings, failures, or specific error conditions.

**Standard Exit Codes:**

| Code | Status | Meaning |
|------|--------|---------|
| 0 | Success | Operation completed successfully |
| 1 | Warning/Minor Error | Operation completed with warnings |
| 2 | Failure | Operation failed, intervention required |
| 127 | Command Error | Command not found or execution error |

---

## lyrebird-diagnostics.sh

**Exit Codes:**

| Code | Status | Description | Action Required |
|------|--------|-------------|-----------------|
| 0 | All Passed | All diagnostic checks passed | None |
| 1 | Warnings | Some checks have warnings, system operational | Review warnings |
| 2 | Failures | Critical checks failed | Immediate attention |
| 127 | Error | Diagnostic tool error (missing dependencies) | Fix tool prerequisites |

**Example Usage:**

```bash
#!/bin/bash
sudo ./lyrebird-diagnostics.sh

case $? in
    0)
        echo "System healthy"
        ;;
    1)
        echo "System has warnings"
        # Send notification but continue operation
        ;;
    2)
        echo "Critical failures detected"
        # Alert administrator
        exit 1
        ;;
    127)
        echo "Diagnostic tool error"
        exit 1
        ;;
esac
```

**Automation Example:**

```bash
# Cron job with exit code handling
0 2 * * * /opt/lyrebird/lyrebird-diagnostics.sh || \
    echo "LyreBird diagnostic failure" | \
    mail -s "Alert: LyreBird Health Check Failed" admin@example.com
```

---

## mediamtx-stream-manager.sh

**Exit Codes:**

| Code | Status | Description |
|------|--------|-------------|
| 0 | Success | All streams started/stopped successfully |
| 1 | Partial Success | Some streams failed but others succeeded |
| 2 | Complete Failure | All operations failed |
| 3 | Configuration Error | Invalid configuration detected |
| 4 | Service Error | MediaMTX service not running |
| 5 | Lock Failed | Another instance is already running |
| 6 | No USB Devices | No USB audio devices detected |
| 7 | MediaMTX Down | MediaMTX server is not responding |
| 10 | Monitor Degraded | Monitoring active but with degraded state |

**Command-Specific Codes:**

### start

```bash
sudo ./mediamtx-stream-manager.sh start
echo $?  # Exit code
```

- `0`: All streams started successfully
- `1`: Some streams failed to start
- `2`: All streams failed to start
- `3`: Configuration file missing or invalid
- `4`: MediaMTX service not running

### stop

```bash
sudo ./mediamtx-stream-manager.sh stop
echo $?  # Exit code
```

- `0`: All streams stopped cleanly
- `1`: Some streams required force kill
- `2`: Failed to stop streams

### status

```bash
sudo ./mediamtx-stream-manager.sh status
echo $?  # Exit code
```

- `0`: All configured streams are running
- `1`: Some streams are down
- `2`: All streams are down or no streams configured

**Example Usage:**

```bash
#!/bin/bash
sudo ./mediamtx-stream-manager.sh start

if [ $? -eq 0 ]; then
    echo "All streams started successfully"
elif [ $? -eq 1 ]; then
    echo "Some streams failed - check logs"
    # Continue but log warning
elif [ $? -eq 4 ]; then
    echo "MediaMTX not running - starting service"
    sudo systemctl start mediamtx
    sudo ./mediamtx-stream-manager.sh start
else
    echo "Critical failure - aborting"
    exit 1
fi
```

---

## lyrebird-mic-check.sh

**Exit Codes:**

| Code | Status | Description |
|------|--------|-------------|
| 0 | Success | Operation completed successfully |
| 1 | Validation Failed | Configuration validation found errors |
| 2 | No Devices | No audio devices detected |
| 3 | Permission Error | Insufficient permissions |

**Mode-Specific Codes:**

### -g (Generate)

- `0`: Configuration generated successfully
- `2`: No devices found
- `3`: Cannot write configuration file

### -V (Validate)

- `0`: Configuration is valid
- `1`: Configuration has errors or warnings
- `3`: Configuration file not found

**Example Usage:**

```bash
#!/bin/bash
sudo ./lyrebird-mic-check.sh -g --quality=normal

if [ $? -eq 2 ]; then
    echo "ERROR: No USB audio devices detected"
    echo "Please connect USB microphones and try again"
    exit 1
fi
```

---

## usb-audio-mapper.sh

**Exit Codes:**

| Code | Status | Description |
|------|--------|-------------|
| 0 | Success | Rules generated and applied successfully |
| 1 | Warning | Rules generated but udev reload had warnings |
| 2 | No Devices | No USB audio devices found |
| 3 | Permission Error | Cannot write to udev rules directory |

**Example Usage:**

```bash
#!/bin/bash
sudo ./usb-audio-mapper.sh

case $? in
    0)
        echo "USB mapping successful"
        ;;
    2)
        echo "No USB audio devices detected"
        exit 1
        ;;
    3)
        echo "Permission error - run with sudo"
        exit 1
        ;;
esac
```

---

## install_mediamtx.sh

**Exit Codes:**

| Code | Status | Description |
|------|--------|-------------|
| 0 | Success | MediaMTX installed and started successfully |
| 1 | Download Failed | Cannot download MediaMTX binary |
| 2 | Installation Failed | Cannot install files or create directories |
| 3 | Service Failed | Cannot start MediaMTX service |

**Example Usage:**

```bash
#!/bin/bash
sudo ./install_mediamtx.sh

if [ $? -ne 0 ]; then
    echo "MediaMTX installation failed"
    # Cleanup partial installation
    sudo rm -f /usr/local/bin/mediamtx
    exit 1
fi
```

---

## lyrebird-updater.sh

**Exit Codes:**

| Code | Status | Description |
|------|--------|-------------|
| 0 | Success | Update completed successfully |
| 1 | General Error | Generic error occurred |
| 2 | Prerequisites Missing | Required tools not installed (git, etc.) |
| 3 | Not a Git Repository | Directory is not a git repository |
| 4 | No Remote Configured | Git remote not configured |
| 5 | Permission Denied | Insufficient permissions |
| 7 | Locked | Another instance is already running |
| 8 | Bad Git State | Git repository in inconsistent state |
| 9 | User Aborted | Operation cancelled by user |

**Example Usage:**

```bash
#!/bin/bash
sudo ./lyrebird-updater.sh --update

case $? in
    0)
        echo "Update successful"
        ;;
    1)
        echo "General error occurred"
        ;;
    2)
        echo "Missing prerequisites - install git"
        ;;
    3)
        echo "Not a git repository"
        ;;
    5)
        echo "Permission denied - run with sudo"
        ;;
    7)
        echo "Another update is already running"
        ;;
    9)
        echo "Update cancelled by user"
        ;;
esac
```

---

## Custom Script Exit Codes

### Best Practices

When creating custom LyreBird scripts, follow these conventions:

```bash
#!/bin/bash

# Standard exit codes
SUCCESS=0
WARNING=1
FAILURE=2

# Custom error codes (10-99)
CONFIG_ERROR=10
PERMISSION_ERROR=11
RESOURCE_ERROR=12

# Example usage
if [ ! -f "/etc/mediamtx/audio-devices.conf" ]; then
    echo "ERROR: Configuration file not found"
    exit $CONFIG_ERROR
fi

if [ ! -w "/var/log/lyrebird" ]; then
    echo "ERROR: Cannot write to log directory"
    exit $PERMISSION_ERROR
fi

# Success
echo "Operation completed"
exit $SUCCESS
```

### Handling Multiple Commands

```bash
#!/bin/bash

# Initialize exit code
EXIT_CODE=0

# Run multiple operations
sudo ./usb-audio-mapper.sh
if [ $? -ne 0 ]; then
    EXIT_CODE=1
fi

sudo ./lyrebird-mic-check.sh -g
if [ $? -ne 0 ]; then
    EXIT_CODE=1
fi

sudo ./mediamtx-stream-manager.sh start
if [ $? -ne 0 ]; then
    EXIT_CODE=2  # Critical failure
fi

exit $EXIT_CODE
```

---

## Monitoring and Alerting

### Nagios/Icinga Integration

```bash
#!/bin/bash
# Nagios check plugin

sudo ./lyrebird-diagnostics.sh > /dev/null 2>&1
RESULT=$?

case $RESULT in
    0)
        echo "OK: All LyreBird checks passed"
        exit 0  # Nagios OK
        ;;
    1)
        echo "WARNING: LyreBird has warnings"
        exit 1  # Nagios WARNING
        ;;
    2)
        echo "CRITICAL: LyreBird has failures"
        exit 2  # Nagios CRITICAL
        ;;
    *)
        echo "UNKNOWN: Diagnostic tool error"
        exit 3  # Nagios UNKNOWN
        ;;
esac
```

### Systemd Service Monitoring

```ini
# /etc/systemd/system/lyrebird-health-check.service

[Service]
Type=oneshot
ExecStart=/opt/lyrebird/lyrebird-diagnostics.sh
# Only restart if exit code is 2 (failure)
Restart=on-failure
# Don't restart on warnings (exit code 1)
RestartPreventExitStatus=1
```

### Cron Job with Email Alerts

```bash
# /etc/cron.daily/lyrebird-health-check

#!/bin/bash
OUTPUT=$(sudo /opt/lyrebird/lyrebird-diagnostics.sh 2>&1)
EXIT_CODE=$?

if [ $EXIT_CODE -eq 2 ]; then
    # Critical failure - send email
    echo "$OUTPUT" | mail -s "CRITICAL: LyreBird Health Check Failed" \
        admin@example.com
elif [ $EXIT_CODE -eq 1 ]; then
    # Warning - log but don't email
    echo "$(date): WARNING - $OUTPUT" >> /var/log/lyrebird-health-warnings.log
fi

exit $EXIT_CODE
```

---

## Exit Code Reference Summary

### Quick Reference Table

| Command | Success (0) | Warning (1) | Failure (2) | Other |
|---------|-------------|-------------|-------------|-------|
| `lyrebird-diagnostics.sh` | All passed | Warnings present | Failures detected | 127: Tool error |
| `mediamtx-stream-manager.sh` | All streams OK | Partial success | Complete failure | 3: Config error, 4: Service error, 5: Lock failed, 6: No USB devices, 7: MediaMTX down, 10: Monitor degraded |
| `lyrebird-mic-check.sh` | Success | Validation failed | No devices | 3: Permission error |
| `usb-audio-mapper.sh` | Success | Warning | No devices | 3: Permission error |
| `install_mediamtx.sh` | Success | N/A | Install failed | 3: Service failed |
| `lyrebird-updater.sh` | Success | General error | Prerequisites missing | 3: Not git repo, 4: No remote, 5: Permission denied, 7: Locked, 8: Bad git state, 9: User aborted |

---

## Related Documentation

- [Command Reference](command-reference.md) - Command usage
- [Diagnostics & Monitoring](../advanced/diagnostics-monitoring.md) - Health checks
- [Troubleshooting](../advanced/troubleshooting.md) - Error resolution
- [Environment Variables](environment-variables.md) - Runtime configuration

---

## Next Steps

<div class="grid" markdown>

<div markdown>
### Log Files
Log locations, formats, and analysis

[Log Files →](log-files.md)
</div>

<div markdown>
### Command Reference
All available commands and utilities

[Command Reference →](command-reference.md)
</div>

</div>
