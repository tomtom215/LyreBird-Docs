# Log Files

Complete reference for all LyreBirdAudio log files, locations, formats, and analysis techniques.

---

## Overview

LyreBirdAudio generates logs across multiple components for debugging, monitoring, and troubleshooting. Understanding log locations and formats is essential for effective system management.

**Log File Locations:**

| Component | Log Location | Format |
|-----------|--------------|--------|
| MediaMTX | `/var/log/mediamtx.out` or journalctl | Text |
| Stream Manager | `/var/log/mediamtx-stream-manager.log` | Text |
| FFmpeg (per-device) | `/var/log/lyrebird/<device-name>.log` | Text |
| Diagnostics | `/var/log/lyrebird-diagnostics.log` | Text |
| System Services | journalctl | Systemd journal |

---

## MediaMTX Logs

### Location

**File:** `/var/log/mediamtx.out` (if using file logging)

**Journalctl:** `journalctl -u mediamtx`

### Access

```bash
# View recent logs
sudo journalctl -u mediamtx -n 100

# Follow logs in real-time
sudo journalctl -u mediamtx -f

# Filter by priority
sudo journalctl -u mediamtx -p err

# View logs since boot
sudo journalctl -u mediamtx -b

# Export logs to file
sudo journalctl -u mediamtx --since today > /tmp/mediamtx-today.log
```

### Log Format

```
YYYY-MM-DD HH:MM:SS [LEVEL] [component] message
```

**Example Entries:**

```
2025-11-15 10:00:00 INF [RTSP] [listener] opened
2025-11-15 10:00:01 INF [path Blue_Yeti] opened
2025-11-15 10:00:02 INF [conn] [c->s] 192.168.1.100:54321 reader opened
2025-11-15 10:00:10 INF [path Blue_Yeti] [c->s] tracks: 1 audio (Opus)
2025-11-15 10:05:00 WRN [path Blue_Yeti] [c->s] no packets received, disconnecting reader
```

### Log Levels

| Level | Description | Example |
|-------|-------------|---------|
| **DBG** | Debug information | Internal state changes |
| **INF** | Informational messages | Stream opened, client connected |
| **WRN** | Warning messages | Connection timeouts, packet loss |
| **ERR** | Error messages | Stream failures, configuration errors |

### Common Log Patterns

**Stream Connection:**

```
INF [path Blue_Yeti] opened
INF [path Blue_Yeti] [s->c] runOnReady command started
```

**Client Connection:**

```
INF [conn] [c->s] 192.168.1.100:54321 reader opened
INF [path Blue_Yeti] [c->s] reader 192.168.1.100:54321 opened
```

**Stream Disconnection:**

```
INF [path Blue_Yeti] [s->c] runOnDisconnect command started
INF [path Blue_Yeti] closed
```

**Errors:**

```
ERR [path Blue_Yeti] [s->c] publisher error: EOF
ERR [RTSP] [conn] unable to bind to port 8554: address already in use
```

---

## Stream Manager Logs

### Location

**File:** `/var/log/mediamtx-stream-manager.log`

**Journalctl:** `journalctl -u mediamtx-audio`

### Access

```bash
# View recent logs
sudo tail -f /var/log/mediamtx-stream-manager.log

# Search for errors
sudo grep -i error /var/log/mediamtx-stream-manager.log

# View recovery events
sudo grep -i "restart" /var/log/mediamtx-stream-manager.log

# Last 24 hours of activity
sudo grep "$(date '+%Y-%m-%d')" /var/log/mediamtx-stream-manager.log
```

### Log Format

```
[YYYY-MM-DD HH:MM:SS] [LEVEL] message
```

**Example Entries:**

```
[2025-11-15 10:00:00] [INFO] Starting stream: Blue_Yeti
[2025-11-15 10:00:01] [INFO] Started stream: Blue_Yeti (PID: 12345)
[2025-11-15 10:00:15] [INFO] Stream Blue_Yeti validated: HEALTHY
[2025-11-15 10:30:00] [ERROR] Stream Blue_Yeti failed health check
[2025-11-15 10:30:01] [INFO] Attempting restart (1/5) in 10 seconds
[2025-11-15 10:30:11] [INFO] Restarted Blue_Yeti successfully (PID: 12567)
```

### Common Log Patterns

**Stream Startup:**

```
[INFO] Starting stream manager
[INFO] Found 2 devices in configuration
[INFO] Starting stream: Blue_Yeti
[INFO] Started stream: Blue_Yeti (PID: 12345)
[INFO] All streams started successfully
```

**Stream Failure and Recovery:**

```
[ERROR] Stream Blue_Yeti failed health check
[INFO] Attempting restart (1/5) in 10 seconds
[INFO] Restarting stream: Blue_Yeti
[INFO] Restarted Blue_Yeti successfully (PID: 12567)
[INFO] Recovery successful after 1 attempt(s)
```

**Maximum Retries Reached:**

```
[ERROR] Stream Blue_Yeti failed health check
[INFO] Attempting restart (5/5) in 300 seconds
[ERROR] Maximum restart attempts reached for Blue_Yeti
[ERROR] Stream Blue_Yeti requires manual intervention
```

**Resource Warnings:**

```
[WARN] CPU usage high: 85%
[WARN] Memory available low: 150 MB
[WARN] File descriptors: 850 / 1024 (83%)
```

---

## FFmpeg Device Logs

### Location

**Directory:** `/var/log/lyrebird/`

**Files:** `<device-name>.log` (one per device)

### Access

```bash
# View specific device log
sudo tail -f /var/log/lyrebird/Blue_Yeti.log

# Check all device logs
sudo tail -f /var/log/lyrebird/*.log

# Search for errors across all devices
sudo grep -i error /var/log/lyrebird/*.log

# View last 100 lines per device
sudo ls /var/log/lyrebird/*.log | xargs -I {} sh -c 'echo "=== {} ==="; tail -100 {}'
```

### Log Format

FFmpeg native format with timestamps:

```
[YYYY-MM-DD HH:MM:SS] ffmpeg message
```

**Example Entries:**

```
ffmpeg version 4.4.2 Copyright (c) 2000-2021 the FFmpeg developers
Input #0, alsa, from 'hw:CARD=Blue_Yeti':
  Duration: N/A, start: 1699876800.000000, bitrate: N/A
  Stream #0:0: Audio: pcm_s16le, 48000 Hz, 1 channels, s16, 768 kb/s
Output #0, rtsp, to 'rtsp://localhost:8554/Blue_Yeti':
  Metadata:
    encoder         : Lavf58.76.100
  Stream #0:0: Audio: opus, 48000 Hz, mono, flt, 128 kb/s
Stream mapping:
  Stream #0:0 -> #0:0 (pcm_s16le (native) -> opus (libopus))
frame= 1000 fps= 24 size= 512kB time=00:00:41.66 bitrate= 100.6kbits/s
```

### Common Log Patterns

**Successful Stream Start:**

```
Input #0, alsa, from 'hw:CARD=Blue_Yeti':
  Duration: N/A, start: 1699876800.000000, bitrate: 768 kb/s
  Stream #0:0: Audio: pcm_s16le, 48000 Hz, 1 channels
Output #0, rtsp, to 'rtsp://localhost:8554/Blue_Yeti':
  Stream #0:0: Audio: opus, 48000 Hz, mono, 128 kb/s
```

**Buffer Issues (ALSA overrun):**

```
ALSA buffer xrun
```

**Device Busy:**

```
hw:CARD=Blue_Yeti: Device or resource busy
```

**Encoding Statistics:**

```
frame= 5000 fps= 24 size= 2560kB time=00:03:28.33 bitrate= 100.8kbits/s speed=1x
```

---

## Diagnostic Logs

### Location

**File:** `/var/log/lyrebird-diagnostics.log`

### Access

```bash
# View diagnostic history
sudo cat /var/log/lyrebird-diagnostics.log

# Recent failures only
sudo grep FAIL /var/log/lyrebird-diagnostics.log | tail -20

# Today's diagnostics
sudo grep "$(date '+%Y-%m-%d')" /var/log/lyrebird-diagnostics.log
```

### Log Format

```
[YYYY-MM-DD HH:MM:SS] [STATUS] Check: message
```

**Example Entries:**

```
[2025-11-15 02:00:00] [PASS] Operating System: Ubuntu 22.04.3 LTS
[2025-11-15 02:00:00] [PASS] FFmpeg: version 4.4.2
[2025-11-15 02:00:00] [PASS] USB Audio Devices: 2 found
[2025-11-15 02:00:00] [PASS] MediaMTX Service: active (running)
[2025-11-15 02:00:00] [PASS] CPU Usage: 25% (threshold: 80%)
[2025-11-15 02:00:00] [WARN] Memory Available: 180 MB (threshold: 200 MB)
[2025-11-15 02:00:00] [PASS] Stream Blue_Yeti: healthy
[2025-11-15 02:00:01] [SUMMARY] 18 passed, 1 warning, 0 failures
```

---

## Log Rotation

### Configuration

LyreBird logs should be rotated to prevent disk space exhaustion:

**Create logrotate configuration:**

```bash
# /etc/logrotate.d/lyrebird

/var/log/lyrebird/*.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
    create 0640 root root
}

/var/log/lyrebird-diagnostics.log {
    weekly
    rotate 4
    compress
    missingok
    notifempty
    create 0640 root root
}

/var/log/mediamtx-stream-manager.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
    create 0640 root root
    postrotate
        systemctl reload mediamtx-audio 2>/dev/null || true
    endscript
}
```

**Test logrotate:**

```bash
# Test configuration
sudo logrotate -d /etc/logrotate.d/lyrebird

# Force rotation
sudo logrotate -f /etc/logrotate.d/lyrebird
```

---

## Log Analysis

### Common Error Patterns

**USB Device Issues:**

```bash
# Search for USB errors
sudo grep -i "usb" /var/log/lyrebird/*.log | grep -i error

# Check for device disconnections
sudo dmesg | grep -i usb | tail -20
```

**Stream Failures:**

```bash
# Find all stream failures in last 24 hours
sudo journalctl -u mediamtx --since "24 hours ago" | grep -i error

# Count failures per stream
sudo grep ERROR /var/log/mediamtx-stream-manager.log | \
    awk '{print $5}' | sort | uniq -c
```

**Resource Issues:**

```bash
# Find resource warnings
sudo grep -E "(CPU|memory|descriptor)" /var/log/mediamtx-stream-manager.log

# Check for OOM events
sudo journalctl -k | grep -i "out of memory"
```

---

## Best Practices

### 1. Regular Log Review

```bash
# Daily log check script
#!/bin/bash
echo "=== LyreBird Daily Log Summary ==="
echo "Date: $(date)"
echo ""
echo "Errors in last 24 hours:"
sudo journalctl -u mediamtx --since "24 hours ago" | grep -c ERROR
sudo grep -c ERROR /var/log/mediamtx-stream-manager.log | tail -1440
echo ""
echo "Stream restarts:"
sudo grep -c "Restarted" /var/log/mediamtx-stream-manager.log
```

### 2. Log Retention Policy

- Keep 7 days of detailed logs
- Archive weekly summaries for 4 weeks
- Maintain monthly summaries for 1 year
- Delete logs older than 1 year

### 3. Storage Management

```bash
# Check log directory size
sudo du -sh /var/log/lyrebird/
sudo du -sh /var/log/mediamtx*

# Find large log files
sudo find /var/log/lyrebird -type f -size +100M

# Clean old compressed logs
sudo find /var/log/lyrebird -name "*.gz" -mtime +30 -delete
```

---

## Related Documentation

- [Diagnostics & Monitoring](../advanced/diagnostics-monitoring.md) - Health checks
- [Troubleshooting](../advanced/troubleshooting.md) - Issue resolution
- [Exit Codes](exit-codes.md) - Command return values
- [Command Reference](command-reference.md) - Log-generating commands

---

## Next Steps

<div class="grid" markdown>

<div markdown>
### Diagnostics & Monitoring
System health checks and monitoring

[Diagnostics & Monitoring →](../advanced/diagnostics-monitoring.md)
</div>

<div markdown>
### Troubleshooting
Resolve common issues and problems

[Troubleshooting →](../advanced/troubleshooting.md)
</div>

</div>
