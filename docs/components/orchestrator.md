# Orchestrator

**Script:** `lyrebird-orchestrator.sh`
**Version:** 2.1.0
**Purpose:** Unified management interface for LyreBirdAudio

---

## Overview

The Orchestrator is the central command interface for LyreBirdAudio, providing an interactive menu-driven system for managing all aspects of your RTSP audio streaming infrastructure. It acts as a unified frontend that delegates to specialized scripts while maintaining no duplicate business logic.

!!! tip "Recommended Entry Point"
    The Orchestrator is the recommended way to manage LyreBirdAudio. It provides guided workflows for common tasks and integrates all components into a single coherent interface.

---

## Key Features

<div class="grid" markdown>

<div markdown>
### Quick Setup Wizard
Complete installation from zero to streaming in minutes with automated guidance through all steps.
</div>

<div markdown>
### Device Capability Inspection
Automatically detect and validate hardware capabilities without manual configuration.
</div>

<div markdown>
### Real-Time System Status
Live monitoring of MediaMTX service, active streams, and system resources.
</div>

<div markdown>
### SHA256 Integrity Checking
Verify script integrity during updates to ensure secure operation.
</div>

<div markdown>
### Auto Log Rotation
Automatic management of log files to prevent disk space issues.
</div>

<div markdown>
### EOF/stdin Handling
Robust handling of user input with graceful error recovery.
</div>

</div>

---

## Usage

### Basic Invocation

```bash
sudo ./lyrebird-orchestrator.sh
```

!!! warning "Root Privileges Required"
    The Orchestrator requires root/sudo access for system-level operations like udev rule management, systemd service installation, and MediaMTX configuration.

---

## Main Menu Structure

When you launch the Orchestrator, you'll see an interactive menu:

```text
========================================
LyreBirdAudio Orchestrator v2.1.0
========================================

Menu Options:
  1) Quick Setup Wizard
  2) MediaMTX Installation & Updates
  3) USB Device Management
  4) Audio Streaming Control
  5) System Diagnostics
  6) Version Management
  7) Logs & Status
  0) Exit

Select an option:
```

---

## Menu Options Explained

### 1. Quick Setup Wizard

The Quick Setup Wizard guides you through a complete installation:

**Wizard Steps:**

1. **MediaMTX Installation (Step 1/4)**
   - Detects your system architecture automatically
   - Downloads and installs the latest MediaMTX version
   - Configures the server for audio streaming
   - Creates systemd service

2. **USB Device Mapping (Step 2/4)**
   - Scans for connected USB audio devices
   - Creates persistent device names via udev rules
   - Maps devices to physical USB ports
   - Generates `/dev/sound/by-id/*` symlinks

3. **Start Audio Streams (Step 3/4)**
   - Launches FFmpeg pipelines for all detected devices
   - Validates RTSP connectivity
   - Confirms stream health via MediaMTX API

4. **Run Quick Diagnostics (Step 4/4)**
   - Executes quick system health check
   - Validates USB devices, MediaMTX service, RTSP connectivity
   - Reports any issues with remediation steps

**Note:** The wizard does NOT automatically generate device configuration. Use option 3 (USB Device Management) -> option 5 (Generate Device Configuration) after the wizard completes if you need custom quality settings.

**Usage:**
```bash
sudo ./lyrebird-orchestrator.sh
# Select: 1. Quick Setup Wizard
# Follow interactive prompts
```

---

### 2. MediaMTX Installation & Updates

Manage MediaMTX server installation and updates:

**Sub-Options:**

1. Install MediaMTX (Fresh Install)
2. Update MediaMTX
3. Reinstall MediaMTX (Force)
4. Uninstall MediaMTX
5. Check Installation Status
0. Back to Main Menu

**Delegated to:** `install_mediamtx.sh`

**Features:**
- Platform-aware installation (Linux/Darwin/FreeBSD)
- Architecture detection (x86_64, ARM64, ARMv7, ARMv6)
- GitHub release fetching with checksums
- Atomic updates with rollback capability
- Systemd service configuration

---

### 3. USB Device Management

Manage USB audio device persistence and mapping:

**Sub-Options:**

1. Map USB Audio Devices (Interactive)
2. Test Device Mapping (Dry-run)
3. View Current Mappings
4. Detect Device Capabilities
5. Generate Device Configuration
6. Validate Device Configuration
7. Remove Device Mappings
8. Reload udev Rules
0. Back to Main Menu

**Delegated to:** `usb-audio-mapper.sh` (options 1-3, 7-8), `lyrebird-mic-check.sh` (options 4-6)

**Features:**
- Physical USB port mapping
- Support for multiple identical devices
- Automatic hardware capability detection (new in v2.1.0)
- Optimal configuration generation based on hardware
- Configuration validation against actual capabilities
- udev rule generation
- Persistent naming across reboots

**Generated Files:**
- `/etc/udev/rules.d/99-usb-soundcards.rules`
- `/dev/sound/by-id/Device_N` symlinks (after reboot)
- `/etc/mediamtx/audio-devices.conf` (from capability detection)

---

### 4. Audio Streaming Control

Control stream lifecycle and configuration:

**Sub-Options:**

1. Start All Streams
2. Stop All Streams
3. Restart All Streams
4. View Stream Status
5. View Stream URLs
6. Monitor Stream Health
0. Back to Main Menu

**Delegated to:** `mediamtx-stream-manager.sh`

**Features:**
- Individual and multiplex streaming modes
- Automatic health monitoring
- Process supervision with exponential backoff
- CPU and file descriptor monitoring
- Graceful shutdown procedures

---

### 5. System Diagnostics

Run comprehensive system health checks:

**Sub-Options:**

1. Quick Diagnostics (Fast)
2. Full System Diagnostics
3. Verbose Diagnostics (Detailed)
4. Export Diagnostic Report
0. Back to Main Menu

**Delegated to:** `lyrebird-diagnostics.sh`

**Check Categories:**
- System info (OS, kernel, uptime, load)
- USB audio devices (detection, ALSA, mapping)
- MediaMTX service (binary, version, config, status)
- Stream status (active count, health, uptime)
- RTSP connectivity (port 8554, protocol validation)
- System resources (CPU, memory, file descriptors)
- Log analysis (errors, warnings, failures)
- Time synchronization (NTP, Chrony, drift)

---

### 6. Version Management

Manage script updates and rollbacks:

**Sub-Options:**

1. Launch Update Manager (Interactive)
2. Show Component Versions
0. Back to Main Menu

**Note:** The Update Manager (option 1) provides an interactive interface with additional sub-options for switching versions, checking updates, and managing git state.

**Delegated to:** `lyrebird-updater.sh`

**Features:**
- Git-based version management
- Tagged release support
- Automatic stashing of local changes
- Transaction-based updates with rollback
- Systemd service coordination
- Self-update capability

---

### 7. Logs & Status

View logs and system status:

**Sub-Options:**

1. View MediaMTX Log (Last 50 lines)
2. View Orchestrator Log
3. View Stream Manager Log
4. View Stream Logs (FFmpeg)
5. Quick System Health Check
0. Back to Main Menu

**Note:** Log files are located at:
- MediaMTX: `/var/log/mediamtx.out`
- Orchestrator: `/var/log/lyrebird-orchestrator.log`
- Stream Manager: `/var/log/mediamtx-stream-manager.log`
- FFmpeg Streams: `/var/log/lyrebird/*.log`

---

## Architecture

### Design Philosophy

The Orchestrator follows the **Single-Responsibility Principle**:

- **No Business Logic**: Orchestrator contains only UI and delegation logic
- **Delegates to Specialists**: Each operation calls the appropriate specialized script
- **Consistent Error Handling**: Unified error reporting across all operations
- **Comprehensive Feedback**: Clear progress indicators and status messages

**Orchestrator Delegation Architecture:** The User interacts with the Orchestrator TUI (text user interface, shown in purple). The Orchestrator delegates different operations to six specialized scripts (all shown in purple): install_mediamtx.sh for installation/updates, mediamtx-stream-manager.sh for stream control, usb-audio-mapper.sh for device mapping, lyrebird-mic-check.sh for capability checking, lyrebird-diagnostics.sh for health checks, and lyrebird-updater.sh for version updates. These scripts then interact with actual system components: install_mediamtx.sh produces MediaMTX Binary, stream-manager produces FFmpeg Pipelines, usb-audio-mapper creates udev Rules, and capability-checker generates Audio Configs. This shows the clear separation between user interface, delegation layer, and actual system operations.

```mermaid
graph TD
    A[User] -->|Interacts| B[Orchestrator TUI]
    B -->|Install/Update| C[install_mediamtx.sh]
    B -->|Stream Control| D[mediamtx-stream-manager.sh]
    B -->|Device Mapping| E[usb-audio-mapper.sh]
    B -->|Capability Check| F[lyrebird-mic-check.sh]
    B -->|Health Checks| G[lyrebird-diagnostics.sh]
    B -->|Version Updates| H[lyrebird-updater.sh]

    C --> I[MediaMTX Binary]
    D --> J[FFmpeg Pipelines]
    E --> K[udev Rules]
    F --> L[Audio Configs]

    style B fill:#4051b5,color:#fff
    style C fill:#7c4dff,color:#fff
    style D fill:#7c4dff,color:#fff
    style E fill:#7c4dff,color:#fff
    style F fill:#7c4dff,color:#fff
    style G fill:#7c4dff,color:#fff
    style H fill:#7c4dff,color:#fff
```

### Component Integration

The Orchestrator integrates all LyreBirdAudio components:

| Component | Purpose | Integration |
|-----------|---------|-------------|
| **MediaMTX Installer** | Server management | Direct invocation with status display |
| **Stream Manager** | Process lifecycle | Status queries and control commands |
| **USB Mapper** | Device persistence | Interactive and automated mapping |
| **Capability Checker** | Hardware detection | Auto-configuration generation |
| **Diagnostics** | System health | Scheduled checks with reporting |
| **Version Manager** | Script updates | Update coordination and rollback |

---

## Exit Codes

The Orchestrator uses standard exit codes for automation:

| Code | Meaning | Details |
|------|---------|---------|
| `0` | Success | Operation completed successfully |
| `1` | General Error | Operation failed (see error message) |
| `2` | Permission Denied | Requires root/sudo privileges |
| `3` | Missing Dependencies | Required utilities not installed |
| `4` | Script Not Found | Delegated script missing or not executable |

**Usage in Scripts:**
```bash
sudo ./lyrebird-orchestrator.sh
if [ $? -eq 0 ]; then
    echo "Success"
else
    echo "Failed with exit code $?"
fi
```

---

## Advanced Features

### Hardware Capability Integration

The Orchestrator integrates real-time hardware capability detection:

- Displays supported sample rates during configuration
- Warns about USB audio adapter chip limitations
- Recommends quality tiers based on hardware
- Validates configurations against actual capabilities

### SHA256 Integrity Verification

The Orchestrator automatically verifies script integrity:

- Integrity checking is **always enabled** when `sha256sum` is available
- No environment variable needed - verification is automatic
- Scripts are validated before execution
- Warnings displayed if checksums don't match

### Auto Log Rotation

The Orchestrator automatically manages its own logs:

- **Location:** `/var/log/lyrebird-orchestrator.log`
- **Rotation:** When exceeding 1MB
- **Retention:** Last 1 rotation kept (`.1` backup file)
- **Compression:** None (plain text backups)

---

## Typical Workflows

### First-Time Setup

```bash
# 1. Clone repository
git clone https://github.com/tomtom215/LyreBirdAudio.git
cd LyreBirdAudio

# 2. Make scripts executable
chmod +x *.sh

# 3. Run Orchestrator
sudo ./lyrebird-orchestrator.sh

# 4. Select Quick Setup Wizard
# Follow prompts for complete installation
```

### Adding a New USB Microphone

```bash
sudo ./lyrebird-orchestrator.sh
# 1. Main Menu -> 3. USB Device Management
# 2. Select: Map USB Devices
# 3. Follow prompts to map new device
# 4. Reboot system for udev rules to take effect
# 5. Main Menu -> 4. Audio Streaming Control
# 6. Select: Add New Stream
# 7. Choose newly mapped device
```

### Updating to Latest Version

```bash
sudo ./lyrebird-orchestrator.sh
# 1. Main Menu -> 6. Version Management
# 2. Select: Check for Updates
# 3. Review changes
# 4. Select: Upgrade to Latest
# 5. Confirm update
# System will update and reload Orchestrator
```

### Troubleshooting a Stream Issue

```bash
sudo ./lyrebird-orchestrator.sh
# 1. Main Menu -> 5. System Diagnostics
# 2. Select: Full Diagnostic
# 3. Review output for errors
# 4. Follow remediation steps
# 5. Main Menu -> 7. Logs & Status
# 6. Select: View Stream Manager Log
# 7. Check for specific error messages
```

---

## Best Practices

!!! tip "Use Quick Setup Wizard for Initial Installation"
    The Quick Setup Wizard automates the entire installation process and ensures all steps are completed in the correct order.

!!! tip "Run Diagnostics After Changes"
    After any configuration change, run a quick health check to verify everything is working correctly.

!!! tip "Enable Stream Manager Service for Production"
    For 24/7 operation, always install the Stream Manager as a systemd service through the Orchestrator menu.

!!! warning "Always Review Diagnostic Output"
    Diagnostic reports provide actionable remediation steps. Follow them to resolve issues before they impact production.

---

## Troubleshooting

### Orchestrator Won't Start

**Symptoms:** Script fails immediately or shows permission errors

**Solutions:**
```bash
# Check if running as root
sudo -v

# Verify script is executable
chmod +x lyrebird-orchestrator.sh

# Check for missing dependencies
bash --version  # Must be 4.0+
command -v git ffmpeg curl
```

### Menu Options Don't Work

**Symptoms:** Selecting menu options results in "script not found" errors

**Solutions:**
```bash
# Ensure all scripts are in same directory
ls -la *.sh

# Make all scripts executable
chmod +x *.sh

# Verify no permission issues
ls -l lyrebird-orchestrator.sh
```

### Log File Errors

**Symptoms:** "Cannot write to log file" errors

**Solutions:**
```bash
# Create log directory
sudo mkdir -p /var/log/lyrebird

# Fix permissions
sudo chmod 755 /var/log/lyrebird

# Check disk space
df -h /var/log
```

---

## Related Documentation

- [Stream Manager](stream-manager.md) - FFmpeg process lifecycle management
- [USB Audio Mapper](usb-audio-mapper.md) - Device persistence configuration
- [Capability Checker](capability-checker.md) - Hardware detection details
- [Diagnostics Tool](diagnostics.md) - System health monitoring
- [Version Manager](version-manager.md) - Git-based updates
- [MediaMTX Installer](installer.md) - Server installation

---

## See Also

- [Getting Started: Quick Start](../getting-started/quick-start.md)
- [User Guide: Stream Management](../user-guide/stream-management.md)
- [Advanced: Troubleshooting](../advanced/troubleshooting.md)
- [Reference: Command Reference](../reference/command-reference.md)
