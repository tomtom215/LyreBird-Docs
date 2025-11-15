# USB Audio Mapper

**Script:** `usb-audio-mapper.sh`
**Version:** 1.2.1
**Purpose:** Create persistent udev rules for USB audio devices

---

## Overview

The USB Audio Mapper is a critical component that solves one of the most challenging problems in USB audio: device naming consistency across reboots. By creating udev rules that map devices to their physical USB ports, it ensures that your USB microphones always get the same device names, regardless of enumeration order.

!!! tip "Reboot Required"
    After creating or modifying udev rules, a system reboot is required for the rules to take effect. This allows the kernel to re-enumerate USB devices with the new naming scheme.

---

## The Problem It Solves

### USB Enumeration Order

Without persistent naming, USB devices are enumerated in unpredictable order:

**Boot 1:**
```
USB Mic in port 1-1.4 → /dev/snd/pcmC0D0c (card 0)
USB Mic in port 1-1.5 → /dev/snd/pcmC1D0c (card 1)
USB Mic in port 2-1.2 → /dev/snd/pcmC2D0c (card 2)
```

**Boot 2 (after power cycle):**
```
USB Mic in port 2-1.2 → /dev/snd/pcmC0D0c (card 0)  ← Different!
USB Mic in port 1-1.5 → /dev/snd/pcmC1D0c (card 1)  ← Different!
USB Mic in port 1-1.4 → /dev/snd/pcmC2D0c (card 2)  ← Different!
```

**Result:** Your streams point to wrong devices or fail entirely.

### The Solution: Physical Port Mapping

The USB Audio Mapper creates udev rules that map each USB port to a consistent device name:

```
Port 1-1.4 → /dev/sound/by-id/Device_1 → /dev/snd/pcmC0D0c
Port 1-1.5 → /dev/sound/by-id/Device_2 → /dev/snd/pcmC1D0c
Port 2-1.2 → /dev/sound/by-id/Device_3 → /dev/snd/pcmC2D0c
```

**Regardless of enumeration order**, each physical port always maps to the same friendly name.

---

## Key Features

<div class="grid" markdown>

<div markdown>
### :material-usb-port: Physical Port Mapping
Maps devices to actual USB ports on your system, not enumeration order.
</div>

<div markdown>
### :material-content-duplicate: Identical Device Support
Handle multiple identical USB microphones by their physical location.
</div>

<div markdown>
### :material-wizard-hat: Interactive Wizard
Step-by-step guided setup for easy device mapping.
</div>

<div markdown>
### :material-robot: Automation Support
Non-interactive mode for scripted deployments.
</div>

<div markdown>
### :material-topology-star-3: Platform ID Support
Handle complex USB topologies with platform-specific paths.
</div>

<div markdown>
### :material-history: Backwards Compatible
Maintains compatibility with existing udev rule formats.
</div>

</div>

---

## Usage

### Interactive Mode (Recommended)

The interactive wizard guides you through device mapping:

```bash
sudo ./usb-audio-mapper.sh
```

**Wizard Steps:**

1. **Device Detection**: Scans for connected USB audio devices
2. **Device Selection**: Shows list of detected devices with details
3. **Port Identification**: Identifies physical USB port for selected device
4. **Friendly Name**: Prompts for easy-to-remember device name
5. **Rule Generation**: Creates udev rule and writes to `/etc/udev/rules.d/99-usb-soundcards.rules`

**Example Session:**
```
===== USB Sound Card Mapper =====

[Shows lsusb output]
[Shows /proc/asound/cards output]

Enter the number of the sound card you want to map: 0

Enter friendly name for this device: front-yard-mic

[USB devices listed with numbers]

Select USB device (1-N): 1

Rule written to /etc/udev/rules.d/99-usb-soundcards.rules

Reboot required for changes to take effect.
```

**Note:** The actual rule generated will be:
```
SUBSYSTEM=="sound", ATTRS{idVendor}=="046d", ATTRS{idProduct}=="0825",
KERNELS=="1-1.4", ATTR{id}="front-yard-mic", SYMLINK+="sound/by-id/front-yard-mic"
```

---

### Non-Interactive Mode

For automation and scripting:

```bash
sudo ./usb-audio-mapper.sh -n \
  -d "USB Microphone" \
  -v 046d \
  -p 0825 \
  -f front-yard-mic
```

**Parameters:**
- `-n, --non-interactive`: Skip interactive prompts
- `-d, --device <name>`: Device name for logging
- `-v, --vendor <id>`: USB Vendor ID (4-digit hex)
- `-p, --product <id>`: USB Product ID (4-digit hex)
- `-f, --friendly <name>`: Friendly name for device
- `-u, --usb-port <path>`: USB port path (optional, auto-detected)

---

### Test Mode

Verify USB port detection without making changes:

```bash
sudo ./usb-audio-mapper.sh --test
```

**Output:**
```
Testing USB Audio Device Detection:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Device 1:
  Name: USB Microphone
  Vendor ID: 046d
  Product ID: 0825
  USB Port: 1-1.4
  Current Path: /dev/snd/pcmC0D0c

Device 2:
  Name: USB Audio Interface
  Vendor ID: 0d8c
  Product ID: 0014
  USB Port: 1-1.5
  Current Path: /dev/snd/pcmC1D0c
```

---

### Debug Mode

Enable verbose output for troubleshooting:

```bash
sudo ./usb-audio-mapper.sh --debug
```

Shows detailed information about:
- USB device enumeration
- Port path detection logic
- udev rule construction
- File system operations

---

## Command Options

### Complete Option Reference

| Option | Short | Description |
|--------|-------|-------------|
| `--help` | `-h` | Show help message and exit |
| `--interactive` | `-i` | Run in interactive mode (default) |
| `--non-interactive` | `-n` | Run without prompts (requires other params) |
| `--device <name>` | `-d` | Device name for logging purposes |
| `--vendor <id>` | `-v` | USB Vendor ID (4-digit hexadecimal) |
| `--product <id>` | `-p` | USB Product ID (4-digit hexadecimal) |
| `--usb-port <path>` | `-u` | USB port path (e.g., "1-1.4") - auto-detected if omitted |
| `--friendly <name>` | `-f` | Friendly name for device symlink |
| `--test` | `-t` | Test mode: detect devices without creating rules |
| `--debug` | `-D` | Enable verbose debug output |

---

## Generated Files

### udev Rules File

**Location:** `/etc/udev/rules.d/99-usb-soundcards.rules`

**Format:**
```bash
# USB Microphone - Front Yard
SUBSYSTEM=="sound", ATTRS{idVendor}=="046d", ATTRS{idProduct}=="0825", KERNELS=="1-1.4", ATTR{id}="front-yard-mic", SYMLINK+="sound/by-id/front-yard-mic"

# USB Microphone - Back Yard
SUBSYSTEM=="sound", ATTRS{idVendor}=="046d", ATTRS{idProduct}=="0825", KERNELS=="1-1.5", ATTR{id}="back-yard-mic", SYMLINK+="sound/by-id/back-yard-mic"

# USB Audio Interface - Studio
SUBSYSTEM=="sound", ATTRS{idVendor}=="0d8c", ATTRS{idProduct}=="0014", KERNELS=="2-1.2", ATTR{id}="studio-interface", SYMLINK+="sound/by-id/studio-interface"
```

**Rule Breakdown:**
- `SUBSYSTEM=="sound"`: Match sound subsystem devices
- `ATTRS{idVendor}=="046d"`: Match USB Vendor ID
- `ATTRS{idProduct}=="0825"`: Match USB Product ID
- `KERNELS=="1-1.4"`: Match specific USB port (or `ENV{ID_PATH}` for complex topologies)
- `ATTR{id}="front-yard-mic"`: Set ALSA card ID
- `SYMLINK+="sound/by-id/..."`: Create persistent symlink

### Device Symlinks

**Location:** `/dev/sound/by-id/`

**Created After Reboot:**
```
/dev/sound/by-id/
├── front-yard-mic -> ../../snd/controlC0
├── back-yard-mic -> ../../snd/controlC1
└── studio-interface -> ../../snd/controlC2
```

**Usage in Configuration:**
```bash
# Reference by friendly name instead of card number
ffmpeg -f alsa -i hw:CARD=front-yard-mic ...
```

---

## How It Works

### USB Port Path Detection

The mapper uses the Linux USB subsystem to identify physical port locations:

```mermaid
graph TD
    A[USB Device Connected] --> B[Kernel Assigns Card Number]
    B --> C[Mapper Reads /sys/class/sound/cardN]
    C --> D[Follows device symlink]
    D --> E[Extracts USB Port from Path]
    E --> F[/sys/devices/.../1-1.4/...]
    F --> G[Creates udev Rule]
    G --> H[Maps Port to Friendly Name]
    H --> I[Reboot]
    I --> J[/dev/sound/by-id/device-name]

    style G fill:#4051b5,color:#fff
    style J fill:#00bcd4,color:#fff
```

### Port Path Examples

**Simple Topology:**
```
1-1.4 → USB Hub Port 4 on Bus 1
1-1.5 → USB Hub Port 5 on Bus 1
2-1.2 → USB Hub Port 2 on Bus 2
```

**Complex Topology (Cascaded Hubs):**
```
1-1.4.2 → Bus 1, Hub Port 4, Cascaded Hub Port 2
1-1.4.3 → Bus 1, Hub Port 4, Cascaded Hub Port 3
```

The mapper handles both simple and complex topologies automatically.

---

## Platform ID Support

For systems with complex USB architectures, the mapper supports platform-specific device identification:

**Platform ID Format:**
```
platform-xhci-hcd.0-usb-0:1.4:1.0
```

**When Platform IDs Are Used:**
- Multiple USB controllers (e.g., Intel + ASMedia)
- USB-C with alternate modes
- Thunderbolt USB devices
- Complex embedded systems

**Automatic Detection:**
The mapper automatically chooses the most stable identifier:
1. Attempts standard USB port path
2. Falls back to platform ID if port path is unstable
3. Uses both for maximum reliability

---

## Multiple Identical Devices

### The Challenge

When you have multiple identical USB microphones:
- Same Vendor ID
- Same Product ID
- Same product string

**Traditional udev rules can't distinguish them.**

### The Solution

The USB Audio Mapper uses **physical port location** as the differentiator:

```bash
# Two identical microphones - differentiated by USB port
SUBSYSTEM=="sound", ATTRS{idVendor}=="046d", ATTRS{idProduct}=="0825",
KERNELS=="1-1.4", ATTR{id}="mic-left", SYMLINK+="sound/by-id/mic-left"

SUBSYSTEM=="sound", ATTRS{idVendor}=="046d", ATTRS{idProduct}=="0825",
KERNELS=="1-1.5", ATTR{id}="mic-right", SYMLINK+="sound/by-id/mic-right"
```

**Result:**
- Microphone in port 1-1.4 → Always `mic-left`
- Microphone in port 1-1.5 → Always `mic-right`

!!! warning "Don't Move Devices Between Ports"
    Once mapped, keep each microphone in its assigned USB port. Moving devices between ports will break the mapping until you re-run the mapper.

---

## Backwards Compatibility

The mapper maintains compatibility with older naming schemes:

**Legacy Format (Serial Number Suffix):**
```
Device_1_ABC123
Device_2_DEF456
```

**Current Format (No Suffix):**
```
Device_1
Device_2
```

Both formats are supported. Existing installations continue to work without modification.

---

## Workflow Examples

### First-Time Setup

```bash
# 1. Connect USB microphone to desired port
# 2. Run mapper interactively
sudo ./usb-audio-mapper.sh

# 3. Follow wizard prompts
# 4. Reboot system
sudo reboot

# 5. Verify symlink creation
ls -la /dev/sound/by-id/

# 6. Use in stream configuration
sudo ./mediamtx-stream-manager.sh start
```

---

### Adding New Device

```bash
# 1. Connect new USB microphone
# 2. Run mapper again
sudo ./usb-audio-mapper.sh

# 3. Select the new device from list
# 4. Provide friendly name
# 5. Reboot
sudo reboot

# 6. Update stream configuration
sudo ./lyrebird-mic-check.sh -g
sudo ./mediamtx-stream-manager.sh restart
```

---

### Automated Deployment

```bash
#!/bin/bash
# deploy-microphones.sh

# Map three identical microphones to different ports
sudo ./usb-audio-mapper.sh -n -v 046d -p 0825 -u 1-1.4 -f mic-north
sudo ./usb-audio-mapper.sh -n -v 046d -p 0825 -u 1-1.5 -f mic-south
sudo ./usb-audio-mapper.sh -n -v 046d -p 0825 -u 2-1.2 -f mic-east

# Reload udev rules
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=sound

# Reboot recommended but can try without
echo "Reboot recommended: sudo reboot"
```

---

### Verifying Mappings

```bash
# Check if rules file exists
cat /etc/udev/rules.d/99-usb-soundcards.rules

# Test current mappings (before reboot)
sudo ./usb-audio-mapper.sh --test

# After reboot, verify symlinks
ls -la /dev/sound/by-id/

# Test with ALSA (use the friendly name directly)
arecord -L | grep -i front-yard-mic
```

---

## Troubleshooting

### Symlinks Not Created After Reboot

**Symptoms:** `/dev/sound/by-id/` directory is empty or symlinks missing

**Solutions:**

```bash
# 1. Check if udev rules file exists
ls -l /etc/udev/rules.d/99-usb-soundcards.rules

# 2. Verify rules syntax
cat /etc/udev/rules.d/99-usb-soundcards.rules

# 3. Reload udev rules
sudo udevadm control --reload-rules

# 4. Trigger udev to re-apply rules
sudo udevadm trigger

# 5. Check udev logs for errors
sudo journalctl -u systemd-udevd | grep -i error

# 6. Test specific rule
sudo udevadm test /sys/class/sound/card0
```

---

### Device Mapping Changes After Reboot

**Symptoms:** Device gets different name after reboot

**Possible Causes:**

1. **USB Port Path Changed:**
   - USB device was physically moved to different port
   - USB hub replaced or reconfigured

2. **USB Enumeration Timing:**
   - Devices powered on in different order
   - USB bus reset during boot

**Solutions:**

```bash
# 1. Verify current USB port
lsusb -t

# 2. Re-map device to current port
sudo ./usb-audio-mapper.sh

# 3. For persistent port changes, update rule manually
sudo nano /etc/udev/rules.d/99-usb-soundcards.rules
```

---

### Multiple Devices Get Same Symlink

**Symptoms:** Two devices compete for the same device name

**Cause:** Identical devices without port specification

**Solution:**

```bash
# Ensure each rule has unique KERNELS value
KERNELS=="1-1.4"  # Device 1
KERNELS=="1-1.5"  # Device 2

# Verify no duplicate KERNELS in rules file
grep KERNELS /etc/udev/rules.d/99-usb-soundcards.rules | sort | uniq -d
```

---

### Permission Issues

**Symptoms:** "Permission denied" when running mapper

**Solution:**

```bash
# Run with sudo
sudo ./usb-audio-mapper.sh

# If issues persist, check script permissions
ls -l usb-audio-mapper.sh
chmod +x usb-audio-mapper.sh
```

---

## Integration with Other Components

### With Capability Checker

The capability checker can use friendly names:

```bash
# Generate config using friendly names
sudo ./usb-audio-mapper.sh  # Create mappings
sudo reboot
sudo ./lyrebird-mic-check.sh -g  # Auto-detects friendly names
```

**Result in `/etc/mediamtx/audio-devices.conf`:**
```bash
DEVICE_FRONT_YARD_MIC_SAMPLE_RATE=48000
DEVICE_FRONT_YARD_MIC_CHANNELS=2
DEVICE_BACK_YARD_MIC_SAMPLE_RATE=48000
DEVICE_BACK_YARD_MIC_CHANNELS=2
```

### With Stream Manager

The stream manager automatically uses friendly names:

```bash
# Streams created with friendly names
rtsp://host:8554/front-yard-mic
rtsp://host:8554/back-yard-mic
rtsp://host:8554/studio-interface
```

### With Orchestrator

The orchestrator integrates USB mapping into setup workflows:

**Quick Setup Wizard:**
1. Install MediaMTX
2. **Run USB Audio Mapper** ← Automatic
3. Generate configuration
4. Start streams
5. Install systemd service

---

## Best Practices

!!! tip "Label Physical Ports"
    Use labels on your USB ports matching the friendly names (e.g., "FRONT-YARD-MIC" label on port 1-1.4).

!!! tip "Document Your Topology"
    Keep a diagram showing which USB port corresponds to which physical location/microphone.

!!! tip "Use Descriptive Names"
    Choose friendly names that indicate location or purpose: `north-fence`, `backyard-tree`, not `device1`.

!!! warning "Avoid USB Hubs When Possible"
    Direct connections to motherboard USB ports are more stable than hubs, especially for multiple devices.

!!! warning "Always Reboot After Mapping"
    udev rules only take effect after a reboot. Don't rely on `udevadm trigger` for production.

---

## Advanced Usage

### Custom Rule Attributes

You can manually edit rules to add additional matching criteria:

```bash
# Match by serial number (if available)
SUBSYSTEM=="sound", ATTRS{idVendor}=="046d", ATTRS{idProduct}=="0825",
ATTR{id}="mic-serial-abc", SYMLINK+="sound/by-id/mic-serial-abc"

# Match by bus and port
SUBSYSTEM=="sound", ATTRS{busnum}=="1", ATTRS{devnum}=="4",
ATTR{id}="mic-bus1-dev4", SYMLINK+="sound/by-id/mic-bus1-dev4"
```

### Multiple Symlinks per Device

Create multiple symlinks for the same device:

```bash
# Device accessible via multiple names
SUBSYSTEM=="sound", ATTRS{idVendor}=="046d", ATTRS{idProduct}=="0825",
KERNELS=="1-1.4", ATTR{id}="front-yard-mic", SYMLINK+="sound/by-id/front-yard-mic sound/by-id/mic-1"
```

**Result:**
- `/dev/sound/by-id/front-yard-mic` → `../../snd/controlC0`
- `/dev/sound/by-id/mic-1` → `../../snd/controlC0`

---

## Related Documentation

- **[Orchestrator](orchestrator.md)** - Includes USB mapping in setup wizard
- **[Capability Checker](capability-checker.md)** - Uses persistent names for config
- **[Stream Manager](stream-manager.md)** - Creates streams using persistent names
- **[User Guide: USB Device Management](../user-guide/usb-device-management.md)** - Operational guide
- **[Advanced: Troubleshooting](../advanced/troubleshooting.md)** - Device naming issues

---

## See Also

- [Getting Started: Installation](../getting-started/installation.md)
- [Getting Started: Quick Start](../getting-started/quick-start.md)
- [Reference: Configuration Files](../reference/configuration-files.md)
- [Linux udev Documentation](https://www.freedesktop.org/software/systemd/man/udev.html)
