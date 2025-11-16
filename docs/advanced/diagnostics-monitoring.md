# Diagnostics & Monitoring

Comprehensive guide to system health monitoring, diagnostics, and automated testing for LyreBirdAudio.

---

## Overview

LyreBirdAudio provides extensive diagnostic and monitoring capabilities to ensure system health, detect issues early, and provide actionable troubleshooting information. The diagnostic system performs automated checks across all system components and provides detailed reporting.

This guide covers:

- Diagnostic tool modes and usage
- 24+ automated health checks
- Exit codes and status reporting
- Continuous monitoring strategies
- Integration with monitoring systems
- Log analysis and interpretation

---

## Diagnostic Tool

## Overview

The `lyrebird-diagnostics.sh` script performs comprehensive system health checks:

```bash
# Quick health check
sudo ./lyrebird-diagnostics.sh

# Verbose output with detailed information
sudo ./lyrebird-diagnostics.sh --verbose

# Export detailed report to file
sudo ./lyrebird-diagnostics.sh --export=/tmp/diagnostic-report.txt
```

**Key Features:**

- 24+ automated checks across all components
- Color-coded pass/warning/fail indicators
- Detailed error messages and remediation steps
- Machine-readable exit codes
- Export capability for support tickets

## Diagnostic Modes

**Quick Mode (Default):**

```bash
sudo ./lyrebird-diagnostics.sh
```

- Runs all critical checks
- Color-coded output
- Summary of issues found
- Recommended actions
- Completes in 10-30 seconds

**Verbose Mode:**

```bash
sudo ./lyrebird-diagnostics.sh --verbose
```

- All quick mode checks
- Detailed test output
- Extended device information
- Configuration validation
- Log file excerpts
- Completes in 30-60 seconds

**Export Mode:**

```bash
sudo ./lyrebird-diagnostics.sh --export=/path/to/report.txt
```

- Complete diagnostic report
- All verbose information
- Timestamp and system info
- Suitable for support tickets
- Machine-readable format

**Debug Mode:**

```bash
sudo DIAGNOSTIC_DEBUG=1 ./lyrebird-diagnostics.sh --verbose
```

- All verbose mode output
- Internal diagnostic logic
- Script execution trace
- For troubleshooting the diagnostic tool itself

---

## Diagnostic Checks

## System Checks

**1. Operating System Compatibility**

Validates OS is supported:

- Check: Debian-based distribution (Ubuntu, Debian, etc.)
- Required: Linux kernel 4.4+
- Validates: Package manager availability

**Example Output:**

```text
[PASS] Operating System: Ubuntu 22.04.3 LTS
[PASS] Kernel Version: 5.15.0-89-generic
[PASS] Architecture: x86_64
```

**2. Required Packages Installed**

Verifies all dependencies present:

- FFmpeg (version 4.4+)
- alsa-utils
- curl
- jq

**Example Output:**

```text
[PASS] FFmpeg: version 4.4.2 (required: 4.4+)
[PASS] alsa-utils: installed
[PASS] curl: installed
[PASS] jq: version 1.6 installed
```

**3. File Permissions**

Checks access to required directories and files:

- `/etc/mediamtx/` readable
- `/var/log/lyrebird/` writable
- Script files executable

**Example Output:**

```text
[PASS] Configuration directory accessible: /etc/mediamtx/
[PASS] Log directory writable: /var/log/lyrebird/
[PASS] Script permissions correct
```

## Device Checks

**4. USB Audio Devices Detected**

Scans for connected USB audio devices:

- Queries `/proc/asound/cards`
- Uses `lsusb` for USB enumeration
- Validates ALSA recognition

**Example Output:**

```text
[PASS] Found 2 USB audio devices:
  - Card 0: Blue_Yeti
  - Card 1: USB_Microphone
```

**5. udev Rules Configured**

Validates USB persistence configuration:

- Checks `/etc/udev/rules.d/99-usb-soundcards.rules` exists
- Verifies rule syntax
- Tests symlink creation

**Example Output:**

```text
[PASS] udev rules file exists: /etc/udev/rules.d/99-usb-soundcards.rules
[PASS] Rules syntax valid
[PASS] Symlinks created: 2 devices
```

**6. Device Persistence Working**

Tests that device names remain consistent:

- Compares device names to udev rules
- Checks symlink targets
- Validates card number assignment

**Example Output:**

```text
[PASS] Device persistence verified:
  - Blue_Yeti: hw:0 (persistent)
  - USB_Microphone: hw:1 (persistent)
```

**7. Device Access Permissions**

Validates user can access audio devices:

- Checks audio group membership
- Validates `/dev/snd/` permissions
- Tests device open capability

**Example Output:**

```text
[PASS] Current user in audio group
[PASS] Device permissions correct (660, group: audio)
[PASS] Can access all audio devices
```

## Service Checks

**8. MediaMTX Service Running**

Checks MediaMTX service status:

- Queries systemd service status
- Verifies process is running
- Checks service uptime

**Example Output:**

```text
[PASS] MediaMTX service: active (running)
[PASS] Process ID: 12340
[PASS] Uptime: 2 days, 5 hours
```

**9. MediaMTX API Accessible**

Tests HTTP API connectivity:

- Connects to port 9997
- Queries `/v3/paths/list` endpoint
- Validates JSON response

**Example Output:**

```text
[PASS] MediaMTX API responding on port 9997
[PASS] API version: v3
[PASS] Endpoint accessible: /v3/paths/list
```

**10. Stream Manager Status**

Checks stream manager service (if installed):

- Queries systemd status
- Verifies monitoring active
- Checks for recent errors

**Example Output:**

```text
[PASS] Stream Manager service: active (running)
[PASS] Monitoring mode: enabled
[PASS] No recent errors
```

## Stream Checks

**11. RTSP Port Listening**

Validates MediaMTX RTSP server:

- Checks port 8554 open
- Verifies MediaMTX bound to port
- Tests connectivity

**Example Output:**

```text
[PASS] RTSP port 8554: listening
[PASS] Process: mediamtx (PID: 12340)
```

**12. Active Streams Detected**

Enumerates currently active streams:

- Queries MediaMTX API
- Lists stream paths
- Reports stream count

**Example Output:**

```text
[PASS] Found 2 active streams:
  - Blue_Yeti
  - USB_Microphone
```

**13. Stream Health**

Tests stream connectivity and data flow:

- Connects to each RTSP stream
- Validates stream metadata
- Checks for active data

**Example Output:**

```text
[PASS] Blue_Yeti: healthy (codec: opus, rate: 48000)
[PASS] USB_Microphone: healthy (codec: opus, rate: 48000)
```

**14. Configuration Validation**

Validates device configuration file:

- Checks `/etc/mediamtx/audio-devices.conf` exists
- Parses configuration syntax
- Validates codec and rate values

**Example Output:**

```text
[PASS] Configuration file exists
[PASS] Configuration syntax valid
[PASS] 2 devices configured
```

## Resource Checks

**15. CPU Usage**

Monitors system CPU utilization:

- Overall system load
- Per-stream FFmpeg usage
- Threshold warnings (>80%)

**Example Output:**

```text
[PASS] System CPU usage: 25.3% (threshold: 80%)
[PASS] FFmpeg processes: 6.5% total
  - Blue_Yeti: 3.2%
  - USB_Microphone: 3.3%
```

**16. Memory Available**

Checks memory availability:

- Free memory
- Swap usage
- Threshold warnings (<200MB free)

**Example Output:**

```text
[PASS] Memory available: 2.4 GB (threshold: 200 MB)
[PASS] Swap usage: 0 MB
[PASS] Memory per stream average: 48 MB
```

**17. File Descriptors**

Monitors file descriptor usage:

- Current count
- System limits
- Threshold warnings (>80% of limit)

**Example Output:**

```text
[PASS] File descriptors: 245 / 1024 (24%)
[PASS] Within safe limits
```

**18. Disk Space**

Checks disk space for logs:

- `/var/log/lyrebird/` space
- `/var/log/` space
- Threshold warnings (<1GB free)

**Example Output:**

```text
[PASS] Log directory space: 15.2 GB available
[PASS] Root filesystem: 42.8 GB available
```

## Network Checks

**19. Network Connectivity**

Tests local network connectivity:

- Localhost connectivity
- Network interface status
- IP address assignment

**Example Output:**

```text
[PASS] Localhost connectivity: OK
[PASS] Network interfaces: 2 active
[PASS] Primary IP: 192.168.1.100
```

**20. RTSP Connectivity**

Tests RTSP stream access:

- Local RTSP connection
- Stream playback test
- Latency measurement

**Example Output:**

```text
[PASS] Local RTSP connectivity: OK
[PASS] Stream connection latency: 15ms
```

**21. Firewall Status**

Checks firewall configuration:

- Required ports open (8554, 9997)
- Firewall service status
- Rule validation

**Example Output:**

```text
[PASS] Firewall: active
[PASS] RTSP port 8554: allowed
[PASS] API port 9997: allowed
```

## Log Checks

**22. Log Files Accessible**

Validates log file access:

- MediaMTX logs readable
- Stream manager logs readable
- Device logs accessible

**Example Output:**

```text
[PASS] MediaMTX logs: accessible
[PASS] Stream manager logs: accessible
[PASS] Device logs: 2 files accessible
```

**23. Recent Errors**

Scans logs for recent errors:

- Last 24 hours of logs
- Error and warning counts
- Critical error highlighting

**Example Output:**

```text
[PASS] No critical errors in last 24 hours
[WARN] 3 warnings found (USB resets)
```

**24. Log Rotation**

Checks log rotation configuration:

- Logrotate configuration
- Log file sizes
- Rotation schedule

**Example Output:**

```text
[PASS] Log rotation: configured
[PASS] Largest log file: 2.4 MB
[PASS] Rotation schedule: weekly
```

---

## Exit Codes

The diagnostic tool returns specific exit codes for automation:

| Exit Code | Status | Description |
|-----------|--------|-------------|
| 0 | Success | All checks passed |
| 1 | Warnings | Some checks have warnings, system operational |
| 2 | Failures | Critical checks failed, intervention required |
| 127 | Error | Diagnostic tool error (missing dependencies, etc.) |

**Usage in Scripts:**

```bash
#!/bin/bash
sudo ./lyrebird-diagnostics.sh

case $? in
    0)
        echo "System healthy"
        ;;
    1)
        echo "System operational with warnings"
        # Send notification
        ;;
    2)
        echo "Critical issues detected"
        # Alert administrator
        exit 1
        ;;
    127)
        echo "Diagnostic tool error"
        exit 1
        ;;
esac
```

---

## Continuous Monitoring

## Stream Manager Monitoring

The stream manager provides continuous monitoring:

```bash
# Real-time monitoring
sudo ./mediamtx-stream-manager.sh monitor
```

**Displays:**

- Stream health status
- CPU usage per stream
- Memory consumption
- File descriptor count
- Automatic recovery actions

**Example Output:**

```text
Stream Monitor - Updating every 5 seconds (Ctrl+C to exit)
===========================================================
[10:30:15] Blue_Yeti: HEALTHY (CPU: 3.2%, MEM: 45 MB)
[10:30:15] USB_Microphone: HEALTHY (CPU: 2.8%, MEM: 42 MB)
[10:30:15] System: FD: 245/1024, Total CPU: 6.0%

Resource Summary:
  Total CPU: 6.0%
  Total Memory: 687 MB
  File Descriptors: 245 / 1024
  Active Streams: 2
  Failed Streams: 0
```

## Systemd Service Monitoring

For production environments, monitor via systemd:

```bash
# Check service status
sudo systemctl status mediamtx-audio

# Follow logs in real-time
sudo journalctl -u mediamtx-audio -f

# Filter for errors only
sudo journalctl -u mediamtx-audio -p err

# View logs since boot
sudo journalctl -u mediamtx-audio -b
```

## Automated Health Checks

Schedule regular diagnostics:

```bash
# Create cron job for daily diagnostics
sudo crontab -e

# Add line for daily check at 2 AM
0 2 * * * /path/to/lyrebird-diagnostics.sh --export=/var/log/lyrebird/daily-diagnostic.txt

# Weekly detailed check
0 3 * * 0 /path/to/lyrebird-diagnostics.sh --verbose --export=/var/log/lyrebird/weekly-diagnostic.txt
```

**Alert on Failures:**

```bash
#!/bin/bash
# /usr/local/bin/lyrebird-health-check.sh

sudo /path/to/lyrebird-diagnostics.sh

if [ $? -eq 2 ]; then
    # Critical failure detected
    echo "LyreBird diagnostic failure" | \
        mail -s "ALERT: LyreBird System Failure" admin@example.com
fi
```

---

## Integration with Monitoring Systems

## Prometheus Integration

Export metrics for Prometheus:

```bash
# Create metrics exporter script
# /usr/local/bin/lyrebird-metrics-exporter.sh

#!/bin/bash
# Export LyreBird metrics in Prometheus format

echo "# HELP lyrebird_streams_active Number of active streams"
echo "# TYPE lyrebird_streams_active gauge"
ACTIVE=$(curl -s http://localhost:9997/v3/paths/list | jq '.items | length')
echo "lyrebird_streams_active $ACTIVE"

echo "# HELP lyrebird_cpu_usage CPU usage percentage"
echo "# TYPE lyrebird_cpu_usage gauge"
CPU=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)
echo "lyrebird_cpu_usage $CPU"

echo "# HELP lyrebird_memory_bytes Memory usage in bytes"
echo "# TYPE lyrebird_memory_bytes gauge"
MEM=$(free -b | awk 'NR==2{print $3}')
echo "lyrebird_memory_bytes $MEM"
```

**Configure as Prometheus Target:**

```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'lyrebird'
    static_configs:
      - targets: ['localhost:9100']
    metrics_path: '/lyrebird-metrics'
```

## Grafana Dashboard

Create visualization dashboard:

**Metrics to Display:**

- Active stream count over time
- CPU usage per stream
- Memory consumption
- Stream uptime
- Error rate
- Recovery events

**Example Panel Queries:**

```promql
# Active streams
lyrebird_streams_active

# CPU usage
lyrebird_cpu_usage

# Memory usage
lyrebird_memory_bytes / 1024 / 1024

# Stream health
up{job="lyrebird"}
```

## Nagios/Icinga Integration

Create Nagios check plugin:

```bash
#!/bin/bash
# /usr/lib/nagios/plugins/check_lyrebird.sh

sudo /path/to/lyrebird-diagnostics.sh > /dev/null 2>&1
RESULT=$?

case $RESULT in
    0)
        echo "OK: All LyreBird checks passed"
        exit 0
        ;;
    1)
        echo "WARNING: LyreBird has warnings"
        exit 1
        ;;
    2)
        echo "CRITICAL: LyreBird has failures"
        exit 2
        ;;
    *)
        echo "UNKNOWN: Diagnostic tool error"
        exit 3
        ;;
esac
```

**Nagios Configuration:**

```cfg
define command{
    command_name check_lyrebird
    command_line /usr/lib/nagios/plugins/check_lyrebird.sh
}

define service{
    use                 generic-service
    host_name           streaming-server
    service_description LyreBird Health
    check_command       check_lyrebird
    check_interval      5
}
```

---

## Log Analysis

## MediaMTX Logs

Location: `/var/log/mediamtx.out` (or via journalctl)

**Access:**

```bash
# View recent logs
sudo journalctl -u mediamtx -n 100

# Follow logs
sudo journalctl -u mediamtx -f

# Filter by priority
sudo journalctl -u mediamtx -p err
```

**Key Log Patterns:**

```log
# Stream connection
"[path Blue_Yeti] opened"

# Client connection
"[conn] [c->s] [client_ip] reader opened"

# Stream disconnection
"[path Blue_Yeti] closed"

# Errors
"[path Blue_Yeti] error"
```

## Stream Manager Logs

Location: `/var/log/mediamtx-stream-manager.log`

**Access:**

```bash
# View recent logs
sudo tail -f /var/log/mediamtx-stream-manager.log

# Search for errors
sudo grep -i error /var/log/mediamtx-stream-manager.log

# View recovery events
sudo grep -i "restart" /var/log/mediamtx-stream-manager.log
```

**Key Log Patterns:**

```log
# Stream started
"Started stream: Blue_Yeti (PID: 12345)"

# Stream failed
"[ERROR] Stream Blue_Yeti failed"

# Recovery attempt
"[INFO] Attempting restart (1/5) in 10 seconds"

# Recovery success
"[INFO] Restarted Blue_Yeti successfully"
```

## FFmpeg Device Logs

Location: `/var/log/lyrebird/<device-name>.log`

**Access:**

```bash
# View specific device log
sudo tail -f /var/log/lyrebird/Blue_Yeti.log

# Check all device logs
sudo tail -f /var/log/lyrebird/*.log
```

**Key Log Patterns:**

```log
# Successful capture
"[alsa] Estimating duration from bitrate"

# Buffer issues
"ALSA buffer xrun"

# Device disconnection
"Device or resource busy"

# Encoding stats
"frame= 1000 fps= 24 size= 512kB time=00:00:41.66"
```

## Diagnostic Logs

Location: `/var/log/lyrebird-diagnostics.log`

**Access:**

```bash
# View diagnostic history
sudo cat /var/log/lyrebird-diagnostics.log

# Recent failures
sudo grep FAIL /var/log/lyrebird-diagnostics.log | tail -20
```

---

## Performance Metrics Collection

## Resource Metrics

Collect system metrics over time:

```bash
#!/bin/bash
# /usr/local/bin/lyrebird-collect-metrics.sh

LOGFILE="/var/log/lyrebird/metrics.csv"

# Create header if file doesn't exist
if [ ! -f "$LOGFILE" ]; then
    echo "Timestamp,CPU%,MemoryMB,FD_Count,Active_Streams" > "$LOGFILE"
fi

# Collect metrics
TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S')
CPU=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)
MEM=$(free -m | awk 'NR==2{print $3}')
FD=$(lsof | wc -l)
STREAMS=$(curl -s http://localhost:9997/v3/paths/list | jq '.items | length')

# Append to log
echo "$TIMESTAMP,$CPU,$MEM,$FD,$STREAMS" >> "$LOGFILE"
```

**Schedule Collection:**

```bash
# Run every minute
* * * * * /usr/local/bin/lyrebird-collect-metrics.sh
```

## Stream Quality Metrics

Monitor stream quality:

```bash
#!/bin/bash
# Test stream quality

STREAM_URL="rtsp://localhost:8554/Blue_Yeti"
DURATION=10  # seconds

# Capture stream sample
ffmpeg -i "$STREAM_URL" -t $DURATION -f null - 2>&1 | \
    grep -E "bitrate|fps|size"
```

---

## Best Practices

## Regular Diagnostics

1. **Daily Quick Checks**
   ```bash
   # Run daily quick diagnostic
   sudo ./lyrebird-diagnostics.sh
   ```

2. **Weekly Detailed Analysis**
   ```bash
   # Weekly verbose check with export
   sudo ./lyrebird-diagnostics.sh --verbose --export=/var/log/lyrebird/weekly.txt
   ```

3. **Monthly Deep Dive**
   - Review all logs
   - Analyze metric trends
   - Capacity planning review

## Monitoring Strategy

1. **Real-Time Monitoring**
   - Use stream manager monitor mode
   - Watch for immediate issues
   - Observe recovery behavior

2. **Historical Analysis**
   - Collect metrics over time
   - Identify trends
   - Plan capacity upgrades

3. **Alerting**
   - Set up critical alerts
   - Define escalation procedures
   - Test alert paths regularly

## Log Management

1. **Log Rotation**
   ```bash
   # Configure logrotate
   # /etc/logrotate.d/lyrebird
   /var/log/lyrebird/*.log {
       daily
       rotate 7
       compress
       delaycompress
       missingok
       notifempty
   }
   ```

2. **Log Retention**
   - Keep 7 days of detailed logs
   - Archive monthly summaries
   - Purge old diagnostics

3. **Log Analysis**
   - Regular pattern analysis
   - Error trend identification
   - Performance baseline establishment

---

## Related Documentation

- [Troubleshooting](troubleshooting.md) - Issue resolution
- [Performance Tuning](performance.md) - Optimization
- [Log Files](../reference/log-files.md) - Log reference
- [Command Reference](../reference/command-reference.md) - Diagnostic commands
- [Exit Codes](../reference/exit-codes.md) - Return codes

---

## Next Steps

<div class="grid" markdown>

<div markdown>
## Custom Integration
Integrate LyreBirdAudio with external systems

[Custom Integration →](custom-integration.md)
</div>

<div markdown>
## Troubleshooting
Resolve common issues and problems

[Troubleshooting →](troubleshooting.md)
</div>

</div>
