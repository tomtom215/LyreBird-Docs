# System Requirements

Hardware and software requirements for running LyreBirdAudio.

---

## Operating System

### Supported Distributions

- **Ubuntu** 20.04 LTS or later
- **Debian** 11 (Bullseye) or later
- **Raspberry Pi OS** (32-bit and 64-bit)
- **Other Linux distributions** with systemd support

### Minimum Kernel Version

- Linux kernel **4.9** or later
- Recommended: **5.4** or later for best USB audio support

---

## Hardware Requirements

### Minimum Specifications

| Component | Requirement |
|-----------|-------------|
| CPU | 1 GHz single-core (ARMv6 or higher) |
| RAM | 512 MB |
| Storage | 500 MB free space |
| USB | USB 2.0 port |

### Recommended Specifications

| Component | Recommendation |
|-----------|----------------|
| CPU | Dual-core 1.5 GHz or better |
| RAM | 1 GB or more |
| Storage | 2 GB free space |
| USB | USB 3.0 for multiple devices |

---

## Supported Architectures

LyreBirdAudio and MediaMTX support:

- **x86_64** (64-bit Intel/AMD)
- **ARM64** (aarch64)
- **ARMv7** (32-bit ARM)
- **ARMv6** (Raspberry Pi Zero/1)

---

## USB Audio Devices

### Compatible Devices

Any USB audio device with ALSA support:

- USB microphones
- USB audio interfaces
- USB sound cards with line-in
- USB headsets (microphone input)

### Device Limitations by Platform

#### Raspberry Pi 3/4/Zero 2 W

!!! warning "USB Bandwidth Limitation"
    Raspberry Pi devices share USB bandwidth across all ports. Maximum recommended:

    - **2 USB microphones** simultaneously
    - More than 2 may cause audio dropouts or device enumeration issues

#### Intel N100/N150 Mini PCs

!!! success "Recommended for Multiple Devices"
    Intel mini PCs provide dedicated USB controllers with better bandwidth:

    - **4+ USB microphones** without issues
    - More reliable for production deployments

---

## Software Dependencies

### Required Packages

=== "Ubuntu/Debian"

    ```bash
    sudo apt-get update
    sudo apt-get install \
        alsa-utils \
        udev \
        curl \
        ffmpeg
    ```

=== "Arch Linux"

    ```bash
    sudo pacman -S \
        alsa-utils \
        udev \
        curl \
        ffmpeg
    ```

### Automatic Installation

The orchestrator will check for and install missing dependencies automatically.

---

## Network Requirements

### Ports

| Port | Protocol | Purpose |
|------|----------|---------|
| 8554 | TCP/UDP | RTSP streaming (MediaMTX) |
| 8888 | TCP | MediaMTX Web UI (optional) |
| 8889 | TCP | MediaMTX API (optional) |

### Bandwidth

Per USB microphone stream:

- **AAC 128 kbps:** ~128 Kbps upstream
- **AAC 192 kbps:** ~192 Kbps upstream
- **PCM (uncompressed):** ~1.5 Mbps upstream

!!! tip "Local Network Deployment"
    For local network use, bandwidth requirements are minimal. Internet upload speed only matters for remote access.

---

## Storage Considerations

### System Files

- LyreBirdAudio scripts: ~100 KB
- MediaMTX binary: ~15-20 MB
- Configuration files: ~10 KB per stream

### Logging

With default settings:

- Log rotation configured automatically
- Logs limited to 50 MB per component
- Old logs automatically deleted

---

## Performance Expectations

### CPU Usage

Per stream (typical):

- **Raspberry Pi Zero W:** 15-25% CPU
- **Raspberry Pi 4:** 5-10% CPU
- **Intel N100:** 2-5% CPU

### Memory Usage

Per stream (typical):

- FFmpeg process: 20-40 MB
- MediaMTX server: 30-50 MB (shared across all streams)
- Stream manager: 5-10 MB

---

## Testing Your System

### Check USB Audio Devices

```bash
arecord -l
```

Expected output:
```
**** List of CAPTURE Hardware Devices ****
card 1: Device [USB Audio Device], device 0: USB Audio [USB Audio]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
```

### Check ALSA Version

```bash
arecord --version
```

Minimum required: **1.1.0**

### Check FFmpeg Codecs

```bash
ffmpeg -codecs | grep aac
```

Should show AAC encoder support.

### Check Available Disk Space

```bash
df -h /opt
```

Ensure at least 500 MB free.

---

## Recommended Platforms

### Production Deployments

1. **Intel N100/N150 Mini PCs** (Best choice)
   - Excellent USB reliability
   - Low power consumption (6W idle)
   - Supports 4+ microphones
   - ~$150 USD

2. **Raspberry Pi 4 (4GB+)** (Good choice)
   - Widely available
   - Good performance
   - Limit to 2 microphones
   - ~$75 USD

3. **x86_64 Desktop/Server** (Overkill but works)
   - Use existing hardware
   - Excellent reliability
   - No microphone limits

### Development/Testing

- Any Linux system with USB audio support
- Virtual machines work for testing (with USB passthrough)

---

## Not Supported

- **Windows** - ALSA and systemd not available
- **macOS** - Different audio subsystem
- **Docker/Containers** - USB device passthrough limitations
- **WSL** (Windows Subsystem for Linux) - Limited USB support

---

## Upgrade Recommendations

If you currently have:

| Current Setup | Recommendation |
|---------------|----------------|
| Raspberry Pi 3B | Upgrade to Pi 4 or Intel N100 for better USB support |
| 512 MB RAM | Upgrade to 1GB+ for multiple streams |
| USB 2.0 hub | Use powered hub or upgrade to USB 3.0 |

---

## Next Steps

Ready to install? Proceed to the installation guide.

[Installation Guide ->](installation.md){ .md-button .md-button--primary }
