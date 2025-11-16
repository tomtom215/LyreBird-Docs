# Uninstallation

Complete removal procedures for LyreBirdAudio, including cleanup of all components and optional data preservation.

---

## Overview

This guide provides comprehensive procedures for completely removing LyreBirdAudio from your system. Whether you're decommissioning a server, troubleshooting a corrupted installation, or migrating to a different solution, these instructions ensure complete and clean removal.

**Components Removed:**

| Component | Location | Description |
|-----------|----------|-------------|
| MediaMTX Binary | `/opt/mediamtx/` | RTSP server executable |
| MediaMTX Configuration | `/etc/mediamtx/` | Server and device settings |
| Stream Manager | `/opt/LyreBirdAudio/` | Scripts and utilities |
| Udev Rules | `/etc/udev/rules.d/99-usb-soundcards.rules` | USB persistence |
| Systemd Services | `/etc/systemd/system/mediamtx*.service` | Auto-start services |
| Log Files | `/var/log/mediamtx*` | Application logs |
| Backup Data | `/var/backups/lyrebird/` | Configuration backups |

---

## Pre-Uninstallation Checklist

Before removing LyreBirdAudio, consider:

- [ ] **Backup Configuration:** Save current settings for potential future use
- [ ] **Document Setup:** Note device configurations and stream settings
- [ ] **Export Data:** Save any logs or recordings you need to keep
- [ ] **Notify Users:** Inform anyone relying on your streams
- [ ] **Alternative Solution:** Ensure replacement system is ready (if applicable)

### Create Final Backup

```bash
# Create final backup before uninstallation
sudo mkdir -p /var/backups/lyrebird/final-backup && \
sudo cp /etc/mediamtx/mediamtx.yml \
       /etc/mediamtx/audio-devices.conf \
       /etc/udev/rules.d/99-usb-soundcards.rules \
       /var/backups/lyrebird/final-backup/ 2>/dev/null

echo "Final backup saved to /var/backups/lyrebird/final-backup"
```

---

## Quick Uninstallation

### One-Command Removal

Complete removal with a single command:

```bash
sudo systemctl stop mediamtx-audio 2>/dev/null && \
sudo systemctl disable mediamtx-audio 2>/dev/null && \
sudo systemctl stop mediamtx 2>/dev/null && \
sudo systemctl disable mediamtx 2>/dev/null && \
sudo rm -f /etc/systemd/system/mediamtx*.service && \
sudo systemctl daemon-reload && \
sudo rm -f /etc/udev/rules.d/99-usb-soundcards.rules && \
sudo udevadm control --reload-rules && \
sudo rm -rf /opt/LyreBirdAudio && \
sudo rm -rf /opt/mediamtx && \
sudo rm -rf /etc/mediamtx && \
echo "LyreBirdAudio uninstalled successfully"
```

!!! warning "Permanent Removal"
    This command permanently removes all LyreBirdAudio components. Ensure you have backups if you may need to restore the configuration later.

---

## Step-by-Step Uninstallation

### Step 1: Stop All Services

Stop running services gracefully:

```bash
# Stop stream manager service
sudo systemctl stop mediamtx-audio

# Verify stopped
sudo systemctl status mediamtx-audio
```

```bash
# Stop MediaMTX server (if running separately)
sudo systemctl stop mediamtx 2>/dev/null || true

# Verify stopped
sudo systemctl status mediamtx 2>/dev/null || echo "MediaMTX service not found"
```

### Step 2: Disable Auto-Start Services

Prevent services from starting on boot:

```bash
# Disable stream manager
sudo systemctl disable mediamtx-audio

# Verify disabled
sudo systemctl is-enabled mediamtx-audio || echo "Service disabled"
```

```bash
# Disable MediaMTX (if configured)
sudo systemctl disable mediamtx 2>/dev/null || true

# Verify disabled
sudo systemctl is-enabled mediamtx 2>/dev/null || echo "Service disabled"
```

### Step 3: Remove Systemd Service Files

Delete service configuration files:

```bash
# Remove service files
sudo rm -f /etc/systemd/system/mediamtx-audio.service
sudo rm -f /etc/systemd/system/mediamtx.service

# Verify removal
ls /etc/systemd/system/mediamtx*.service 2>/dev/null || echo "Service files removed"
```

```bash
# Reload systemd to apply changes
sudo systemctl daemon-reload

# Reset failed units
sudo systemctl reset-failed
```

### Step 4: Remove USB Persistence Rules

Delete udev rules for USB device mapping:

```bash
# Remove udev rules
sudo rm -f /etc/udev/rules.d/99-usb-soundcards.rules

# Verify removal
ls /etc/udev/rules.d/99-usb-soundcards.rules 2>/dev/null || echo "Udev rules removed"
```

```bash
# Reload udev rules
sudo udevadm control --reload-rules

# Trigger udev to apply changes
sudo udevadm trigger
```

```bash
# Remove persistent device symlinks (if they exist)
sudo rm -f /dev/lyrebird-* 2>/dev/null || true

# Verify removal
ls -l /dev/lyrebird-* 2>/dev/null || echo "Device symlinks removed"
```

### Step 5: Remove MediaMTX Installation

Delete MediaMTX binary and configuration:

```bash
# Remove MediaMTX installation directory
sudo rm -rf /opt/mediamtx

# Verify removal
ls -ld /opt/mediamtx 2>/dev/null || echo "MediaMTX directory removed"
```

```bash
# Remove MediaMTX configuration directory
sudo rm -rf /etc/mediamtx

# Verify removal
ls -ld /etc/mediamtx 2>/dev/null || echo "MediaMTX configuration removed"
```

### Step 6: Remove LyreBirdAudio Scripts

Delete all LyreBirdAudio components:

```bash
# Remove LyreBirdAudio installation directory
sudo rm -rf /opt/LyreBirdAudio

# Verify removal
ls -ld /opt/LyreBirdAudio 2>/dev/null || echo "LyreBirdAudio directory removed"
```

### Step 7: Remove Log Files

Clean up application logs:

```bash
# Remove MediaMTX logs
sudo rm -f /var/log/mediamtx*.log
sudo rm -f /var/log/mediamtx*.out

# Remove LyreBird logs
sudo rm -f /var/log/lyrebird*.log

# Verify removal
ls /var/log/mediamtx* /var/log/lyrebird* 2>/dev/null || echo "Log files removed"
```

```bash
# Clear systemd journal entries (optional)
sudo journalctl --vacuum-time=1s --rotate

echo "Systemd journal cleaned"
```

### Step 8: Remove Backup Data (Optional)

Remove configuration backups:

```bash
# List backups before removal
echo "Existing backups:"
ls -lh /var/backups/lyrebird/ 2>/dev/null || echo "No backups found"
```

```bash
# Remove all backups
sudo rm -rf /var/backups/lyrebird

# Verify removal
ls -ld /var/backups/lyrebird 2>/dev/null || echo "Backup directory removed"
```

!!! note "Preserve Backups"
    If you want to keep backups for potential future use, skip this step or move backups to a different location before uninstalling.

### Step 9: Remove Cron Jobs (If Configured)

Remove any scheduled backup or maintenance jobs:

```bash
# Check for LyreBird cron jobs
sudo crontab -l | grep -i lyrebird || echo "No LyreBird cron jobs found"
```

```bash
# Edit crontab to remove LyreBird entries
sudo crontab -e

# Remove lines containing:
# - /opt/LyreBirdAudio/
# - lyrebird
# - mediamtx (if applicable)
```

### Step 10: Verify Complete Removal

Confirm all components are removed:

```bash
# Check for remaining files
echo "Checking for remaining LyreBird components..."

echo "MediaMTX binary:"
which mediamtx 2>/dev/null || echo "  Not found"

echo "MediaMTX directory:"
ls -ld /opt/mediamtx 2>/dev/null || echo "  Not found"

echo "LyreBird directory:"
ls -ld /opt/LyreBirdAudio 2>/dev/null || echo "  Not found"

echo "Configuration directory:"
ls -ld /etc/mediamtx 2>/dev/null || echo "  Not found"

echo "Systemd services:"
ls /etc/systemd/system/mediamtx*.service 2>/dev/null || echo "  Not found"

echo "Udev rules:"
ls /etc/udev/rules.d/99-usb-soundcards.rules 2>/dev/null || echo "  Not found"

echo "Device symlinks:"
ls -l /dev/lyrebird-* 2>/dev/null || echo "  Not found"
```

```bash
# Check for running processes
echo "Checking for running MediaMTX processes..."
ps aux | grep -i mediamtx | grep -v grep || echo "  No processes found"
```

!!! success "Uninstallation Complete"
    LyreBirdAudio has been completely removed from your system.

---

## Selective Removal

### Remove Only Stream Manager

Keep MediaMTX but remove LyreBird stream management:

```bash
# Stop and disable stream manager
sudo systemctl stop mediamtx-audio
sudo systemctl disable mediamtx-audio
sudo rm -f /etc/systemd/system/mediamtx-audio.service
sudo systemctl daemon-reload

# Remove LyreBird scripts
sudo rm -rf /opt/LyreBirdAudio

# Keep MediaMTX and configuration
echo "MediaMTX retained, stream manager removed"
```

### Remove Only MediaMTX

Keep LyreBird scripts but remove MediaMTX server:

```bash
# Stop and disable MediaMTX
sudo systemctl stop mediamtx 2>/dev/null || true
sudo systemctl disable mediamtx 2>/dev/null || true
sudo rm -f /etc/systemd/system/mediamtx.service
sudo systemctl daemon-reload

# Remove MediaMTX binary
sudo rm -rf /opt/mediamtx

# Keep LyreBird scripts and configuration
echo "MediaMTX removed, LyreBird scripts retained"
```

### Remove Only USB Persistence

Remove USB device mapping but keep services:

```bash
# Remove udev rules
sudo rm -f /etc/udev/rules.d/99-usb-soundcards.rules
sudo udevadm control --reload-rules
sudo rm -f /dev/lyrebird-* 2>/dev/null || true

echo "USB persistence removed, services retained"
```

---

## Data Preservation

### Archive Configuration Before Removal

```bash
# Create archival backup
ARCHIVE_DIR="/root/lyrebird-archive-$(date +%Y%m%d)"
sudo mkdir -p $ARCHIVE_DIR

# Copy all configuration
sudo cp -r /etc/mediamtx $ARCHIVE_DIR/
sudo cp -r /opt/LyreBirdAudio $ARCHIVE_DIR/scripts 2>/dev/null || true
sudo cp /etc/systemd/system/mediamtx*.service $ARCHIVE_DIR/ 2>/dev/null || true
sudo cp /etc/udev/rules.d/99-usb-soundcards.rules $ARCHIVE_DIR/ 2>/dev/null || true

# Create archive manifest
sudo bash -c "cat > $ARCHIVE_DIR/README.txt << EOF
LyreBirdAudio Archive
Created: $(date)
Hostname: $(hostname)
User: $(whoami)

This archive contains all LyreBirdAudio configuration
files before uninstallation. To restore:

1. Copy configuration files to original locations
2. Reinstall LyreBirdAudio from GitHub
3. Reload services: systemctl daemon-reload
4. Start services: systemctl start mediamtx-audio

For support: https://github.com/tomtom215/LyreBirdAudio
EOF"

# Compress archive
cd $(dirname $ARCHIVE_DIR)
sudo tar czf lyrebird-archive-$(date +%Y%m%d).tar.gz $(basename $ARCHIVE_DIR)

echo "Archive created: lyrebird-archive-$(date +%Y%m%d).tar.gz"
```

### Export Logs Before Removal

```bash
# Export logs for analysis
sudo journalctl -u mediamtx-audio > /tmp/lyrebird-logs-$(date +%Y%m%d).txt
sudo journalctl -u mediamtx >> /tmp/lyrebird-logs-$(date +%Y%m%d).txt 2>/dev/null || true

echo "Logs exported to: /tmp/lyrebird-logs-$(date +%Y%m%d).txt"
```

---

## Reinstallation

### Restore from Archive

If you need to reinstall after uninstallation:

```bash
# Reinstall LyreBirdAudio
cd /opt
sudo git clone https://github.com/tomtom215/LyreBirdAudio.git

# Extract archived configuration
cd /root
sudo tar xzf lyrebird-archive-*.tar.gz

# Restore configuration
ARCHIVE_DIR=$(ls -1d /root/lyrebird-archive-* | tail -1)
sudo cp -r $ARCHIVE_DIR/mediamtx /etc/
sudo cp $ARCHIVE_DIR/mediamtx*.service /etc/systemd/system/
sudo cp $ARCHIVE_DIR/99-usb-soundcards.rules /etc/udev/rules.d/

# Reinstall MediaMTX
sudo /opt/LyreBirdAudio/lyrebird-orchestrator.sh
# Select option 1: Install/Update MediaMTX

# Apply configuration
sudo udevadm control --reload-rules
sudo systemctl daemon-reload
sudo systemctl start mediamtx-audio

echo "LyreBirdAudio restored from archive"
```

---

## Post-Uninstallation Cleanup

### Remove Dependencies (Optional)

If you no longer need audio streaming tools:

```bash
# Remove FFmpeg (if not used by other applications)
sudo apt-get remove --purge ffmpeg

# Remove ALSA utilities (if not needed)
sudo apt-get remove --purge alsa-utils

# Clean up unused packages
sudo apt-get autoremove
```

!!! warning "System Dependencies"
    Only remove dependencies if you're certain they're not needed by other applications on your system.

### Clean Package Cache

```bash
# Clean apt cache
sudo apt-get clean

# Remove old package lists
sudo apt-get autoclean
```

### Verify USB Devices Return to Default

```bash
# Check USB audio devices
arecord -l

# Verify no persistent device names
ls -l /dev/lyrebird-* 2>/dev/null || echo "No LyreBird device links"

# Check ALSA card numbering (should be default)
cat /proc/asound/cards
```

---

## Troubleshooting Uninstallation

### Issue: Services Won't Stop

**Force stop services:**

```bash
# Get process IDs
ps aux | grep mediamtx

# Kill processes
sudo pkill -f mediamtx

# Force kill if needed
sudo pkill -9 -f mediamtx

# Verify stopped
ps aux | grep mediamtx | grep -v grep || echo "All processes stopped"
```

### Issue: Permission Denied

**Ensure running with sudo:**

```bash
# All removal commands require root privileges
sudo -i

# Then run uninstallation commands
```

### Issue: Directory Not Empty

**Force remove directories:**

```bash
# Remove directories recursively with force
sudo rm -rf /opt/LyreBirdAudio
sudo rm -rf /opt/mediamtx
sudo rm -rf /etc/mediamtx

# Verify removal
ls -ld /opt/LyreBirdAudio /opt/mediamtx /etc/mediamtx 2>/dev/null || echo "Removed"
```

### Issue: Files Still Exist

**Search for remaining files:**

```bash
# Find all LyreBird-related files
sudo find / -name "*lyrebird*" -o -name "*mediamtx*" 2>/dev/null

# Remove manually if needed
sudo rm -f <file_path>
```

### Issue: Service Fails to Disable

**Mask the service:**

```bash
# Mask service to prevent activation
sudo systemctl mask mediamtx-audio
sudo systemctl mask mediamtx

# Remove service files
sudo rm -f /etc/systemd/system/mediamtx*.service
sudo systemctl daemon-reload

# Unmask (service is now gone)
sudo systemctl unmask mediamtx-audio 2>/dev/null || true
sudo systemctl unmask mediamtx 2>/dev/null || true
```

---

## Verification Checklist

After uninstallation, verify complete removal:

- [ ] No MediaMTX processes running: `ps aux | grep mediamtx`
- [ ] No systemd services: `systemctl list-units | grep mediamtx`
- [ ] No service files: `ls /etc/systemd/system/mediamtx*.service`
- [ ] No installation directories: `ls -ld /opt/LyreBirdAudio /opt/mediamtx /etc/mediamtx`
- [ ] No udev rules: `ls /etc/udev/rules.d/99-usb-soundcards.rules`
- [ ] No device symlinks: `ls -l /dev/lyrebird-*`
- [ ] No cron jobs: `sudo crontab -l | grep lyrebird`
- [ ] Backups preserved (if desired): `ls /var/backups/lyrebird/`

**Run verification script:**

```bash
#!/bin/bash
echo "LyreBirdAudio Uninstallation Verification"
echo "=========================================="

FOUND=0

if ps aux | grep -i mediamtx | grep -v grep > /dev/null; then
    echo "WARNING: MediaMTX processes still running"
    FOUND=1
fi

if systemctl list-units | grep mediamtx > /dev/null; then
    echo "WARNING: MediaMTX services still active"
    FOUND=1
fi

if [ -d /opt/LyreBirdAudio ] || [ -d /opt/mediamtx ] || [ -d /etc/mediamtx ]; then
    echo "WARNING: Installation directories still exist"
    FOUND=1
fi

if [ -f /etc/udev/rules.d/99-usb-soundcards.rules ]; then
    echo "WARNING: Udev rules still exist"
    FOUND=1
fi

if [ $FOUND -eq 0 ]; then
    echo "SUCCESS: LyreBirdAudio completely removed"
else
    echo "INCOMPLETE: Some components remain"
fi
```

---

## Related Documentation

- [Backup and Restore](backup-restore.md) - Preserve configuration before removal
- [Installation](../getting-started/installation.md) - Reinstallation procedures
- [Configuration Files](../reference/configuration-files.md) - File locations reference
- [Troubleshooting](../advanced/troubleshooting.md) - Problem resolution

---

## Next Steps

<div class="grid" markdown>

<div markdown>
### Backup and Restore
Configuration backup and recovery procedures

[Backup and Restore →](backup-restore.md)
</div>

<div markdown>
### Main Documentation
Complete documentation index

[Main Documentation →](../index.md)
</div>

</div>
