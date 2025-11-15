# Backup and Restore

Complete backup and restore procedures for LyreBirdAudio configurations, ensuring safe recovery and migration.

---

## Overview

Regular backups protect your LyreBirdAudio configuration from accidental changes, system failures, or hardware issues. This guide covers manual and automated backup strategies, restoration procedures, and best practices for production environments.

**What to Backup:**

| Component | Files | Purpose |
|-----------|-------|---------|
| MediaMTX Configuration | `/etc/mediamtx/mediamtx.yml` | Server settings |
| Audio Device Settings | `/etc/mediamtx/audio-devices.conf` | Device encoding configuration |
| USB Persistence Rules | `/etc/udev/rules.d/99-usb-soundcards.rules` | USB device naming |
| Systemd Services | `/etc/systemd/system/mediamtx*.service` | Service configuration |
| Stream Configurations | `/opt/LyreBirdAudio/streams/*.env` | Individual stream settings |

---

## Quick Backup

### One-Command Backup

Create a complete backup with a single command:

```bash
sudo mkdir -p /var/backups/lyrebird/$(date +%Y%m%d-%H%M%S) && \
sudo cp /etc/mediamtx/mediamtx.yml \
       /etc/mediamtx/audio-devices.conf \
       /etc/udev/rules.d/99-usb-soundcards.rules \
       /var/backups/lyrebird/$(date +%Y%m%d-%H%M%S)/ && \
sudo cp /etc/systemd/system/mediamtx*.service \
       /var/backups/lyrebird/$(date +%Y%m%d-%H%M%S)/ 2>/dev/null || true && \
echo "Backup complete: /var/backups/lyrebird/$(date +%Y%m%d-%H%M%S)"
```

### Quick Restore

Restore from the most recent backup:

```bash
# Find latest backup
LATEST=$(ls -1d /var/backups/lyrebird/* | tail -1)

# Restore configuration files
sudo cp $LATEST/mediamtx.yml /etc/mediamtx/
sudo cp $LATEST/audio-devices.conf /etc/mediamtx/
sudo cp $LATEST/99-usb-soundcards.rules /etc/udev/rules.d/
sudo cp $LATEST/mediamtx*.service /etc/systemd/system/ 2>/dev/null || true

# Apply changes
sudo udevadm control --reload-rules
sudo systemctl daemon-reload
sudo systemctl restart mediamtx-stream-manager

echo "Restored from: $LATEST"
```

---

## Manual Backup

### Step-by-Step Backup

**1. Create Backup Directory**

```bash
# Create timestamped backup directory
BACKUP_DIR="/var/backups/lyrebird/$(date +%Y%m%d-%H%M%S)"
sudo mkdir -p $BACKUP_DIR

echo "Backup directory: $BACKUP_DIR"
```

**2. Backup MediaMTX Configuration**

```bash
# Copy MediaMTX server configuration
sudo cp /etc/mediamtx/mediamtx.yml $BACKUP_DIR/

# Verify
ls -lh $BACKUP_DIR/mediamtx.yml
```

**3. Backup Audio Device Settings**

```bash
# Copy audio encoding configuration
sudo cp /etc/mediamtx/audio-devices.conf $BACKUP_DIR/

# Verify
ls -lh $BACKUP_DIR/audio-devices.conf
```

**4. Backup USB Persistence Rules**

```bash
# Copy udev rules
sudo cp /etc/udev/rules.d/99-usb-soundcards.rules $BACKUP_DIR/

# Verify
ls -lh $BACKUP_DIR/99-usb-soundcards.rules
```

**5. Backup Systemd Services**

```bash
# Copy all MediaMTX-related service files
sudo cp /etc/systemd/system/mediamtx.service $BACKUP_DIR/ 2>/dev/null || true
sudo cp /etc/systemd/system/mediamtx-audio.service $BACKUP_DIR/ 2>/dev/null || true

# Verify
ls -lh $BACKUP_DIR/*.service
```

**6. Backup Stream Configurations (Optional)**

```bash
# Copy stream-specific settings if they exist
if [ -d /opt/LyreBirdAudio/streams ]; then
    sudo cp -r /opt/LyreBirdAudio/streams $BACKUP_DIR/
    echo "Stream configurations backed up"
fi
```

**7. Create Backup Manifest**

```bash
# Document backup contents
sudo bash -c "cat > $BACKUP_DIR/MANIFEST.txt << EOF
LyreBirdAudio Backup
Created: $(date)
Hostname: $(hostname)
User: $(whoami)

Files:
$(ls -lh $BACKUP_DIR)

System Info:
$(uname -a)

MediaMTX Version:
$(mediamtx --version 2>/dev/null || echo 'Not found')
EOF"

# View manifest
cat $BACKUP_DIR/MANIFEST.txt
```

!!! success "Backup Complete"
    Your configuration is backed up to: `$BACKUP_DIR`

---

## Automated Backup

!!! info "Backup Script - TBD"
    A dedicated `backup-config.sh` script is planned for future implementation. For now, you can create your own backup script using the example below.

### Backup Script

Create a reusable backup script:

```bash
# Create script directory
sudo mkdir -p /opt/LyreBirdAudio/scripts

# Create backup script
sudo tee /opt/LyreBirdAudio/scripts/backup-config.sh << 'EOF'
#!/bin/bash
#
# LyreBirdAudio Configuration Backup Script
#

set -e

# Configuration
BACKUP_ROOT="/var/backups/lyrebird"
RETENTION_DAYS=30

# Create timestamped backup directory
BACKUP_DIR="$BACKUP_ROOT/$(date +%Y%m%d-%H%M%S)"
mkdir -p "$BACKUP_DIR"

echo "Starting LyreBirdAudio backup..."
echo "Backup directory: $BACKUP_DIR"

# Backup configuration files
echo "Backing up configuration files..."
cp /etc/mediamtx/mediamtx.yml "$BACKUP_DIR/" 2>/dev/null || echo "Warning: mediamtx.yml not found"
cp /etc/mediamtx/audio-devices.conf "$BACKUP_DIR/" 2>/dev/null || echo "Warning: audio-devices.conf not found"
cp /etc/udev/rules.d/99-usb-soundcards.rules "$BACKUP_DIR/" 2>/dev/null || echo "Warning: USB rules not found"

# Backup systemd services
echo "Backing up systemd services..."
cp /etc/systemd/system/mediamtx.service "$BACKUP_DIR/" 2>/dev/null || true
cp /etc/systemd/system/mediamtx-audio.service "$BACKUP_DIR/" 2>/dev/null || true

# Backup stream configurations
if [ -d /opt/LyreBirdAudio/streams ]; then
    echo "Backing up stream configurations..."
    cp -r /opt/LyreBirdAudio/streams "$BACKUP_DIR/"
fi

# Create manifest
cat > "$BACKUP_DIR/MANIFEST.txt" << MANIFEST
LyreBirdAudio Backup
Created: $(date)
Hostname: $(hostname)
User: $(whoami)

Files:
$(ls -lh "$BACKUP_DIR")

System Info:
$(uname -a)
MANIFEST

# Clean old backups
echo "Cleaning backups older than $RETENTION_DAYS days..."
find "$BACKUP_ROOT" -type d -mtime +$RETENTION_DAYS -exec rm -rf {} + 2>/dev/null || true

# Create latest symlink
ln -sfn "$BACKUP_DIR" "$BACKUP_ROOT/latest"

echo "Backup complete: $BACKUP_DIR"
echo "Backup size: $(du -sh $BACKUP_DIR | cut -f1)"

exit 0
EOF

# Make executable
sudo chmod +x /opt/LyreBirdAudio/scripts/backup-config.sh
```

### Run Backup Script

```bash
sudo /opt/LyreBirdAudio/scripts/backup-config.sh
```

### Scheduled Backups with Cron

**Daily Backup at 2 AM:**

```bash
# Edit root crontab
sudo crontab -e
```

Add:
```
# LyreBirdAudio daily backup at 2 AM
0 2 * * * /opt/LyreBirdAudio/scripts/backup-config.sh >> /var/log/lyrebird-backup.log 2>&1
```

**Weekly Backup (Sunday at 3 AM):**

```
# LyreBirdAudio weekly backup
0 3 * * 0 /opt/LyreBirdAudio/scripts/backup-config.sh >> /var/log/lyrebird-backup.log 2>&1
```

**Before System Changes:**

```bash
# Run backup before making changes
sudo /opt/LyreBirdAudio/scripts/backup-config.sh

# Make your changes...
sudo vim /etc/mediamtx/audio-devices.conf
```

---

## Git-Based Backup

### Initialize Git Repository

```bash
# Create configuration repository
sudo mkdir -p /etc/mediamtx/config-repo
cd /etc/mediamtx/config-repo

# Initialize git
sudo git init

# Copy configurations
sudo cp /etc/mediamtx/mediamtx.yml .
sudo cp /etc/mediamtx/audio-devices.conf .
sudo cp /etc/udev/rules.d/99-usb-soundcards.rules .

# Create .gitignore
sudo tee .gitignore << 'EOF'
*.log
*.tmp
.DS_Store
EOF

# Initial commit
sudo git add .
sudo git commit -m "Initial LyreBirdAudio configuration"

echo "Git repository initialized"
```

### Track Changes with Git

```bash
# Make configuration changes
sudo vim /etc/mediamtx/audio-devices.conf

# Update repository
cd /etc/mediamtx/config-repo
sudo cp /etc/mediamtx/audio-devices.conf .
sudo git diff audio-devices.conf

# Commit changes
sudo git add audio-devices.conf
sudo git commit -m "Update Blue Yeti bitrate to 192k"

# View history
sudo git log --oneline
```

### Remote Backup with Git

```bash
# Add remote repository
sudo git remote add origin git@github.com:yourorg/lyrebird-config.git

# Push to remote
sudo git push -u origin main

# Automated push after changes
cd /etc/mediamtx/config-repo
sudo cp /etc/mediamtx/*.conf .
sudo cp /etc/mediamtx/*.yml .
sudo git add .
sudo git commit -m "Automated backup $(date)"
sudo git push
```

!!! warning "Security Consideration"
    Never commit sensitive data (passwords, API keys) to git repositories. Use environment variables or encrypted secrets.

---

## Restoration Procedures

### Full System Restore

**1. Stop Services**

```bash
# Stop all LyreBirdAudio services
sudo systemctl stop mediamtx-audio
sudo systemctl stop mediamtx 2>/dev/null || true
```

**2. List Available Backups**

```bash
# Show all backups
ls -lh /var/backups/lyrebird/

# Show latest backup
ls -lh /var/backups/lyrebird/latest
```

**3. Select Backup to Restore**

```bash
# Set backup directory
RESTORE_DIR="/var/backups/lyrebird/20250115-140530"

# Or use latest
RESTORE_DIR="/var/backups/lyrebird/latest"

echo "Restoring from: $RESTORE_DIR"
```

**4. Restore Configuration Files**

```bash
# Restore MediaMTX configuration
sudo cp $RESTORE_DIR/mediamtx.yml /etc/mediamtx/

# Restore audio device settings
sudo cp $RESTORE_DIR/audio-devices.conf /etc/mediamtx/

# Restore USB rules
sudo cp $RESTORE_DIR/99-usb-soundcards.rules /etc/udev/rules.d/

# Verify
ls -lh /etc/mediamtx/mediamtx.yml
ls -lh /etc/mediamtx/audio-devices.conf
ls -lh /etc/udev/rules.d/99-usb-soundcards.rules
```

**5. Restore Systemd Services**

```bash
# Restore service files
sudo cp $RESTORE_DIR/mediamtx.service /etc/systemd/system/ 2>/dev/null || true
sudo cp $RESTORE_DIR/mediamtx-audio.service /etc/systemd/system/ 2>/dev/null || true

# Verify
ls -lh /etc/systemd/system/mediamtx*.service
```

**6. Restore Stream Configurations (if applicable)**

```bash
# Restore stream settings
if [ -d $RESTORE_DIR/streams ]; then
    sudo cp -r $RESTORE_DIR/streams /opt/LyreBirdAudio/
    echo "Stream configurations restored"
fi
```

**7. Apply Changes**

```bash
# Reload udev rules
sudo udevadm control --reload-rules
sudo udevadm trigger

# Reload systemd
sudo systemctl daemon-reload

# Verify configuration syntax
sudo /opt/LyreBirdAudio/lyrebird-mic-check.sh -V
```

**8. Restart Services**

```bash
# Start MediaMTX
sudo systemctl start mediamtx 2>/dev/null || true

# Start stream manager
sudo systemctl start mediamtx-audio

# Verify status
sudo systemctl status mediamtx-audio
```

**9. Verify Restoration**

```bash
# Check service status
sudo systemctl status mediamtx-audio

# Check streams
sudo /opt/LyreBirdAudio/mediamtx-stream-manager.sh status

# Test stream playback
ffmpeg -i rtsp://localhost:8554/your-stream -t 5 -acodec copy test.aac
```

!!! success "Restoration Complete"
    Your LyreBirdAudio configuration has been restored successfully.

---

### Selective Restore

**Restore Only MediaMTX Configuration:**

```bash
RESTORE_DIR="/var/backups/lyrebird/latest"

sudo cp $RESTORE_DIR/mediamtx.yml /etc/mediamtx/
sudo systemctl restart mediamtx-audio
```

**Restore Only Audio Device Settings:**

```bash
RESTORE_DIR="/var/backups/lyrebird/latest"

sudo cp $RESTORE_DIR/audio-devices.conf /etc/mediamtx/
sudo systemctl restart mediamtx-audio
```

**Restore Only USB Rules:**

```bash
RESTORE_DIR="/var/backups/lyrebird/latest"

sudo cp $RESTORE_DIR/99-usb-soundcards.rules /etc/udev/rules.d/
sudo udevadm control --reload-rules
sudo udevadm trigger
```

---

## Migration to New System

### Export Configuration

**On Source System:**

```bash
# Create timestamped backup
BACKUP_DIR="/var/backups/lyrebird/$(date +%Y%m%d-%H%M%S)"
sudo mkdir -p $BACKUP_DIR
sudo cp /etc/mediamtx/mediamtx.yml \
       /etc/mediamtx/audio-devices.conf \
       /etc/udev/rules.d/99-usb-soundcards.rules \
       $BACKUP_DIR/

# Compress backup
cd /var/backups/lyrebird
sudo tar czf lyrebird-migration-$(date +%Y%m%d).tar.gz latest/

# Transfer to new system
scp lyrebird-migration-*.tar.gz user@new-server:/tmp/
```

### Import Configuration

**On Destination System:**

```bash
# Extract migration package
cd /tmp
tar xzf lyrebird-migration-*.tar.gz

# Install LyreBirdAudio first
cd /opt
sudo git clone https://github.com/tomtom215/LyreBirdAudio.git

# Restore configuration
sudo cp /tmp/latest/mediamtx.yml /etc/mediamtx/
sudo cp /tmp/latest/audio-devices.conf /etc/mediamtx/
sudo cp /tmp/latest/99-usb-soundcards.rules /etc/udev/rules.d/

# Apply and verify
sudo udevadm control --reload-rules
sudo systemctl daemon-reload
sudo systemctl restart mediamtx-audio
```

!!! note "USB Device IDs"
    USB device IDs may differ on the new system. Review and update `/etc/udev/rules.d/99-usb-soundcards.rules` with correct vendor/product IDs using `lsusb`.

---

## Best Practices

### Production Environment

**1. Regular Scheduled Backups**

- Daily automated backups
- Weekly full system snapshots
- Monthly archival backups

**2. Off-Site Storage**

```bash
# Upload to remote storage
rsync -avz /var/backups/lyrebird/ backup-server:/backups/lyrebird/

# Or cloud storage
aws s3 sync /var/backups/lyrebird/ s3://your-bucket/lyrebird-backups/
```

**3. Backup Verification**

```bash
# Test restore monthly
RESTORE_DIR="/var/backups/lyrebird/latest"

# Verify backup integrity
tar tzf $RESTORE_DIR/../lyrebird-backup.tar.gz > /dev/null && echo "Backup OK"

# Verify file checksums
md5sum $RESTORE_DIR/* > $RESTORE_DIR/checksums.md5
md5sum -c $RESTORE_DIR/checksums.md5
```

**4. Pre-Change Backups**

```bash
# Always backup before changes
BACKUP_DIR="/var/backups/lyrebird/$(date +%Y%m%d-%H%M%S)"
sudo mkdir -p $BACKUP_DIR
sudo cp /etc/mediamtx/mediamtx.yml \
       /etc/mediamtx/audio-devices.conf \
       /etc/udev/rules.d/99-usb-soundcards.rules \
       $BACKUP_DIR/

# Document the change
echo "Reason: Upgrading MediaMTX to v1.9.0" | \
  sudo tee $BACKUP_DIR/CHANGE_REASON.txt
```

**5. Retention Policy**

```bash
# Keep:
# - Daily backups: 7 days
# - Weekly backups: 4 weeks
# - Monthly backups: 12 months

# Automated cleanup in backup script
find /var/backups/lyrebird -type d -name "2025*" -mtime +7 -delete  # Daily
find /var/backups/lyrebird -type d -name "2025*" -mtime +30 -delete # Weekly
```

### Configuration Management

**Document Changes:**

```bash
# Add comments to configurations
# /etc/mediamtx/audio-devices.conf

# Blue Yeti - Updated 2025-01-15 by admin
# Increased bitrate for better quality
DEVICE_Blue_Yeti_BITRATE=192k  # Was: 128k
```

**Use Version Tags:**

```bash
# Tag major configuration changes
cd /etc/mediamtx/config-repo
sudo git tag -a v1.0 -m "Production configuration v1.0"
sudo git push origin v1.0
```

---

## Troubleshooting

### Backup Issues

**Issue: Permission Denied**

```bash
# Ensure running backup commands with sudo
sudo mkdir -p /var/backups/lyrebird
sudo cp /etc/mediamtx/*.conf /var/backups/lyrebird/
```

**Issue: Disk Space Full**

```bash
# Check available space
df -h /var/backups

# Clean old backups
sudo find /var/backups/lyrebird -type d -mtime +30 -exec rm -rf {} +

# Compress backups
cd /var/backups/lyrebird
sudo tar czf archive-$(date +%Y%m).tar.gz 2025* && sudo rm -rf 2025*
```

### Restore Issues

**Issue: Service Won't Start After Restore**

```bash
# Check service status
sudo systemctl status mediamtx-audio

# Check logs
sudo journalctl -u mediamtx-audio -n 50

# Verify configuration syntax with lyrebird-mic-check
sudo /opt/LyreBirdAudio/lyrebird-mic-check.sh -V
```

**Issue: Streams Not Working After Restore**

```bash
# Verify USB devices
arecord -l

# Check device mapping
ls -l /dev/lyrebird-*

# Re-run device mapper
sudo /opt/LyreBirdAudio/usb-audio-mapper.sh
```

---

## Related Documentation

- [Uninstallation](uninstallation.md) - Complete system removal
- [Version Management](version-management.md) - Update and rollback procedures
- [Configuration Files](../reference/configuration-files.md) - File format reference
- [Troubleshooting](../advanced/troubleshooting.md) - Problem resolution

---

## Next Steps

<div class="grid" markdown>

<div markdown>
### Version Management
Safe updates, rollbacks, and version control

[Version Management →](version-management.md)
</div>

<div markdown>
### Main Documentation
Complete documentation index

[Main Documentation →](../index.md)
</div>

</div>
