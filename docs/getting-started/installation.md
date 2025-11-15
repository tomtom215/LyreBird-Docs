# Installation

Complete installation guide for LyreBirdAudio.

---

## Overview

LyreBirdAudio installation involves:

1. Cloning the repository
2. Running the orchestrator wizard
3. Installing MediaMTX
4. Mapping USB devices
5. Configuring streams
6. Installing the systemd service

Total time: **10-15 minutes**

---

## Step 1: Verify Prerequisites

Ensure your system meets the requirements:

```bash
# Check Linux distribution
cat /etc/os-release

# Check kernel version (should be 4.9+)
uname -r

# Check available disk space (need 500MB+)
df -h /opt

# List USB audio devices
arecord -l
```

!!! warning "No Audio Devices Found?"
    If `arecord -l` shows no devices, check:

    - USB microphone is connected
    - USB cable is functional
    - Device drivers are loaded (`lsusb` should show the device)

---

## Step 2: Install Dependencies

=== "Ubuntu/Debian"

    ```bash
    sudo apt-get update
    sudo apt-get install -y \
        git \
        alsa-utils \
        udev \
        curl \
        ffmpeg
    ```

=== "Raspberry Pi OS"

    ```bash
    sudo apt-get update
    sudo apt-get install -y \
        git \
        alsa-utils \
        udev \
        curl \
        ffmpeg
    ```

=== "Arch Linux"

    ```bash
    sudo pacman -Sy
    sudo pacman -S \
        git \
        alsa-utils \
        udev \
        curl \
        ffmpeg
    ```

---

## Step 3: Clone Repository

```bash
# Choose installation location
cd /opt

# Clone the repository
sudo git clone https://github.com/tomtom215/LyreBirdAudio.git

# Change ownership (replace 'youruser' with your username)
sudo chown -R youruser:youruser LyreBirdAudio

# Enter directory
cd LyreBirdAudio

# Verify download
ls -l
```

Expected files:
```
lyrebird-orchestrator.sh
mediamtx-stream-manager.sh
usb-audio-mapper.sh
lyrebird-mic-check.sh
lyrebird-diagnostics.sh
lyrebird-updater.sh
install_mediamtx.sh
README.md
LICENSE
```

---

## Step 4: Run the Orchestrator

```bash
sudo ./lyrebird-orchestrator.sh
```

You'll see the main menu:

```
========================================
LyreBirdAudio Orchestrator v2.1.0
========================================

Installation Directory: /opt/LyreBirdAudio
MediaMTX Install Path: /opt/mediamtx
System Architecture: x86_64

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

## Step 5: Install MediaMTX

Select **Option 1** from the menu.

The installer will:

1. Detect your system architecture
2. Download the latest MediaMTX release
3. Extract to `/opt/mediamtx`
4. Create default configuration
5. Set appropriate permissions

Output:
```
Detecting system architecture...
Architecture: x86_64

Downloading MediaMTX v1.8.0...
##################################################  100%

Extracting MediaMTX...
Installing to /opt/mediamtx...

MediaMTX installed successfully!
Binary: /opt/mediamtx/mediamtx
Config: /opt/mediamtx/mediamtx.yml
```

!!! success "MediaMTX Installed"
    MediaMTX is now ready. Proceed to device mapping.

---

## Step 6: Map USB Audio Devices

Select **Option 2** from the menu.

The mapper will:

1. Scan for USB audio devices
2. Display device information
3. Create persistent device names
4. Configure udev rules

Interactive session:
```
Scanning for USB audio devices...

Found USB Audio Devices:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Card 1: USB Audio Device
  Vendor: Generic
  Product: USB2.0 Device
  Serial: 123456789ABC
  Device Path: /dev/snd/pcmC1D0c

Map this device? (y/n): y
Enter friendly name (e.g., mic-front-door): mic-1

Creating persistent device link: /dev/lyrebird-mic-1
Writing udev rule: /etc/udev/rules.d/99-lyrebird-mic-1.rules
Reloading udev rules...

[PASS] Device mapped successfully!
```

Repeat for additional microphones.

---

## Step 7: Create First Stream

Select **Option 4** from the menu.

The stream wizard will:

1. List mapped devices
2. Auto-detect audio capabilities
3. Suggest optimal settings
4. Create stream configuration

Interactive session:
```
Available Devices:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. /dev/lyrebird-mic-1 (USB Audio Device)

Select device number: 1

Detecting audio capabilities...
Supported sample rates: 44100, 48000
Supported formats: S16_LE, S24_LE

Stream Configuration:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Device: /dev/lyrebird-mic-1
Stream Name: lyrebird-mic-1
Sample Rate: 48000 Hz
Audio Codec: AAC
Bitrate: 128 kbps

Create this stream? (y/n): y

[PASS] Stream configuration created
  Config file: /opt/mediamtx/streams/lyrebird-mic-1.env
  RTSP URL: rtsp://localhost:8554/lyrebird-mic-1
```

---

## Step 8: Install Systemd Service

Select **Option 8** from the menu.

The service installer will:

1. Create systemd service file
2. Enable auto-start on boot
3. Start the service immediately

Output:
```
Installing Stream Manager Service...

Creating service file: /etc/systemd/system/mediamtx-audio.service
Reloading systemd daemon...
Enabling service for auto-start...
Starting service...

[PASS] Service installed and started

Service Status:
● mediamtx-audio.service - MediaMTX Stream Manager
   Loaded: loaded
   Active: active (running) since [timestamp]
   Process: MediaMTX Server, Stream Monitor
```

---

## Step 9: Verify Installation

### Check Service Status

```bash
sudo systemctl status mediamtx-audio
```

### List Active Streams

```bash
sudo ./mediamtx-stream-manager.sh status
```

### Test Stream Playback

```bash
# Test with FFmpeg (10 second capture)
ffmpeg -i rtsp://localhost:8554/lyrebird-mic-1 -t 10 -acodec copy test.aac

# Test with VLC
vlc rtsp://localhost:8554/lyrebird-mic-1
```

!!! success "Installation Complete!"
    Your LyreBirdAudio system is now running and will start automatically on boot.

---

## Post-Installation

### Enable MediaMTX Web UI (Optional)

Edit `/opt/mediamtx/mediamtx.yml`:

```yaml
webrtcAddress: :8889
api: yes
apiAddress: :9997
```

Restart services:
```bash
sudo systemctl restart mediamtx-stream-manager
```

Access Web UI: `http://your-server-ip:8889`

---

### Configure Firewall (If Applicable)

```bash
# Allow RTSP traffic
sudo ufw allow 8554/tcp
sudo ufw allow 8554/udp

# Optional: Web UI
sudo ufw allow 8889/tcp
```

---

### Setup Monitoring (Recommended)

Create a cron job for health checks:

```bash
sudo crontab -e
```

Add:
```
*/5 * * * * /opt/LyreBirdAudio/lyrebird-diagnostics.sh quick >> /var/log/lyrebird-health.log 2>&1
```

---

## Verification Checklist

After installation, verify:

- [ ] MediaMTX binary exists at `/opt/mediamtx/mediamtx`
- [ ] USB devices mapped with persistent names
- [ ] Stream configuration files created
- [ ] Systemd service running
- [ ] Service enabled for auto-start
- [ ] Streams accessible via RTSP
- [ ] No errors in logs

Check logs:
```bash
journalctl -u mediamtx-audio -n 50
```

---

## Troubleshooting

### Issue: "Permission Denied"

Run orchestrator with sudo:
```bash
sudo ./lyrebird-orchestrator.sh
```

### Issue: "MediaMTX Download Failed"

Check internet connection:
```bash
curl -I https://github.com
```

Download manually: [MediaMTX Releases](https://github.com/bluenviron/mediamtx/releases)

### Issue: "No USB Audio Devices Found"

Check device connection:
```bash
lsusb
arecord -l
```

### Issue: "Stream Won't Start"

Check device capabilities:
```bash
# List all devices and their capabilities
sudo ./lyrebird-mic-check.sh
```

---

## Uninstall

To remove LyreBirdAudio:

```bash
# Stop and disable service
sudo systemctl stop mediamtx-audio
sudo systemctl disable mediamtx-audio
sudo rm /etc/systemd/system/mediamtx-audio.service

# Remove udev rules
sudo rm /etc/udev/rules.d/99-usb-soundcards.rules
sudo udevadm control --reload-rules

# Remove software
sudo rm -rf /opt/LyreBirdAudio
sudo rm -rf /opt/mediamtx
```

---

## Next Steps

<div class="grid" markdown>

<div markdown>
###  Basic Usage
Learn how to manage streams and monitor your system

[Basic Usage ->](basic-usage.md)
</div>

<div markdown>
###  Configuration
Configure advanced audio settings and stream options

[Configuration ->](../user-guide/configuration.md)
</div>

<div markdown>
###  Multiple Devices
Add and manage additional USB microphones

[USB Management ->](../user-guide/usb-device-management.md)
</div>

<div markdown>
###  Troubleshooting
Solutions for common issues

[Troubleshooting ->](../advanced/troubleshooting.md)
</div>

</div>
