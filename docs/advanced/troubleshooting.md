# Troubleshooting

Comprehensive troubleshooting guide for diagnosing and resolving common LyreBirdAudio issues.

---

## Overview

This guide provides systematic approaches to diagnosing and resolving issues with LyreBirdAudio. Each problem includes symptoms, root causes, diagnostic steps, and detailed solutions.

This guide covers:

- Device detection and recognition issues
- Stream startup and stability problems
- USB persistence failures
- MediaMTX service issues
- Resource exhaustion problems
- Network connectivity issues
- Audio quality problems

---

## Quick Diagnostics

Before diving into specific issues, run the diagnostic tool:

```bash
# Run comprehensive health check
sudo ./lyrebird-diagnostics.sh

# Run with verbose output
sudo ./lyrebird-diagnostics.sh --verbose

# Export detailed report
sudo ./lyrebird-diagnostics.sh --export=/tmp/diagnostic-report.txt
```

The diagnostics tool performs 24+ automated checks and reports:

- Device detection status
- Service health
- Configuration validation
- Resource availability
- Network connectivity
- Stream status

---

## Device Detection Issues

### USB Device Not Detected

**Symptoms:**

- Device doesn't appear in `/proc/asound/cards`
- `arecord -l` doesn't list the device
- udev rules don't create symlinks

**Diagnostic Steps:**

```bash
# Check if device is connected via USB
lsusb

# Check if kernel recognizes audio device
cat /proc/asound/cards

# Check dmesg for USB errors
dmesg | grep -i usb | tail -20

# List all audio devices
arecord -l
```

**Common Causes and Solutions:**

1. **Device Not Properly Connected**

   **Solution:**
   ```bash
   # Unplug and replug the device
   # Wait 5 seconds for detection

   # Verify connection
   lsusb | grep -i audio
   ```

2. **USB Power Issues**

   **Symptoms:**
   - Device appears and disappears
   - `dmesg` shows "device not accepting address"
   - Undervoltage warnings (Raspberry Pi)

   **Solution:**
   ```bash
   # Use powered USB hub
   # Check power supply adequacy

   # For Raspberry Pi, check for undervoltage
   vcgencmd get_throttled
   # 0x0 = no issues
   # Other values indicate power problems
   ```

3. **Driver Not Loaded**

   **Solution:**
   ```bash
   # Load USB audio driver
   sudo modprobe snd-usb-audio

   # Make permanent
   echo "snd-usb-audio" | sudo tee -a /etc/modules

   # Verify module loaded
   lsmod | grep snd_usb_audio
   ```

4. **Device Requires Firmware**

   **Solution:**
   ```bash
   # Install linux-firmware package
   sudo apt-get update
   sudo apt-get install linux-firmware

   # Reboot
   sudo reboot
   ```

5. **USB Controller Issues**

   **Solution:**
   ```bash
   # Try different USB port
   # Check USB controller assignment
   lsusb -t

   # Reset USB controller (last resort)
   echo -n "0000:00:14.0" | sudo tee /sys/bus/pci/drivers/xhci_hcd/unbind
   sleep 5
   echo -n "0000:00:14.0" | sudo tee /sys/bus/pci/drivers/xhci_hcd/bind
   ```

### Device Detected But Not Persistent

**Symptoms:**

- Device appears in `/proc/asound/cards` but card number changes
- Streams fail after device reconnection
- udev symlinks not created

**Diagnostic Steps:**

```bash
# Check if udev rules exist
ls -l /etc/udev/rules.d/99-usb-soundcards.rules

# Check if symlinks exist
ls -l /dev/snd/by-id/

# Test udev rule application
sudo udevadm test /sys/class/sound/card0
```

**Solutions:**

1. **Generate udev Rules**
   ```bash
   # Run USB mapper to create rules
   sudo ./usb-audio-mapper.sh

   # Reload udev rules
   sudo udevadm control --reload-rules
   sudo udevadm trigger

   # Verify rules applied
   ls -l /dev/snd/by-id/
   ```

2. **Fix Incorrect udev Rules**
   ```bash
   # Check rule syntax
   sudo cat /etc/udev/rules.d/99-usb-soundcards.rules

   # Common issues:
   # - Incorrect vendor/product IDs
   # - Missing quotes
   # - Incorrect SUBSYSTEM

   # Regenerate rules
   sudo ./usb-audio-mapper.sh
   ```

3. **Multiple Devices with Same ID**

   **Problem:** Two identical devices (same vendor/product ID)

   **Solution:**
   ```bash
   # Use USB topology-based naming
   # Edit /etc/udev/rules.d/99-usb-soundcards.rules

   # Example: Use KERNELS (USB port) instead of just vendor/product
   SUBSYSTEM=="sound", KERNELS=="1-1.2", ATTR{id}="Mic_Port_1", SYMLINK+="sound/by-id/Mic_Port_1"
   SUBSYSTEM=="sound", KERNELS=="1-1.3", ATTR{id}="Mic_Port_2", SYMLINK+="sound/by-id/Mic_Port_2"

   # Reload rules
   sudo udevadm control --reload-rules
   sudo udevadm trigger
   ```

### Device Access Permission Issues

**Symptoms:**

- "Permission denied" errors when accessing device
- Streams fail to start with permission errors
- FFmpeg cannot open audio device

**Solutions:**

1. **Add User to Audio Group**
   ```bash
   # Add current user to audio group
   sudo usermod -a -G audio $USER

   # Log out and back in for changes to take effect
   # Or use:
   sudo su - $USER

   # Verify group membership
   groups
   ```

2. **Fix Device Permissions**
   ```bash
   # Check current permissions
   ls -l /dev/snd/

   # Should show group=audio, permissions 660
   # If not, fix with udev rule:

   # Add to /etc/udev/rules.d/99-usb-soundcards.rules
   SUBSYSTEM=="sound", MODE="0660", GROUP="audio"

   # Reload rules
   sudo udevadm control --reload-rules
   sudo udevadm trigger
   ```

---

## Stream Startup Issues

### Streams Won't Start

**Symptoms:**

- `mediamtx-stream-manager.sh start` fails
- FFmpeg processes exit immediately
- No error messages or unclear errors

**Diagnostic Steps:**

```bash
# Check stream manager status
sudo ./mediamtx-stream-manager.sh status

# Try starting with verbose FFmpeg output
sudo LOGLEVEL=debug ./mediamtx-stream-manager.sh start

# Check if MediaMTX is running
sudo systemctl status mediamtx

# Test FFmpeg manually
ffmpeg -f alsa -i hw:CARD=Blue_Yeti -f null -
```

**Common Causes and Solutions:**

1. **MediaMTX Not Running**

   **Solution:**
   ```bash
   # Start MediaMTX service
   sudo systemctl start mediamtx

   # Enable automatic startup
   sudo systemctl enable mediamtx

   # Verify running
   sudo systemctl status mediamtx
   ```

2. **Device Configuration Missing**

   **Solution:**
   ```bash
   # Check if configuration exists
   cat /etc/mediamtx/audio-devices.conf

   # Generate configuration
   sudo ./lyrebird-mic-check.sh -g

   # Verify configuration
   sudo ./lyrebird-mic-check.sh -V
   ```

3. **Incorrect Device Name in Configuration**

   **Solution:**
   ```bash
   # List actual device names
   cat /proc/asound/cards

   # Update configuration to match
   # Edit /etc/mediamtx/audio-devices.conf
   # Change DEVICE_<NAME> to match actual card name

   # Example:
   # If device is "Blue Yeti" in /proc/asound/cards
   # Use: DEVICE_Blue_Yeti_...
   ```

4. **Device Already in Use**

   **Solution:**
   ```bash
   # Check if another process is using device
   sudo lsof /dev/snd/*

   # Kill conflicting processes
   sudo pkill -f "process-name"

   # Or stop PulseAudio if interfering
   systemctl --user stop pulseaudio.socket
   systemctl --user stop pulseaudio.service
   ```

5. **Codec Not Supported**

   **Solution:**
   ```bash
   # Check FFmpeg codec support
   ffmpeg -codecs | grep -i opus
   ffmpeg -codecs | grep -i aac

   # Install additional codecs if needed
   sudo apt-get install ffmpeg libavcodec-extra

   # Or use different codec
   DEVICE_NAME_CODEC=mp3  # Instead of opus
   ```

### Streams Start Then Immediately Fail

**Symptoms:**

- Streams start successfully but exit within seconds
- FFmpeg processes appear then disappear
- MediaMTX shows stream connects then disconnects

**Diagnostic Steps:**

```bash
# Check FFmpeg logs
tail -f /var/log/lyrebird/*.log

# Check MediaMTX logs
sudo journalctl -u mediamtx -f

# Watch stream status in real-time
sudo ./mediamtx-stream-manager.sh monitor
```

**Solutions:**

1. **Audio Buffer Underruns**

   **Symptoms:**
   - FFmpeg logs show "ALSA buffer xrun"
   - Intermittent audio capture

   **Solution:**
   ```bash
   # Increase thread queue size
   # Edit /etc/mediamtx/audio-devices.conf
   DEVICE_NAME_THREAD_QUEUE=2048  # Increase from 1024

   # Restart streams
   sudo ./mediamtx-stream-manager.sh restart
   ```

2. **USB Device Disconnections**

   **Symptoms:**
   - dmesg shows USB resets or disconnects
   - Device appears/disappears

   **Solution:**
   ```bash
   # Check USB stability
   dmesg | grep -i usb | tail -20

   # Use powered USB hub
   # Try different USB port
   # Check USB cable quality

   # Disable USB autosuspend
   echo -1 | sudo tee /sys/module/usbcore/parameters/autosuspend
   ```

3. **MediaMTX Connection Refused**

   **Symptoms:**
   - FFmpeg logs show "Connection refused"
   - Can't connect to localhost:8554

   **Solution:**
   ```bash
   # Check if MediaMTX is listening
   sudo netstat -tlnp | grep 8554

   # Check MediaMTX configuration
   grep rtspAddress /etc/mediamtx/mediamtx.yml

   # Should show: rtspAddress: :8554
   # If different, update configuration

   # Restart MediaMTX
   sudo systemctl restart mediamtx
   ```

4. **Sample Rate Not Supported**

   **Symptoms:**
   - FFmpeg errors about sample rate
   - Device rejects sample rate

   **Solution:**
   ```bash
   # Test device capabilities
   sudo ./lyrebird-mic-check.sh

   # Use supported sample rate
   # Common rates: 16000, 44100, 48000

   # Update configuration
   DEVICE_NAME_SAMPLE_RATE=48000
   ```

---

## Stream Stability Issues

### Intermittent Stream Dropouts

**Symptoms:**

- Streams run but periodically fail
- Audio stuttering or gaps
- Clients experience buffering

**Diagnostic Steps:**

```bash
# Monitor stream health
sudo ./mediamtx-stream-manager.sh monitor

# Check system resources
top
free -h

# Check network stability
ping -c 100 localhost

# Monitor USB events
sudo udevadm monitor --environment --udev
```

**Solutions:**

1. **CPU Overload**

   **Solution:**
   ```bash
   # Lower encoding settings
   DEVICE_NAME_CODEC=opus
   DEVICE_NAME_SAMPLE_RATE=16000
   DEVICE_NAME_BITRATE=64k

   # Reduce number of streams
   # See Performance Tuning guide
   ```

2. **Memory Pressure**

   **Solution:**
   ```bash
   # Check memory usage
   free -h

   # Increase swap if needed
   sudo fallocate -l 2G /swapfile
   sudo chmod 600 /swapfile
   sudo mkswap /swapfile
   sudo swapon /swapfile

   # Make permanent in /etc/fstab
   echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
   ```

3. **Network Instability**

   **Solution:**
   ```bash
   # Check for packet loss
   ping -c 100 server-ip

   # Increase buffer sizes
   DEVICE_NAME_THREAD_QUEUE=4096

   # Use QoS on router to prioritize RTSP traffic
   ```

### Streams Fail to Auto-Recover

**Symptoms:**

- Failed streams stay failed
- No automatic restart attempts
- Manual restart required

**Diagnostic Steps:**

```bash
# Check if monitoring is active
sudo systemctl status mediamtx-stream-manager

# Check auto-recovery logs
sudo journalctl -u mediamtx-stream-manager | grep -i recover
```

**Solutions:**

1. **Monitoring Mode Not Running**

   **Solution:**
   ```bash
   # Install systemd service
   sudo ./mediamtx-stream-manager.sh install-service

   # Enable and start
   sudo systemctl enable mediamtx-stream-manager
   sudo systemctl start mediamtx-stream-manager

   # Verify monitoring active
   sudo systemctl status mediamtx-stream-manager
   ```

2. **Maximum Retry Attempts Reached**

   **Solution:**
   ```bash
   # Check retry count in logs
   sudo journalctl -u mediamtx-stream-manager | grep "attempt"

   # If max retries reached, fix underlying issue then restart
   sudo ./mediamtx-stream-manager.sh restart
   ```

3. **Persistent Hardware Issue**

   **Solution:**
   ```bash
   # Identify problematic device
   sudo ./lyrebird-diagnostics.sh

   # Test device manually
   arecord -D hw:CARD=DeviceName -d 5 test.wav

   # Replace device if hardware failure
   ```

---

## USB Persistence Problems

### Device Names Change After Reboot

**Symptoms:**

- Device is card0 before reboot, card1 after
- Streams fail after reboot
- Configuration references wrong device

**Solutions:**

1. **Regenerate udev Rules**
   ```bash
   # Create persistent device mapping
   sudo ./usb-audio-mapper.sh

   # Verify rules created
   cat /etc/udev/rules.d/99-usb-soundcards.rules

   # Reboot to test
   sudo reboot

   # Verify persistence
   cat /proc/asound/cards
   # Should show same card numbers
   ```

2. **Use Symbolic Links**
   ```bash
   # Reference devices by name, not card number
   # Instead of: hw:0
   # Use: hw:CARD=Blue_Yeti

   # Update configuration
   # Edit /etc/mediamtx/audio-devices.conf
   # All references should use CARD=<name> format
   ```

### Multiple Boot Ordering Issues

**Symptoms:**

- Devices swap card numbers unpredictably
- Different order each boot

**Solutions:**

```bash
# Create index-based udev rules
# Edit /etc/modprobe.d/alsa-base.conf

# Add device-specific index assignments
options snd-usb-audio index=0 vid=0x046d pid=0x0a44
options snd-usb-audio index=1 vid=0x0d8c pid=0x0014

# Update initramfs
sudo update-initramfs -u

# Reboot
sudo reboot
```

---

## MediaMTX Service Issues

### MediaMTX Won't Start

**Symptoms:**

- `systemctl start mediamtx` fails
- Service shows failed status
- Port 8554 not listening

**Diagnostic Steps:**

```bash
# Check service status
sudo systemctl status mediamtx

# View detailed logs
sudo journalctl -u mediamtx -n 50

# Check if port in use
sudo netstat -tlnp | grep 8554

# Test MediaMTX manually
sudo /usr/local/bin/mediamtx /etc/mediamtx/mediamtx.yml
```

**Solutions:**

1. **Port Already in Use**

   **Solution:**
   ```bash
   # Find process using port
   sudo lsof -i :8554

   # Kill conflicting process
   sudo kill <PID>

   # Or change MediaMTX port
   # Edit /etc/mediamtx/mediamtx.yml
   rtspAddress: :8555  # Use different port

   # Update stream configurations
   # Update URLs to use new port
   ```

2. **Configuration File Error**

   **Solution:**
   ```bash
   # Validate YAML syntax
   sudo /usr/local/bin/mediamtx --validate /etc/mediamtx/mediamtx.yml

   # Common issues:
   # - Incorrect indentation
   # - Missing colons
   # - Wrong data types

   # Restore from backup if needed
   sudo cp /etc/mediamtx/mediamtx.yml.backup /etc/mediamtx/mediamtx.yml
   ```

3. **Binary Not Found**

   **Solution:**
   ```bash
   # Check if MediaMTX installed
   which mediamtx

   # Reinstall if needed
   sudo ./install_mediamtx.sh

   # Verify installation
   /usr/local/bin/mediamtx --version
   ```

4. **Permission Issues**

   **Solution:**
   ```bash
   # Check binary permissions
   ls -l /usr/local/bin/mediamtx

   # Should be executable
   sudo chmod +x /usr/local/bin/mediamtx

   # Check config file permissions
   ls -l /etc/mediamtx/mediamtx.yml
   sudo chmod 644 /etc/mediamtx/mediamtx.yml
   ```

### MediaMTX API Not Responding

**Symptoms:**

- Cannot query stream status
- HTTP API timeouts
- Port 9997 not accessible

**Solutions:**

1. **API Not Enabled**

   **Solution:**
   ```bash
   # Edit /etc/mediamtx/mediamtx.yml
   # Ensure API enabled:
   api: yes
   apiAddress: :9997

   # Restart MediaMTX
   sudo systemctl restart mediamtx

   # Test API
   curl http://localhost:9997/v3/paths/list
   ```

2. **Firewall Blocking API**

   **Solution:**
   ```bash
   # Allow API port
   sudo ufw allow 9997/tcp

   # For external access
   sudo ufw allow from any to any port 9997 proto tcp
   ```

---

## Resource Exhaustion

### CPU Overload

**Symptoms:**

- System unresponsive
- High load average
- Stream stuttering

**Solutions:**

See [Performance Tuning](performance.md) for detailed optimization strategies:

```bash
# Quick fixes:
# 1. Reduce stream count
# 2. Lower quality settings
# 3. Switch to Opus codec
# 4. Reduce sample rate to 16kHz

# Check CPU usage
top -p $(pgrep ffmpeg | tr '\n' ',')
```

### Memory Exhaustion

**Symptoms:**

- OOM killer activating
- Processes being killed
- System swap thrashing

**Solutions:**

```bash
# Calculate required memory
# (Number of streams × 50 MB) + 200 MB base

# Add swap space
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Reduce stream count or upgrade RAM
```

### File Descriptor Exhaustion

**Symptoms:**

- "Too many open files" errors
- New connections rejected
- Streams fail to start

**Solutions:**

```bash
# Check limit
ulimit -n

# Increase permanently
# Edit /etc/security/limits.conf
* soft nofile 4096
* hard nofile 8192

# Reboot or re-login
sudo reboot
```

---

## Audio Quality Problems

### Poor Audio Quality

**Symptoms:**

- Distorted or muffled audio
- Background noise
- Low volume

**Solutions:**

1. **Increase Bitrate**
   ```bash
   DEVICE_NAME_BITRATE=192k  # Increase from 128k
   ```

2. **Increase Sample Rate**
   ```bash
   DEVICE_NAME_SAMPLE_RATE=48000  # Increase from 16000
   ```

3. **Switch to Better Codec**
   ```bash
   DEVICE_NAME_CODEC=aac  # Switch from mp3
   ```

4. **Check Device Capabilities**
   ```bash
   sudo ./lyrebird-mic-check.sh
   # Ensure device supports high quality settings
   ```

### Audio Clipping or Distortion

**Symptoms:**

- Audio peaks are cut off
- Digital distortion
- Crackles and pops

**Solutions:**

1. **Reduce Input Gain**
   ```bash
   # Use alsamixer to adjust gain
   alsamixer -c 0
   # Reduce capture level
   ```

2. **Increase Buffer Size**
   ```bash
   DEVICE_NAME_THREAD_QUEUE=2048
   ```

3. **Check USB Power**
   ```bash
   # Use powered USB hub
   # Check for voltage warnings
   ```

---

## Network Issues

### Cannot Connect to Streams Remotely

**Symptoms:**

- Local playback works (localhost)
- Remote clients cannot connect
- Connection timeout

**Solutions:**

1. **Firewall Blocking Connections**
   ```bash
   # Allow RTSP port
   sudo ufw allow 8554/tcp

   # Check firewall status
   sudo ufw status
   ```

2. **MediaMTX Bound to Localhost Only**
   ```bash
   # Edit /etc/mediamtx/mediamtx.yml
   # Change from:
   rtspAddress: 127.0.0.1:8554
   # To:
   rtspAddress: :8554  # Listen on all interfaces

   # Restart MediaMTX
   sudo systemctl restart mediamtx
   ```

3. **Incorrect URL**
   ```bash
   # Use server IP, not localhost
   # Correct: rtsp://192.168.1.100:8554/StreamName
   # Incorrect: rtsp://localhost:8554/StreamName

   # Find server IP
   ip addr show
   ```

---

## Best Practices

### Systematic Troubleshooting

1. **Start with Diagnostics**
   ```bash
   sudo ./lyrebird-diagnostics.sh --verbose
   ```

2. **Check Logs**
   ```bash
   sudo journalctl -u mediamtx -n 50
   sudo journalctl -u mediamtx-stream-manager -n 50
   tail -f /var/log/lyrebird/*.log
   ```

3. **Isolate the Problem**
   - Test one component at a time
   - Use manual FFmpeg commands
   - Test with minimal configuration

4. **Document Issues**
   - Record error messages
   - Note what changed before failure
   - Keep diagnostic reports

### Prevention

1. **Regular Health Checks**
   ```bash
   # Weekly diagnostics
   sudo ./lyrebird-diagnostics.sh
   ```

2. **Monitor Resource Usage**
   ```bash
   # Daily monitoring
   sudo ./mediamtx-stream-manager.sh monitor
   ```

3. **Backup Configurations**
   ```bash
   # Backup before changes
   sudo cp /etc/mediamtx/audio-devices.conf \
          /etc/mediamtx/audio-devices.conf.backup
   ```

4. **Keep Logs**
   ```bash
   # Enable log rotation
   # Create /etc/logrotate.d/lyrebird
   /var/log/lyrebird/*.log {
       weekly
       rotate 4
       compress
       missingok
       notifempty
   }
   ```

---

## Getting Help

### Information to Collect

When seeking help, collect:

1. **Diagnostic Report**
   ```bash
   sudo ./lyrebird-diagnostics.sh --export=/tmp/diagnostic-report.txt
   ```

2. **System Information**
   ```bash
   uname -a
   cat /etc/os-release
   ```

3. **Device Information**
   ```bash
   lsusb
   cat /proc/asound/cards
   arecord -l
   ```

4. **Service Logs**
   ```bash
   sudo journalctl -u mediamtx -n 100 > /tmp/mediamtx.log
   sudo journalctl -u mediamtx-stream-manager -n 100 > /tmp/stream-manager.log
   ```

5. **Configuration Files**
   ```bash
   cat /etc/mediamtx/audio-devices.conf
   cat /etc/mediamtx/mediamtx.yml
   ```

---

## Related Documentation

- [Diagnostics & Monitoring](diagnostics-monitoring.md) - Health check tools
- [Performance Tuning](performance.md) - Resource optimization
- [Architecture](architecture.md) - System components
- [Log Files](../reference/log-files.md) - Log locations and formats
- [Command Reference](../reference/command-reference.md) - All available commands

---

**Next Steps:** [Diagnostics & Monitoring](diagnostics-monitoring.md) | [Performance Tuning](performance.md)
