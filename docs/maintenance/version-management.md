# Version Management

LyreBirdAudio includes a sophisticated version management system for safe updates and rollbacks.

---

## Overview

The `lyrebird-updater.sh` script provides git-based version management with the following capabilities:

- Check current version and status
- List available versions (tags and branches)
- Update to latest release or specific version
- Rollback to previous versions
- View version history and changes

---

## Version Management Tool

### Checking Current Version

```bash
./lyrebird-updater.sh --status
```

This displays:

- Current version/tag/branch
- Git repository status
- Available updates
- Last update timestamp

### Listing Available Versions

```bash
./lyrebird-updater.sh --list
```

Shows all available:

- Tagged releases (v1.0.0, v1.1.0, etc.)
- Available branches
- Commit information

### Interactive Update Manager

```bash
./lyrebird-updater.sh
```

Launches an interactive menu with options to:

1. Update to latest version
2. Switch to specific version/tag
3. View version history
4. Check for updates
5. Rollback to previous version

---

## Production vs. Development Versions

### Tagged Releases (Recommended for Production)

Tagged releases (e.g., v1.0.0, v1.1.0) are:

- Tested for at least 72 hours before release
- Tested on Intel N100 mini-PC with 5 USB microphones running Ubuntu
- Considered stable for production use
- Recommended for unattended 24/7 deployments

**Example:**
```bash
./lyrebird-updater.sh
# Select: Switch to Specific Version
# Choose: v1.1.0
```

### Main Branch (Latest Features)

The main branch contains:

- Latest features and improvements
- Generally stable but may include work-in-progress code
- Suitable for testing and development
- May have features not fully documented

**Best Practice:** Production deployments should pin to specific tagged releases.

---

## Update Workflow

### Standard Update Process

1. **Check for updates:**
   ```bash
   ./lyrebird-updater.sh --status
   ```

2. **Review available versions:**
   ```bash
   ./lyrebird-updater.sh --list
   ```

3. **Run interactive updater:**
   ```bash
   ./lyrebird-updater.sh
   ```

4. **Select update option** from menu

5. **Restart services** after update:
   ```bash
   sudo systemctl restart mediamtx-audio
   ```

### Automated Update Check

The updater can be integrated into cron jobs for periodic update notifications:

```bash
# Check for updates weekly
0 3 * * 0 /path/to/lyrebird-updater.sh --status | mail -s "LyreBird Update Status" admin@example.com
```

---

## Rollback Capability

If an update causes issues, you can rollback to a previous version:

### Rollback Steps

1. **Launch updater:**
   ```bash
   ./lyrebird-updater.sh
   ```

2. **Select "Switch to Specific Version"**

3. **Choose previous stable version** from the list

4. **Restart services:**
   ```bash
   sudo systemctl restart mediamtx-audio
   ```

### Immediate Rollback

If you know the previous version tag:

```bash
cd /path/to/LyreBirdAudio
git checkout v1.0.0  # Replace with your target version
sudo systemctl restart mediamtx-audio
```

---

## Version Testing

After updating or rolling back:

### Verify Installation

```bash
sudo ./lyrebird-diagnostics.sh quick
```

### Check Service Status

```bash
sudo systemctl status mediamtx-audio
```

### Validate Streams

```bash
sudo ./mediamtx-stream-manager.sh status
```

### Test RTSP Connectivity

```bash
ffmpeg -i rtsp://localhost:8554/device-name -t 5 -f null -
```

---

## Git-Based Management

LyreBirdAudio uses git for version control, enabling:

- **Precise version tracking** - Every change is tracked
- **Safe updates** - Can always revert to known-good state
- **Branch switching** - Test features without affecting production
- **Local modifications** - Track custom changes alongside updates

### Checking Git Status

```bash
cd /path/to/LyreBirdAudio
git status
git log --oneline -10
```

### Handling Local Modifications

If you have local modifications:

```bash
# Stash changes before updating
git stash save "My local changes"

# Update to new version
./lyrebird-updater.sh

# Reapply changes if needed
git stash pop
```

---

## Best Practices

### Production Environments

1. **Always use tagged releases** for production deployments
2. **Test updates in staging** environment first
3. **Document your current version** before updating
4. **Have rollback plan** ready
5. **Monitor services** after updates for 24-48 hours
6. **Backup configuration** before major version changes

### Development Environments

1. **Main branch acceptable** for testing new features
2. **Test thoroughly** before promoting to production
3. **Report issues** on GitHub with version information
4. **Contribute fixes** back to the project

### Version Pinning

For critical production systems, consider pinning to specific versions:

```bash
cd /path/to/LyreBirdAudio
git checkout v1.1.0
git branch production-pin
```

This creates a local branch that won't be affected by updates.

---

## Troubleshooting Updates

### Update Fails with Merge Conflicts

If local modifications conflict with updates:

```bash
# Backup your changes
cp -r /path/to/LyreBirdAudio /path/to/LyreBirdAudio.backup

# Reset to clean state
git reset --hard HEAD

# Try update again
./lyrebird-updater.sh
```

### Services Don't Restart After Update

```bash
# Force stop all services
sudo ./mediamtx-stream-manager.sh force-stop

# Restart MediaMTX
sudo systemctl restart mediamtx

# Start streams again
sudo ./mediamtx-stream-manager.sh start
```

### Version Mismatch Issues

If components show version mismatches:

```bash
# Verify all scripts are from same version
./lyrebird-orchestrator.sh
# Select: Version Management -> Show Component Versions
```

---

## Related Documentation

- [Backup & Restore](backup-restore.md) - Before major updates
- [Diagnostics & Monitoring](../advanced/diagnostics-monitoring.md) - Post-update verification
- [Uninstallation](uninstallation.md) - Clean removal if needed

---

## Next Steps

<div class="grid" markdown>

<div markdown>
### Backup & Restore
Configuration backup and recovery procedures

[Backup & Restore →](backup-restore.md)
</div>

<div markdown>
### Main Documentation
Complete documentation index

[Main Documentation →](../index.md)
</div>

</div>
