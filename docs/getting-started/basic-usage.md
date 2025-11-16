# Basic Usage

Learn the essential operations for managing LyreBirdAudio streams.

---

## Daily Operations

## Starting Streams

The systemd service starts automatically on boot. To start manually:

```bash
sudo systemctl start mediamtx-audio
```

## Stopping Streams

```bash
sudo systemctl stop mediamtx-audio
```

## Restarting Streams

```bash
sudo systemctl restart mediamtx-audio
```

## Checking Status

```bash
sudo systemctl status mediamtx-audio
```

---

## Stream Manager Commands

The stream manager provides several commands:

```bash
sudo ./mediamtx-stream-manager.sh <command>
```

## Available Commands

| Command | Description |
|---------|-------------|
| `start` | Start all configured streams |
| `stop` | Stop all running streams |
| `restart` | Restart all streams |
| `status` | Show status of all streams |
| `add <name>` | Add a new stream interactively |
| `remove <name>` | Remove a stream |
| `logs <name>` | View logs for a specific stream |

---

## Viewing Stream Status

Get detailed status information:

```bash
sudo ./mediamtx-stream-manager.sh status
```

Example output:
```text
Stream Status Report
Generated: 2025-11-15 14:30:45

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Stream Name       Status     PID      Uptime    Restarts
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
lyrebird-mic-1    RUNNING    1234     2h 15m    0
lyrebird-mic-2    RUNNING    1235     2h 15m    1
front-door        RUNNING    1236     45m       0
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

MediaMTX Server: RUNNING (PID: 1200)
Total Streams: 3 running, 0 stopped
System Load: 0.45, 0.38, 0.32
```

---

## Viewing Logs

### Stream-Specific Logs

```bash
sudo ./mediamtx-stream-manager.sh logs lyrebird-mic-1
```

## Service Logs

```bash
# Last 50 lines
journalctl -u mediamtx-audio -n 50

# Follow live logs
journalctl -u mediamtx-audio -f

# Logs since boot
journalctl -u mediamtx-audio -b
```

## MediaMTX Server Logs

```bash
tail -f /opt/mediamtx/mediamtx.log
```

---

## Accessing Streams

## RTSP URL Format

```text
rtsp://[server-ip]:8554/[stream-name]
```

Examples:
```text
rtsp://localhost:8554/lyrebird-mic-1
rtsp://192.168.1.100:8554/front-door
rtsp://myserver.local:8554/bird-monitor
```

## Using VLC Player

=== "Command Line"

    ```bash
    vlc rtsp://localhost:8554/lyrebird-mic-1
    ```

=== "VLC GUI"

    1. Open VLC
    2. Media -> Open Network Stream
    3. Enter: `rtsp://localhost:8554/lyrebird-mic-1`
    4. Click Play

## Using FFmpeg

### Listen to Stream

```bash
ffplay -nodisp rtsp://localhost:8554/lyrebird-mic-1
```

#### Record Stream

```bash
# Record for 60 seconds
ffmpeg -i rtsp://localhost:8554/lyrebird-mic-1 \
  -t 60 \
  -acodec copy \
  recording.aac
```

## Continuous Recording

```bash
# Record with automatic file rotation every hour
ffmpeg -i rtsp://localhost:8554/lyrebird-mic-1 \
  -acodec copy \
  -f segment \
  -segment_time 3600 \
  -strftime 1 \
  "recordings/%Y-%m-%d_%H-%M-%S.aac"
```

---

## Adding New Streams

### Using the Orchestrator

```bash
sudo ./lyrebird-orchestrator.sh
```

Select **Option 4: Add New Stream**

## Using Stream Manager Directly

```bash
sudo ./mediamtx-stream-manager.sh add my-new-stream
```

Follow the interactive prompts to:

1. Select a mapped USB device
2. Configure audio settings
3. Set RTSP path name

---

## Removing Streams

### Using the Orchestrator

```bash
sudo ./lyrebird-orchestrator.sh
```

Select **Option 5: Remove Stream**

## Using Stream Manager

```bash
sudo ./mediamtx-stream-manager.sh remove lyrebird-mic-1
```

!!! warning "Permanent Deletion"
    Removing a stream deletes its configuration file. This action cannot be undone.

---

## Managing USB Devices

## List Mapped Devices

```bash
ls -l /dev/lyrebird-*
```

Example output:
```text
lrwxrwxrwx 1 root root 12 Nov 15 10:00 /dev/lyrebird-mic-1 -> snd/pcmC1D0c
lrwxrwxrwx 1 root root 12 Nov 15 10:00 /dev/lyrebird-mic-2 -> snd/pcmC2D0c
```

## Add New USB Device

```bash
sudo ./lyrebird-orchestrator.sh
```

Select **Option 2: Map USB Audio Devices**

## Remove USB Device Mapping

```bash
# Find the udev rule
ls /etc/udev/rules.d/99-lyrebird-*.rules

# Remove specific rule
sudo rm /etc/udev/rules.d/99-lyrebird-mic-1.rules

# Reload udev
sudo udevadm control --reload-rules
sudo udevadm trigger
```

---

## System Diagnostics

## Quick Health Check

```bash
sudo ./lyrebird-diagnostics.sh quick
```

Output:
```text
LyreBirdAudio System Diagnostics
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[PASS] MediaMTX Server: Running
[PASS] Stream Manager: Active
[PASS] USB Devices: 2 mapped
[PASS] Active Streams: 2/2
[PASS] Disk Space: 45% used
[PASS] System Load: Normal

Status: HEALTHY
```

## Comprehensive Diagnostics

```bash
sudo ./lyrebird-diagnostics.sh full
```

Generates detailed report including:

- System information
- USB device details
- Stream health
- Audio capability tests
- Log analysis
- Performance metrics

---

## Updating LyreBirdAudio

### Check for Updates

```bash
cd /opt/LyreBirdAudio
git fetch origin
git status
```

## Apply Updates

```bash
sudo ./lyrebird-updater.sh update
```

The version manager will:

1. Create backup of current version
2. Pull latest changes
3. Preserve configurations
4. Restart services if needed

### Rollback if Needed

```bash
sudo ./lyrebird-updater.sh rollback
```

---

## Best Practices

## Regular Monitoring

Setup a cron job for daily health checks:

```bash
sudo crontab -e
```

Add:
```bash
0 8 * * * /opt/LyreBirdAudio/lyrebird-diagnostics.sh quick | mail -s "LyreBird Status" admin@example.com
```

## Log Rotation

Logs are automatically rotated. Verify configuration:

```bash
cat /etc/logrotate.d/mediamtx
```

## Backup Configurations

Regularly backup stream configurations:

```bash
sudo tar -czf lyrebird-backup-$(date +%Y%m%d).tar.gz \
  /opt/mediamtx/streams \
  /opt/mediamtx/mediamtx.yml \
  /etc/udev/rules.d/99-lyrebird-*.rules
```

---

## Common Tasks

### Restart a Specific Stream

Streams automatically restart on failure, but you can manually restart:

```bash
# Stop the stream
sudo systemctl stop mediamtx-audio
sudo pkill -f "lyrebird-mic-1"

# Start the service again
sudo systemctl start mediamtx-audio
```

## Change Stream Settings

Edit the stream configuration:

```bash
sudo nano /opt/mediamtx/streams/lyrebird-mic-1.env
```

Restart for changes to take effect:

```bash
sudo systemctl restart mediamtx-audio
```

## Test Audio Input

```bash
# Record 5 seconds directly from device
arecord -D /dev/lyrebird-mic-1 -d 5 -f cd test.wav

# Play back the recording
aplay test.wav
```

---

## Monitoring Stream Health

## Check Stream Uptime

```bash
ps -eo pid,etime,cmd | grep ffmpeg
```

## Monitor Resource Usage

```bash
# Real-time process monitoring
htop -p $(pgrep -d',' ffmpeg)

# Check CPU and memory
ps aux | grep ffmpeg
```

## Network Traffic

```bash
# Monitor RTSP connections
sudo netstat -an | grep 8554

# Active client connections
sudo lsof -i :8554
```

---

## Next Steps

<div class="grid" markdown>

<div markdown>
###  Advanced Configuration
Learn about codec selection, sample rates, and bitrate tuning

[Configuration Guide ->](../user-guide/configuration.md)
</div>

<div markdown>
###  Monitoring
Setup comprehensive monitoring and alerts

[Diagnostics & Monitoring ->](../advanced/diagnostics-monitoring.md)
</div>

<div markdown>
###  Troubleshooting
Solutions for common issues

[Troubleshooting ->](../advanced/troubleshooting.md)
</div>

<div markdown>
###  Component Reference
Deep dive into each component

[Components ->](../components/index.md)
</div>

</div>
