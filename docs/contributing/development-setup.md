# Development Setup

Complete development environment setup guide for LyreBirdAudio contributors.

---

## Overview

LyreBirdAudio is built entirely with Bash scripts (version 4.0+), making it straightforward to develop and test. This guide covers setting up your development environment, working with the codebase, and preparing contributions for submission.

**Development Prerequisites:**

| Requirement | Version | Purpose |
|-------------|---------|---------|
| Bash | 4.0+ | Associative array support |
| Git | 2.0+ | Version control |
| Linux | Kernel 4.9+ | Core platform |
| USB Audio Device | Any | Hardware testing |
| Text Editor | Any | Code editing |

---

## Quick Start

### Fork and Clone Repository

**1. Fork the Repository**

Visit [https://github.com/tomtom215/LyreBirdAudio](https://github.com/tomtom215/LyreBirdAudio) and click **Fork** to create your own copy.

**2. Clone Your Fork**

```bash
# Clone your fork
git clone https://github.com/YOUR-USERNAME/LyreBirdAudio.git
cd LyreBirdAudio

# Add upstream remote
git remote add upstream https://github.com/tomtom215/LyreBirdAudio.git

# Verify remotes
git remote -v
```

Expected output:
```
origin    https://github.com/YOUR-USERNAME/LyreBirdAudio.git (fetch)
origin    https://github.com/YOUR-USERNAME/LyreBirdAudio.git (push)
upstream  https://github.com/tomtom215/LyreBirdAudio.git (fetch)
upstream  https://github.com/tomtom215/LyreBirdAudio.git (push)
```

**3. Create Development Branch**

```bash
# Create feature branch
git checkout -b feature/your-feature-name

# Or bug fix branch
git checkout -b fix/your-bug-fix
```

!!! tip "Branch Naming Convention"
    - Features: `feature/descriptive-name`
    - Bug fixes: `fix/issue-description`
    - Documentation: `docs/what-changed`
    - Performance: `perf/optimization-name`

---

## Development Environment Setup

### Install System Dependencies

=== "Ubuntu/Debian"

    ```bash
    # Core dependencies
    sudo apt-get update
    sudo apt-get install -y \
        git \
        bash \
        alsa-utils \
        udev \
        systemd \
        curl \
        wget \
        ffmpeg

    # Development tools
    sudo apt-get install -y \
        shellcheck \
        jq \
        vim \
        tree
    ```

=== "Arch Linux"

    ```bash
    # Core dependencies
    sudo pacman -Sy
    sudo pacman -S \
        git \
        bash \
        alsa-utils \
        udev \
        systemd \
        curl \
        wget \
        ffmpeg

    # Development tools
    sudo pacman -S \
        shellcheck \
        jq \
        vim \
        tree
    ```

=== "Fedora/RHEL"

    ```bash
    # Core dependencies
    sudo dnf install -y \
        git \
        bash \
        alsa-utils \
        systemd \
        curl \
        wget \
        ffmpeg

    # Development tools
    sudo dnf install -y \
        ShellCheck \
        jq \
        vim \
        tree
    ```

### Verify Installation

```bash
# Check Bash version (must be 4.0+)
bash --version

# Check FFmpeg
ffmpeg -version

# Check shellcheck (for code validation)
shellcheck --version

# List USB audio devices
arecord -l
```

---

## Repository Structure

Understanding the codebase organization:

```
LyreBirdAudio/
├── lyrebird-orchestrator.sh          # Main TUI management interface
├── mediamtx-stream-manager.sh        # Stream lifecycle manager
├── lyrebird-usb-mapper.sh            # USB device mapping
├── lyrebird-capability-checker.sh    # Audio capability detection
├── lyrebird-diagnostics.sh           # System diagnostics
├── lyrebird-version-manager.sh       # Version management
├── install-mediamtx.sh               # MediaMTX installer
├── README.md                         # Project documentation
├── LICENSE                           # Apache 2.0 license
└── .git/                             # Git repository
```

**Script Responsibilities:**

| Script | Purpose |
|--------|---------|
| `lyrebird-orchestrator.sh` | Unified management TUI, delegates to other scripts |
| `mediamtx-stream-manager.sh` | FFmpeg process management, health monitoring |
| `lyrebird-usb-mapper.sh` | USB persistence, udev rule generation |
| `lyrebird-capability-checker.sh` | Hardware detection, optimal settings |
| `lyrebird-diagnostics.sh` | System health checks, troubleshooting |
| `lyrebird-version-manager.sh` | Git-based version control, rollbacks |
| `install-mediamtx.sh` | MediaMTX binary installation |

---

## Local Development Workflow

### Setting Up Test Environment

**1. Create Development Installation**

```bash
# Use a separate directory for development
sudo mkdir -p /opt/LyreBirdAudio-dev
sudo chown $USER:$USER /opt/LyreBirdAudio-dev

# Copy your working directory
cp -r ~/LyreBirdAudio/* /opt/LyreBirdAudio-dev/

# Or create symlink
ln -s ~/LyreBirdAudio /opt/LyreBirdAudio-dev
```

**2. Install MediaMTX for Testing**

```bash
cd /opt/LyreBirdAudio-dev
sudo ./install-mediamtx.sh
```

**3. Configure Test Devices**

```bash
# Map USB devices for testing
sudo ./lyrebird-usb-mapper.sh

# Verify mapping
ls -l /dev/lyrebird-*
```

### Making Changes

**1. Edit Scripts**

```bash
# Use your preferred editor
vim lyrebird-orchestrator.sh

# Or
nano mediamtx-stream-manager.sh

# Or
code .  # VS Code
```

**2. Validate Syntax**

```bash
# Check script syntax with shellcheck
shellcheck lyrebird-orchestrator.sh

# Fix all warnings before committing
shellcheck *.sh
```

**3. Test Changes Locally**

```bash
# Test specific script
sudo ./lyrebird-orchestrator.sh

# Test stream manager
sudo ./mediamtx-stream-manager.sh status

# Run diagnostics
sudo ./lyrebird-diagnostics.sh quick
```

**4. Review Changes**

```bash
# View your modifications
git diff

# Check status
git status

# View specific file changes
git diff lyrebird-orchestrator.sh
```

---

## Testing Your Changes

### Unit Testing Individual Scripts

**Test USB Mapper:**

```bash
# Dry run (no changes)
sudo ./lyrebird-usb-mapper.sh --dry-run

# Full execution
sudo ./lyrebird-usb-mapper.sh

# Verify udev rules
cat /etc/udev/rules.d/99-usb-soundcards.rules
```

**Test Capability Checker:**

```bash
# Check specific device
sudo ./lyrebird-capability-checker.sh /dev/lyrebird-mic-1

# Validate configuration
sudo ./lyrebird-capability-checker.sh --validate-config
```

**Test Stream Manager:**

```bash
# Start streams
sudo ./mediamtx-stream-manager.sh start

# Check status
sudo ./mediamtx-stream-manager.sh status

# Stop streams
sudo ./mediamtx-stream-manager.sh stop
```

### Integration Testing

**Full System Test:**

```bash
# 1. Install MediaMTX
sudo ./install-mediamtx.sh

# 2. Map USB devices
sudo ./lyrebird-usb-mapper.sh

# 3. Create test stream
sudo ./lyrebird-orchestrator.sh
# Select: 4. Add New Stream

# 4. Start streams
sudo ./mediamtx-stream-manager.sh start

# 5. Verify stream
ffmpeg -i rtsp://localhost:8554/your-stream -t 5 -acodec copy test.aac

# 6. Check diagnostics
sudo ./lyrebird-diagnostics.sh full

# 7. Stop streams
sudo ./mediamtx-stream-manager.sh stop
```

### Hardware Testing

**Multiple USB Devices:**

```bash
# Connect multiple USB microphones
# Map all devices
sudo ./lyrebird-usb-mapper.sh

# Verify device persistence
lsusb
ls -l /dev/lyrebird-*

# Test each device independently
for dev in /dev/lyrebird-*; do
    echo "Testing $dev"
    sudo ./lyrebird-capability-checker.sh $dev
done
```

---

## Keeping Your Fork Updated

### Sync with Upstream

**Fetch Latest Changes:**

```bash
# Fetch upstream changes
git fetch upstream

# View changes
git log --oneline main..upstream/main

# Merge into your main branch
git checkout main
git merge upstream/main

# Push to your fork
git push origin main
```

**Rebase Your Feature Branch:**

```bash
# Switch to your feature branch
git checkout feature/your-feature-name

# Rebase onto latest main
git rebase main

# Force push if needed (only on your fork)
git push origin feature/your-feature-name --force-with-lease
```

### Resolve Conflicts

```bash
# If conflicts occur during rebase
git status

# Edit conflicting files
vim conflicted-file.sh

# Mark as resolved
git add conflicted-file.sh

# Continue rebase
git rebase --continue
```

---

## Code Validation

### Shellcheck Validation

**Check All Scripts:**

```bash
# Validate all shell scripts
for script in *.sh; do
    echo "Checking $script..."
    shellcheck "$script"
done
```

**Common Shellcheck Issues:**

```bash
# SC2086: Quote variables
BAD:  rm $FILE
GOOD: rm "$FILE"

# SC2164: Check cd success
BAD:  cd /some/dir
GOOD: cd /some/dir || exit 1

# SC2155: Declare and assign separately
BAD:  local var=$(command)
GOOD: local var
      var=$(command)
```

### Bash Compatibility Testing

```bash
# Ensure Bash 4.0+ compatibility
bash --version

# Test associative arrays
declare -A test_array
test_array["key"]="value"
echo "${test_array["key"]}"
```

---

## Committing Changes

### Commit Guidelines

**1. Stage Your Changes**

```bash
# Stage specific files
git add lyrebird-orchestrator.sh
git add mediamtx-stream-manager.sh

# Or stage all changes
git add .
```

**2. Write Descriptive Commit Messages**

```bash
git commit -m "Fix: Resolve stream restart race condition

- Add mutex lock for concurrent stream operations
- Improve error handling in FFmpeg process spawning
- Add 5-second cooldown between restart attempts

Fixes #42"
```

**Commit Message Format:**

```
Type: Brief summary (50 chars or less)

Detailed explanation of what changed and why.
Include any breaking changes or special notes.

Fixes #issue-number
```

**Commit Types:**
- `Fix:` Bug fixes
- `Feature:` New features
- `Docs:` Documentation changes
- `Refactor:` Code restructuring
- `Perf:` Performance improvements
- `Test:` Testing additions/changes

**3. Push to Your Fork**

```bash
# Push feature branch
git push origin feature/your-feature-name
```

---

## Creating Pull Requests

### Before Submitting

**Pre-PR Checklist:**

- [ ] Code passes shellcheck validation
- [ ] Changes tested locally
- [ ] Hardware tested (if applicable)
- [ ] Documentation updated
- [ ] Commit messages are clear
- [ ] Branch is up-to-date with main
- [ ] No merge conflicts

### Submit Pull Request

**1. Go to GitHub**

Navigate to your fork: `https://github.com/YOUR-USERNAME/LyreBirdAudio`

**2. Create Pull Request**

Click **Compare & pull request** or **New pull request**

**3. Fill PR Template**

```markdown
## Summary
Brief description of changes

## Changes Made
- Added feature X
- Fixed bug Y
- Updated documentation for Z

## Testing Performed
- Tested on Ubuntu 22.04 with 3 USB microphones
- Verified stream stability for 24 hours
- Validated udev rule generation

## Related Issues
Fixes #42
Related to #38

## Screenshots (if applicable)
[Add screenshots for UI changes]

## Checklist
- [x] Code follows project standards
- [x] Shellcheck validation passed
- [x] Documentation updated
- [x] Tested locally
- [x] Hardware tested
```

**4. Request Review**

Tag maintainers or wait for automated review assignment.

---

## Development Best Practices

### Code Organization

**Modular Design:**

```bash
# Good: Separate functions
function validate_device() {
    local device="$1"
    # Validation logic
}

function process_device() {
    local device="$1"
    validate_device "$device" || return 1
    # Processing logic
}

# Bad: Monolithic code
function do_everything() {
    # 500 lines of mixed logic
}
```

**Error Handling:**

```bash
# Always check command success
if ! command_that_might_fail; then
    echo "Error: Command failed" >&2
    return 1
fi

# Use set -e for critical sections
set -e
critical_command_1
critical_command_2
set +e
```

### Documentation

**Inline Comments:**

```bash
# Good: Explain why, not what
# Workaround for ALSA buffer underrun on Raspberry Pi
sleep 0.5

# Bad: State the obvious
# Sleep for 0.5 seconds
sleep 0.5
```

**Function Documentation:**

```bash
#
# Validates USB audio device capabilities
#
# Arguments:
#   $1 - Device path (e.g., /dev/lyrebird-mic-1)
#
# Returns:
#   0 on success
#   1 on validation failure
#
# Example:
#   validate_device "/dev/lyrebird-mic-1"
#
function validate_device() {
    local device="$1"
    # Function implementation
}
```

### Security Considerations

**Input Validation:**

```bash
# Sanitize user input
function sanitize_name() {
    local name="$1"
    # Remove special characters
    name=$(echo "$name" | tr -cd '[:alnum:]-_')
    echo "$name"
}

# Validate paths
function is_valid_device() {
    local device="$1"
    [[ "$device" =~ ^/dev/lyrebird-[a-zA-Z0-9_-]+$ ]]
}
```

**Secure File Operations:**

```bash
# Use absolute paths
CONFIG_FILE="/etc/mediamtx/mediamtx.yml"

# Check file permissions
if [[ ! -r "$CONFIG_FILE" ]]; then
    echo "Error: Cannot read $CONFIG_FILE" >&2
    exit 1
fi

# Create files with appropriate permissions
install -m 644 config.yml "$CONFIG_FILE"
```

---

## Debugging

### Enable Debug Output

**Bash Debug Mode:**

```bash
# Run script with debug output
bash -x ./lyrebird-orchestrator.sh

# Or add to script
set -x  # Enable debug
# ... code to debug ...
set +x  # Disable debug
```

**Verbose Logging:**

```bash
# Add verbose flag to scripts
VERBOSE=${VERBOSE:-0}

function log_debug() {
    if [[ $VERBOSE -eq 1 ]]; then
        echo "[DEBUG] $*" >&2
    fi
}

# Run with verbose output
VERBOSE=1 sudo ./script.sh
```

### Common Debugging Techniques

**Print Variables:**

```bash
# Debug variable contents
echo "Device: $DEVICE"
echo "Config: $CONFIG_FILE"
declare -p ASSOCIATIVE_ARRAY  # Print array contents
```

**Trace Execution:**

```bash
# Add execution markers
echo "=== Starting device validation ===" >&2
validate_device "$device"
echo "=== Validation complete ===" >&2
```

**Check Exit Codes:**

```bash
# Capture and check exit codes
command_that_might_fail
EXIT_CODE=$?
echo "Exit code: $EXIT_CODE" >&2
```

---

## Performance Testing

### Monitor Resource Usage

```bash
# Monitor script execution
time sudo ./lyrebird-orchestrator.sh

# Monitor FFmpeg processes
top -p $(pgrep -d',' ffmpeg)

# Check memory usage
ps aux | grep ffmpeg | awk '{print $6}'
```

### Profiling Scripts

```bash
# Time specific operations
TIMEFORMAT='%R seconds'
time {
    # Code to profile
    process_all_devices
}
```

---

## Getting Help

### Development Support

**Documentation:**
- [Code Standards](code-standards.md) - Coding conventions
- [Testing Guidelines](testing.md) - Testing procedures
- [Architecture](../advanced/architecture.md) - System design

**Community:**
- GitHub Issues: [Report bugs or request features](https://github.com/tomtom215/LyreBirdAudio/issues)
- GitHub Discussions: Ask questions and share ideas
- Pull Requests: Review others' contributions

**Maintainer Contact:**
- Create an issue for questions
- Tag `@tomtom215` in PR comments
- Use GitHub Discussions for general questions

---

## Related Documentation

- [Code Standards](code-standards.md) - Coding conventions and best practices
- [Testing](testing.md) - Comprehensive testing guidelines
- [Architecture](../advanced/architecture.md) - System architecture overview
- [Troubleshooting](../advanced/troubleshooting.md) - Common issues and solutions

---

**Next Steps:** [Code Standards](code-standards.md) | [Contributing Overview](index.md)
