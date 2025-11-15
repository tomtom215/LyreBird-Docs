# Stream Management

Comprehensive guide to managing RTSP stream lifecycle, monitoring, and automatic recovery.

---

## Overview

LyreBirdAudio's stream management is handled by the `mediamtx-stream-manager.sh` script, which provides complete lifecycle control for RTSP audio streams. The stream manager handles FFmpeg process orchestration, health monitoring, automatic recovery, and resource tracking.

This guide covers:

- Stream lifecycle management (start, stop, restart)
- Stream monitoring and health checks
- Self-healing and automatic recovery
- Resource usage tracking
- Production deployment with systemd
- Troubleshooting stream issues

---

## Stream Manager Script

### Purpose

The `mediamtx-stream-manager.sh` script manages all aspects of RTSP audio streaming:

- Launches and controls FFmpeg processes for each audio device
- Monitors stream health via MediaMTX API
- Automatically restarts failed streams with exponential backoff
- Tracks system resources (CPU, memory, file descriptors)
- Provides production-ready systemd integration

### Key Features

- **Complete Lifecycle Control:** Start, stop, restart, and monitor streams
- **Self-Healing:** Automatic recovery from failures with intelligent backoff
- **Health Monitoring:** Real-time stream status via MediaMTX API
- **Resource Tracking:** Monitor CPU, memory, and file descriptor usage
- **Production Ready:** Systemd service integration for automatic startup
- **Multiplex Support:** Combine multiple audio sources into single streams

---

## Basic Commands

### Start Streams

Start all configured audio streams:

```bash
sudo ./mediamtx-stream-manager.sh start
```

This will:

1. Read device configuration from `/etc/mediamtx/audio-devices.conf`
2. Launch FFmpeg process for each configured device
3. Wait for stream stabilization (default: 10 seconds)
4. Validate streams are active via MediaMTX API
5. Report status of all streams

**Example output:**

```
Starting streams for all configured devices...
Started stream: Blue_Yeti (PID: 12345)
Started stream: USB_Microphone (PID: 12346)
Waiting 10 seconds for streams to stabilize...
All streams started successfully.
```

### Stop Streams

Stop all running streams:

```bash
sudo ./mediamtx-stream-manager.sh stop
```

This will:

1. Identify all running FFmpeg stream processes
2. Send graceful shutdown signal (SIGTERM)
3. Wait for clean shutdown
4. Force kill if processes don't exit (SIGKILL)
5. Clean up PID files and state

**Example output:**

```
Stopping all streams...
Stopped stream: Blue_Yeti (PID: 12345)
Stopped stream: USB_Microphone (PID: 12346)
All streams stopped.
```

### Restart Streams

Restart all streams (stop then start):

```bash
sudo ./mediamtx-stream-manager.sh restart
```

Useful after:

- Configuration changes
- Device reconnection
- System updates
- Troubleshooting stream issues

### Check Stream Status

View current status of all streams:

```bash
sudo ./mediamtx-stream-manager.sh status
```

**Example output:**

```
Stream Status:
==============
Blue_Yeti: RUNNING (PID: 12345, Uptime: 2h 15m)
  URL: rtsp://localhost:8554/Blue_Yeti
  Codec: opus, Sample Rate: 48000 Hz, Bitrate: 128k

USB_Microphone: RUNNING (PID: 12346, Uptime: 2h 15m)
  URL: rtsp://localhost:8554/USB_Microphone
  Codec: aac, Sample Rate: 44100 Hz, Bitrate: 192k

Total Streams: 2 running, 0 failed
```

### List Available Streams

List all streams available via MediaMTX:

```bash
sudo ./mediamtx-stream-manager.sh list
```

**Example output:**

```
Available RTSP Streams:
=======================
Blue_Yeti - rtsp://localhost:8554/Blue_Yeti
USB_Microphone - rtsp://localhost:8554/USB_Microphone
Office_Multiplex - rtsp://localhost:8554/Office_Multiplex

Total: 3 streams
```

### Monitor Streams

Continuous monitoring mode with real-time updates:

```bash
sudo ./mediamtx-stream-manager.sh monitor
```

Displays:

- Stream health status
- CPU usage per stream
- Memory consumption
- File descriptor count
- Recent errors or warnings
- Automatic recovery actions

**Example output:**

```
Stream Monitor - Updating every 5 seconds (Ctrl+C to exit)
===========================================================
[10:30:15] Blue_Yeti: HEALTHY (CPU: 3.2%, MEM: 45 MB)
[10:30:15] USB_Microphone: HEALTHY (CPU: 2.8%, MEM: 42 MB)
[10:30:15] System: FD: 245/1024, Total CPU: 6.0%

[10:30:20] Blue_Yeti: HEALTHY (CPU: 3.1%, MEM: 45 MB)
[10:30:20] USB_Microphone: RECOVERING (Restart attempt 1/5)
[10:30:20] System: FD: 248/1024, Total CPU: 5.8%
```

Press `Ctrl+C` to exit monitoring mode.

---

## Self-Healing Streams

### Automatic Recovery System

LyreBirdAudio implements intelligent self-healing for stream failures:

**Recovery Features:**

- Automatic detection of failed streams
- Exponential backoff restart strategy
- Maximum retry attempts
- Resource cleanup between restarts
- Health validation after recovery

### Exponential Backoff Strategy

When a stream fails, the system uses exponential backoff to avoid rapid restart loops:

**Backoff Schedule:**

| Attempt | Delay | Total Wait Time |
|---------|-------|-----------------|
| 1 | 10 seconds | 10 seconds |
| 2 | 20 seconds | 30 seconds |
| 3 | 40 seconds | 70 seconds |
| 4 | 80 seconds | 150 seconds |
| 5 | 160 seconds | 310 seconds |
| 6+ | 300 seconds (max) | Capped at 5 minutes |

**Configuration:**

```bash
# Override default max delay (300 seconds)
MAX_RESTART_DELAY=600 ./mediamtx-stream-manager.sh start
```

### Recovery Process

When a stream failure is detected:

1. **Failure Detection**
   - MediaMTX API returns stream inactive
   - FFmpeg process terminated unexpectedly
   - Health check timeout

2. **Cleanup**
   - Kill existing FFmpeg process (if still running)
   - Remove stale PID files
   - Clear stream state

3. **Backoff Wait**
   - Calculate exponential delay
   - Wait for backoff period
   - Log restart attempt

4. **Restart**
   - Launch new FFmpeg process
   - Wait for stabilization
   - Validate stream health

5. **Validation**
   - Check MediaMTX API for stream presence
   - Verify FFmpeg process is running
   - Confirm audio data flow

**Example recovery log:**

```
[ERROR] Stream Blue_Yeti failed - device disconnected
[INFO] Attempting restart (1/5) in 10 seconds...
[INFO] Restarted Blue_Yeti successfully
[WARN] Stream Blue_Yeti failed again
[INFO] Attempting restart (2/5) in 20 seconds...
[INFO] Restarted Blue_Yeti successfully
```

### Permanent Failure Handling

After maximum retry attempts, the stream enters failed state:

- Stops automatic restart attempts
- Logs permanent failure
- Requires manual intervention

**Manual recovery:**

```bash
# Check device connection
cat /proc/asound/cards

# Restart specific stream after fixing issue
sudo ./mediamtx-stream-manager.sh restart
```

---

## Health Monitoring

### Stream Health Checks

The stream manager performs continuous health monitoring:

**Monitoring Methods:**

1. **MediaMTX API Checks**
   - Queries `/v3/paths/get/<stream-name>`
   - Verifies stream is active
   - Checks for client connections

2. **Process Monitoring**
   - Verifies FFmpeg process is running
   - Checks process hasn't become zombie
   - Monitors process resource usage

3. **Resource Tracking**
   - CPU usage per stream
   - Memory consumption
   - File descriptor count

### Health Check Endpoints

Query MediaMTX API directly for stream health:

```bash
# List all streams
curl http://localhost:9997/v3/paths/list

# Get specific stream details
curl http://localhost:9997/v3/paths/get/Blue_Yeti
```

**Example response:**

```json
{
  "name": "Blue_Yeti",
  "ready": true,
  "tracks": ["audio"],
  "bytesReceived": 1048576,
  "readers": 2
}
```

### Health Status Indicators

| Status | Description | Action Required |
|--------|-------------|-----------------|
| HEALTHY | Stream active and stable | None |
| STARTING | Stream initializing | Wait for stabilization |
| RECOVERING | Automatic restart in progress | Monitor for success |
| DEGRADED | Stream running but issues detected | Investigate warnings |
| FAILED | Stream permanently failed | Manual intervention needed |

---

## Resource Tracking

### CPU Monitoring

Monitor CPU usage per stream and total system load:

```bash
# View CPU usage in monitoring mode
sudo ./mediamtx-stream-manager.sh monitor
```

**CPU Thresholds:**

- **Normal:** < 5% per stream
- **Warning:** 5-20% per stream
- **Critical:** > 20% per stream

**High CPU causes:**

- Codec encoding complexity (PCM < Opus < AAC < MP3)
- High sample rates (96kHz vs 48kHz)
- Multiple concurrent streams
- System resource contention

**Solutions:**

- Lower bitrate or sample rate
- Switch to more efficient codec (Opus)
- Reduce number of concurrent streams
- Upgrade hardware

### Memory Monitoring

Each FFmpeg stream typically uses 40-80 MB of RAM:

**Memory per stream:**

- **Low Quality (16kHz):** ~30 MB
- **Normal Quality (48kHz):** ~50 MB
- **High Quality (96kHz):** ~80 MB

**Memory calculation:**

```
Total Memory = (Number of Streams × 50 MB) + 200 MB base system
```

**Example:**

```
10 streams × 50 MB = 500 MB
+ 200 MB system overhead
= 700 MB total
```

### File Descriptor Tracking

Monitor file descriptor usage to prevent exhaustion:

```bash
# Check current FD usage
sudo ./mediamtx-stream-manager.sh status
```

**File Descriptor Usage:**

- Each stream uses ~10-15 file descriptors
- System limits typically: 1024 (soft) / 4096 (hard)

**Warning thresholds:**

| FD Count | Status | Action |
|----------|--------|--------|
| < 500 | Normal | None |
| 500-800 | Warning | Monitor closely |
| > 800 | Critical | Reduce streams or increase limit |

**Increase FD limit:**

```bash
# Temporary increase
ulimit -n 4096

# Permanent increase (add to /etc/security/limits.conf)
* soft nofile 4096
* hard nofile 8192
```

---

## Production Deployment

### Systemd Service Installation

For production environments, install the stream manager as a systemd service:

```bash
sudo ./mediamtx-stream-manager.sh install-service
```

This creates a systemd unit file that:

- Starts streams automatically on boot
- Manages stream lifecycle
- Provides logging via journalctl
- Enables dependency management

### Service Management

After installation, manage the service using systemd commands:

**Enable automatic startup:**

```bash
sudo systemctl enable mediamtx-audio
```

**Start the service:**

```bash
sudo systemctl start mediamtx-audio
```

**Stop the service:**

```bash
sudo systemctl stop mediamtx-audio
```

**Restart the service:**

```bash
sudo systemctl restart mediamtx-audio
```

**Check service status:**

```bash
sudo systemctl status mediamtx-audio
```

**Example status output:**

```
● mediamtx-audio.service - LyreBirdAudio Stream Manager
   Loaded: loaded (/etc/systemd/system/mediamtx-audio.service)
   Active: active (running) since Mon 2025-11-15 10:00:00 UTC; 2h 15m ago
 Main PID: 12340 (mediamtx-stream)
    Tasks: 12 (limit: 4096)
   Memory: 450.0M
   CGroup: /system.slice/mediamtx-audio.service
           ├─12340 /bin/bash ./mediamtx-stream-manager.sh monitor
           ├─12345 ffmpeg -f alsa -i hw:CARD=Blue_Yeti ...
           └─12346 ffmpeg -f alsa -i hw:CARD=USB_Microphone ...
```

### Viewing Service Logs

Monitor stream activity via journalctl:

```bash
# View recent logs
sudo journalctl -u mediamtx-audio -n 50

# Follow logs in real-time
sudo journalctl -u mediamtx-audio -f

# View logs since boot
sudo journalctl -u mediamtx-audio -b

# Filter for errors only
sudo journalctl -u mediamtx-audio -p err
```

**Example log output:**

```
Nov 15 10:00:00 server mediamtx-audio[12340]: Starting streams...
Nov 15 10:00:01 server mediamtx-audio[12340]: Started Blue_Yeti (PID: 12345)
Nov 15 10:00:02 server mediamtx-audio[12340]: Started USB_Microphone (PID: 12346)
Nov 15 10:00:12 server mediamtx-audio[12340]: All streams validated
Nov 15 12:15:30 server mediamtx-audio[12340]: Stream Blue_Yeti health check: OK
```

---

## Environment Variables

### Timing Controls

Override default timing parameters:

```bash
# Stream startup delay (default: 10 seconds)
STREAM_STARTUP_DELAY=20 ./mediamtx-stream-manager.sh start

# USB device stabilization delay (default: 5 seconds)
USB_STABILIZATION_DELAY=10 ./mediamtx-stream-manager.sh start

# Maximum restart backoff delay (default: 300 seconds)
MAX_RESTART_DELAY=600 ./mediamtx-stream-manager.sh start
```

**Timing Parameters:**

| Variable | Default | Description |
|----------|---------|-------------|
| `STREAM_STARTUP_DELAY` | 10 sec | Wait before stream validation |
| `USB_STABILIZATION_DELAY` | 5 sec | USB device initialization time |
| `MAX_RESTART_DELAY` | 300 sec | Maximum exponential backoff |

### Resource Thresholds

Configure resource warning levels:

```bash
# File descriptor warning threshold (default: 500)
MAX_FD_WARNING=800 ./mediamtx-stream-manager.sh monitor

# CPU usage warning percentage (default: 20%)
MAX_CPU_WARNING=30 ./mediamtx-stream-manager.sh monitor
```

**Resource Parameters:**

| Variable | Default | Description |
|----------|---------|-------------|
| `MAX_FD_WARNING` | 500 | File descriptor warning level |
| `MAX_CPU_WARNING` | 20% | CPU usage warning threshold |

### Configuration Paths

Override default file locations:

```bash
# Audio device configuration
AUDIO_CONFIG=/path/to/audio-devices.conf ./mediamtx-stream-manager.sh start

# MediaMTX configuration
MEDIAMTX_CONFIG=/path/to/mediamtx.yml ./mediamtx-stream-manager.sh start
```

---

## Troubleshooting

### Streams Won't Start

**Symptom:** Streams fail to start or exit immediately

**Solutions:**

1. **Check device availability:**
   ```bash
   cat /proc/asound/cards
   ```

2. **Verify configuration:**
   ```bash
   sudo ./lyrebird-mic-check.sh -V
   ```

3. **Test FFmpeg manually:**
   ```bash
   ffmpeg -f alsa -i hw:CARD=Blue_Yeti -f rtsp rtsp://localhost:8554/test
   ```

4. **Check MediaMTX status:**
   ```bash
   sudo systemctl status mediamtx
   ```

5. **Review logs:**
   ```bash
   sudo journalctl -u mediamtx-audio -n 50
   ```

### Streams Start Then Fail

**Symptom:** Streams start successfully but fail after running

**Common Causes:**

- USB device disconnection
- Audio buffer underruns
- MediaMTX server issues
- Resource exhaustion

**Solutions:**

1. **Check USB connection:**
   ```bash
   dmesg | grep -i usb | tail -20
   ```

2. **Increase thread queue size:**
   ```bash
   # In /etc/mediamtx/audio-devices.conf
   DEVICE_<NAME>_THREAD_QUEUE=2048
   ```

3. **Monitor resource usage:**
   ```bash
   sudo ./mediamtx-stream-manager.sh monitor
   ```

4. **Check MediaMTX logs:**
   ```bash
   sudo journalctl -u mediamtx -f
   ```

### High CPU Usage

**Symptom:** FFmpeg processes consuming excessive CPU

**Solutions:**

1. **Lower sample rate:**
   ```bash
   DEVICE_<NAME>_SAMPLE_RATE=16000
   ```

2. **Switch to Opus codec:**
   ```bash
   DEVICE_<NAME>_CODEC=opus
   ```

3. **Reduce bitrate:**
   ```bash
   DEVICE_<NAME>_BITRATE=64k
   ```

4. **Limit concurrent streams:**
   - Reduce number of active devices
   - Implement stream rotation

### Memory Exhaustion

**Symptom:** System runs out of memory with many streams

**Solutions:**

1. **Calculate required memory:**
   ```
   Required = (Num_Streams × 50 MB) + 200 MB
   ```

2. **Reduce stream count or add RAM**

3. **Use lower quality settings:**
   ```bash
   sudo ./lyrebird-mic-check.sh -g --quality=low
   ```

4. **Monitor with system tools:**
   ```bash
   free -h
   top -p $(pgrep ffmpeg | tr '\n' ',')
   ```

### File Descriptor Exhaustion

**Symptom:** "Too many open files" errors

**Solutions:**

1. **Check current limits:**
   ```bash
   ulimit -n
   ```

2. **Increase limits temporarily:**
   ```bash
   ulimit -n 4096
   sudo ./mediamtx-stream-manager.sh restart
   ```

3. **Increase limits permanently:**
   ```bash
   # Edit /etc/security/limits.conf
   * soft nofile 4096
   * hard nofile 8192
   ```

4. **Verify after restart:**
   ```bash
   sudo su - $USER
   ulimit -n
   ```

### Automatic Recovery Not Working

**Symptom:** Failed streams don't restart automatically

**Solutions:**

1. **Check monitoring mode is active:**
   ```bash
   sudo systemctl status mediamtx-audio
   ```

2. **Verify auto-recovery isn't disabled:**
   ```bash
   # Recovery should happen in monitor mode
   sudo ./mediamtx-stream-manager.sh monitor
   ```

3. **Check max retry count:**
   - Default is 5 attempts
   - After 5 failures, manual restart required

4. **Review failure logs:**
   ```bash
   sudo journalctl -u mediamtx-audio -p err
   ```

---

## Best Practices

### Production Deployments

1. **Use systemd service** - Enable automatic startup and management
2. **Monitor continuously** - Run in monitor mode or check status regularly
3. **Set appropriate timeouts** - Adjust startup delays for your hardware
4. **Plan for capacity** - Calculate CPU and memory requirements before deployment
5. **Implement alerting** - Monitor logs and integrate with monitoring systems

### Performance Optimization

1. **Start with normal quality** - Use default settings initially
2. **Monitor resource usage** - Watch CPU, memory, and FD counts
3. **Scale gradually** - Add streams incrementally to identify limits
4. **Use efficient codecs** - Prefer Opus for best quality/performance ratio
5. **Match hardware capabilities** - Don't exceed device specifications

### Reliability

1. **Test recovery** - Simulate failures to verify auto-recovery works
2. **Use powered USB hubs** - Prevent power-related device issues
3. **Enable auto-start** - Configure systemd for automatic startup
4. **Backup configurations** - Version control configuration files
5. **Document deployments** - Keep records of device mappings and settings

### Monitoring and Maintenance

1. **Regular health checks** - Monitor stream status daily
2. **Review logs weekly** - Check for patterns of failures or warnings
3. **Update configurations** - Adjust settings based on observed performance
4. **Test after changes** - Validate configuration changes before production deployment
5. **Plan maintenance windows** - Schedule system updates during low-usage periods

---

## Related Documentation

- [Configuration](configuration.md) - Audio encoding and quality settings
- [USB Device Management](usb-device-management.md) - Device detection and mapping
- [MediaMTX Integration](mediamtx-integration.md) - RTSP server configuration
- [Multiplex Streaming](multiplex-streaming.md) - Multi-microphone stream combining
- [Performance Tuning](../advanced/performance.md) - Optimization strategies

---

## Next Steps

<div class="grid" markdown>

<div markdown>
### MediaMTX Integration
RTSP server configuration and setup

[MediaMTX Integration →](mediamtx-integration.md)
</div>

<div markdown>
### Multiplex Streaming
Multi-microphone stream combining

[Multiplex Streaming →](multiplex-streaming.md)
</div>

</div>
