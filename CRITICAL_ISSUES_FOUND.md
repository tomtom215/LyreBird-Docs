# Critical Issues Found During Documentation Review

**Review Date:** 2025-11-15
**Reviewer:** Claude (Comprehensive Source Code Verification)
**Scope:** Complete verification of LyreBird-Docs against LyreBirdAudio source repository

---

## Critical Path Inconsistencies

### Issue 1: Device Path Mismatch Between Source Scripts

**Severity:** CRITICAL
**Impact:** Users will have incorrect paths, symlinks won't exist where documented

**Problem:**
The `usb-audio-mapper.sh` script creates device symlinks at `/dev/sound/by-id/` but multiple places in documentation and even the source repository reference `/dev/snd/by-usb-port/`.

**Source Code Evidence:**
- `usb-audio-mapper.sh` line 609: `SYMLINK+="sound/by-id/$friendly_name"` creates `/dev/sound/by-id/`
- `lyrebird-orchestrator.sh` line 849: References `/dev/snd/by-usb-port/` (INCORRECT)
- `lyrebird-orchestrator.sh` line 1127: `ls -la /dev/snd/by-usb-port/` (WILL ALWAYS FAIL)
- `README.md` (source repo) lines 170-171, 1677: Shows `/dev/snd/by-usb-port/` (INCORRECT)

**CORRECT PATH:** `/dev/sound/by-id/` (based on actual udev rule generation)

**Documentation Files Requiring Correction:**
1. `docs/components/capability-checker.md` lines 76, 95, 205
2. `docs/components/diagnostics.md` line 239

**Note:** Most documentation already uses `/dev/sound/by-id/` correctly. Only these 4 references need correction.

**Upstream Issue:** The LyreBirdAudio source repository itself has this inconsistency. The orchestrator.sh and README.md need to be updated to match usb-audio-mapper.sh.

---

### Issue 2: Log File Path Inconsistencies

**Severity:** HIGH
**Impact:** Users cannot find logs where documentation says they are

**Problem:**
Documentation states Stream Manager log is at `/var/log/lyrebird/stream-manager.log` but the actual script uses `/var/log/mediamtx-stream-manager.log`.

**Source Code Evidence:**
- `mediamtx-stream-manager.sh` line 134: `readonly LOG_FILE="${MEDIAMTX_LOG_FILE:-/var/log/mediamtx-stream-manager.log}"`

**CORRECT PATH:** `/var/log/mediamtx-stream-manager.log`

**Documentation Files Requiring Verification:**
- All references to stream manager log paths need review

---

## Minor Documentation Enhancements Needed

### 1. capability-checker.md - Missing Exit Codes Table

**Severity:** LOW
**Impact:** Users may not know proper error handling

**Required Addition:**
Add exit codes table:
- 0: Success
- 1: General error (invalid arguments, no devices, etc.)
- 2: Configuration error (validation failed, write error, etc.)

**Source:** Script lines 347-349 in lyrebird-mic-check.sh

---

### 2. version-manager.md - Sudo Usage Clarification

**Severity:** LOW
**Impact:** Minor confusion about permissions

**Problem:**
Documentation lines 336-339 implies script uses `sudo` internally, but it doesn't. Script assumes user already has necessary permissions.

**Required Change:**
Clarify that user must have permissions before running script; script does not escalate privileges internally.

---

### 3. installer.md - Missing Backup Cleanup Policy

**Severity:** LOW
**Impact:** Users don't know automatic cleanup behavior

**Required Addition:**
Document that script automatically:
- Keeps last 3 backups
- Removes backups older than 7 days
- Source: install_mediamtx.sh lines 1336-1348

---

## Component Verification Summary

All six component documentation files were verified against source scripts:

1. ✅ **capability-checker.md** ↔ lyrebird-mic-check.sh v1.0.0 - ACCURATE (needs exit codes table)
2. ✅ **diagnostics.md** ↔ lyrebird-diagnostics.sh v1.0.2 - EXCELLENT
3. ✅ **version-manager.md** ↔ lyrebird-updater.sh v1.5.1 - ACCURATE (needs sudo clarification)
4. ✅ **installer.md** ↔ install_mediamtx.sh v2.0.1 - ACCURATE (needs backup policy)
5. ✅ **stream-manager.md** ↔ mediamtx-stream-manager.sh v1.4.1 - EXCELLENT
6. ✅ **usb-audio-mapper.md** ↔ usb-audio-mapper.sh v1.2.1 - EXCELLENT
7. ✅ **orchestrator.md** ↔ lyrebird-orchestrator.sh v2.1.0 - EXCELLENT

**Overall Documentation Quality:** EXCELLENT
**Critical Issues:** 2 (both related to path inconsistencies)
**Minor Enhancements:** 3

---

## Recommended Actions

### Immediate (This PR):
1. Fix all `/dev/snd/by-usb-port/` → `/dev/sound/by-id/` in documentation (4 references)
2. Verify and correct all log file paths
3. Add exit codes table to capability-checker.md
4. Clarify sudo usage in version-manager.md
5. Document backup cleanup policy in installer.md

### Upstream (LyreBirdAudio Repository):
1. Fix orchestrator.sh lines 849, 1127 to use `/dev/sound/by-id/`
2. Update README.md to show correct path `/dev/sound/by-id/`
3. Ensure all scripts reference consistent device paths

---

## Files Requiring Changes in This Review

1. `docs/components/capability-checker.md`
2. `docs/components/diagnostics.md`
3. `docs/components/version-manager.md`
4. `docs/components/installer.md`
5. `docs/components/orchestrator.md` (verify log paths)
6. All files referencing stream manager log location
