# Testing

Comprehensive testing guidelines for LyreBirdAudio contributors.

---

## Overview

Thorough testing ensures LyreBirdAudio remains reliable and stable across diverse hardware configurations. This guide covers manual testing procedures, hardware validation, stream verification, performance testing, and pre-release validation.

**Testing Principles:**

- **Hardware First:** Test with real USB devices
- **Real-World Conditions:** Simulate production environments
- **Duration Testing:** Verify long-term stability
- **Edge Cases:** Test boundary conditions
- **Regression Prevention:** Ensure fixes don't break existing functionality

---

## Testing Environment

## Recommended Test Hardware

**Minimum Test Configuration:**

| Component | Specification | Purpose |
|-----------|---------------|---------|
| Test System | Linux x86_64 or ARM | Primary platform |
| USB Microphones | 2-3 devices | Multi-device testing |
| Network | Ethernet or WiFi | Stream delivery testing |
| Storage | 10GB free | Logs and recordings |

**Production-Grade Test Configuration:**

| Component | Specification | Purpose |
|-----------|---------------|---------|
| Test System | Intel N100 or equivalent | Performance validation |
| USB Microphones | 5+ devices | Load testing |
| Network | Gigabit Ethernet | Bandwidth testing |
| Storage | 50GB+ free | Extended testing |
| Test Duration | 72+ hours | Stability validation |

## Test Environment Setup

**1. Create Test Directory**

```bash
# Create isolated test environment
sudo mkdir -p /opt/LyreBirdAudio-test
sudo chown $USER:$USER /opt/LyreBirdAudio-test

# Copy development code
cp -r ~/LyreBirdAudio/* /opt/LyreBirdAudio-test/
cd /opt/LyreBirdAudio-test
```

**2. Install Test Dependencies**

```bash
# Install testing tools
sudo apt-get install -y \
    ffmpeg \
    alsa-utils \
    vlc \
    curl \
    jq \
    shellcheck

# Verify installations
ffmpeg -version
arecord -l
shellcheck --version
```

**3. Configure Test Devices**

```bash
# List available USB devices
lsusb
arecord -l

# Map test devices
sudo ./lyrebird-usb-mapper.sh

# Verify device mapping
ls -l /dev/lyrebird-*
```

---

## Pre-Submission Testing

## Code Validation

**1. Shellcheck Validation**

```bash
# Check all scripts
for script in *.sh; do
    echo "Validating $script..."
    shellcheck "$script" || {
        echo "FAILED: $script has shellcheck errors"
        exit 1
    }
done

echo "All scripts passed shellcheck validation"
```

**2. Syntax Checking**

```bash
# Verify Bash syntax
for script in *.sh; do
    echo "Checking syntax: $script"
    bash -n "$script" || {
        echo "FAILED: Syntax error in $script"
        exit 1
    }
done

echo "All scripts have valid syntax"
```

**3. Bash Version Compatibility**

```bash
# Ensure Bash 4.0+ compatibility
bash --version | head -1

# Test associative arrays
bash -c 'declare -A test; test["key"]="value"; echo ${test["key"]}' || {
    echo "FAILED: Bash 4.0+ features not supported"
    exit 1
}
```

---

## Functional Testing

## Script Testing

**Test MediaMTX Installation:**

```bash
# Test MediaMTX installer
echo "Testing MediaMTX installation..."
sudo ./install-mediamtx.sh

# Verify installation
if [[ -x /opt/mediamtx/mediamtx ]]; then
    echo "PASS: MediaMTX installed successfully"
    /opt/mediamtx/mediamtx --version
else
    echo "FAIL: MediaMTX not installed"
    exit 1
fi

# Verify configuration
if [[ -f /etc/mediamtx/mediamtx.yml ]]; then
    echo "PASS: Configuration file created"
else
    echo "FAIL: Configuration file missing"
    exit 1
fi
```

**Test USB Mapper:**

```bash
# Test USB device mapping
echo "Testing USB device mapper..."

# Count devices before mapping
BEFORE_COUNT=$(ls -1 /dev/lyrebird-* 2>/dev/null | wc -l)

# Run mapper
sudo ./lyrebird-usb-mapper.sh

# Count devices after mapping
AFTER_COUNT=$(ls -1 /dev/lyrebird-* 2>/dev/null | wc -l)

if [[ $AFTER_COUNT -gt $BEFORE_COUNT ]]; then
    echo "PASS: USB devices mapped successfully"
    ls -l /dev/lyrebird-*
else
    echo "FAIL: No new devices mapped"
    exit 1
fi

# Verify udev rules
if [[ -f /etc/udev/rules.d/99-usb-soundcards.rules ]]; then
    echo "PASS: Udev rules created"
    cat /etc/udev/rules.d/99-usb-soundcards.rules
else
    echo "FAIL: Udev rules not created"
    exit 1
fi
```

**Test Capability Checker:**

```bash
# Test device capability detection
echo "Testing capability checker..."

for device in /dev/lyrebird-*; do
    echo "Checking device: $device"

    # Run capability checker
    if sudo ./lyrebird-capability-checker.sh "$device"; then
        echo "PASS: Capability check successful for $device"
    else
        echo "FAIL: Capability check failed for $device"
        exit 1
    fi
done
```

**Test Stream Manager:**

```bash
# Test stream manager functionality
echo "Testing stream manager..."

# Start streams
sudo ./mediamtx-stream-manager.sh start
sleep 5

# Check status
if sudo ./mediamtx-stream-manager.sh status | grep -q "running"; then
    echo "PASS: Streams started successfully"
else
    echo "FAIL: Streams failed to start"
    sudo journalctl -u mediamtx-audio -n 20
    exit 1
fi

# Stop streams
sudo ./mediamtx-stream-manager.sh stop
sleep 3

# Verify stopped
if ! pgrep -f "mediamtx" > /dev/null; then
    echo "PASS: Streams stopped successfully"
else
    echo "FAIL: Streams still running"
    exit 1
fi
```

---

## Hardware Testing

## USB Device Testing

**Single Device Test:**

```bash
# Test single USB microphone
DEVICE="/dev/lyrebird-test-mic"

echo "Testing device: $DEVICE"

# 1. Verify device exists
if [[ ! -e "$DEVICE" ]]; then
    echo "FAIL: Device not found: $DEVICE"
    exit 1
fi

# 2. Test ALSA recording
arecord -D "$DEVICE" -f S16_LE -r 48000 -c 1 -d 5 test-recording.wav

# 3. Verify recording
if [[ -f test-recording.wav ]] && [[ $(stat -c%s test-recording.wav) -gt 0 ]]; then
    echo "PASS: Device recording successful"
    rm test-recording.wav
else
    echo "FAIL: Recording failed"
    exit 1
fi

# 4. Test FFmpeg encoding
ffmpeg -f alsa -i "$DEVICE" -t 5 -acodec aac -b:a 128k test-stream.aac -y

# 5. Verify encoding
if [[ -f test-stream.aac ]] && [[ $(stat -c%s test-stream.aac) -gt 0 ]]; then
    echo "PASS: FFmpeg encoding successful"
    rm test-stream.aac
else
    echo "FAIL: Encoding failed"
    exit 1
fi
```

**Multiple Device Test:**

```bash
# Test all mapped devices simultaneously
echo "Testing all USB devices..."

for device in /dev/lyrebird-*; do
    device_name=$(basename "$device")

    echo "Starting test for: $device_name"

    # Start background recording
    ffmpeg -f alsa -i "$device" -t 10 -acodec aac \
        -b:a 128k "test-${device_name}.aac" -y &

    echo "Recording from $device_name in background (PID: $!)"
done

# Wait for all recordings to complete
wait

# Verify all recordings
FAILED=0
for device in /dev/lyrebird-*; do
    device_name=$(basename "$device")
    test_file="test-${device_name}.aac"

    if [[ -f "$test_file" ]] && [[ $(stat -c%s "$test_file") -gt 0 ]]; then
        echo "PASS: $device_name recording successful"
        rm "$test_file"
    else
        echo "FAIL: $device_name recording failed"
        FAILED=1
    fi
done

[[ $FAILED -eq 0 ]] && echo "All devices passed" || exit 1
```

## USB Persistence Testing

**Device Reconnection Test:**

```bash
# Test USB device persistence after reconnection
echo "Testing USB device persistence..."

# 1. Record initial device paths
declare -A DEVICE_PATHS
for device in /dev/lyrebird-*; do
    device_name=$(basename "$device")
    device_target=$(readlink -f "$device")
    DEVICE_PATHS["$device_name"]="$device_target"
    echo "Initial: $device_name -> $device_target"
done

# 2. Prompt for device reconnection
echo ""
echo "Please disconnect and reconnect a USB microphone"
echo "Press Enter when reconnection is complete..."
read -r

# 3. Verify device paths unchanged
sleep 2
udevadm trigger

FAILED=0
for device_name in "${!DEVICE_PATHS[@]}"; do
    device="/dev/$device_name"
    if [[ -e "$device" ]]; then
        new_target=$(readlink -f "$device")
        old_target="${DEVICE_PATHS[$device_name]}"

        if [[ "$new_target" == "$old_target" ]]; then
            echo "PASS: $device_name persistence maintained"
        else
            echo "FAIL: $device_name changed from $old_target to $new_target"
            FAILED=1
        fi
    else
        echo "WARN: $device_name no longer exists (was it disconnected?)"
    fi
done

[[ $FAILED -eq 0 ]] && echo "Persistence test passed" || exit 1
```

---

## Stream Validation

## RTSP Stream Testing

**Stream Connectivity Test:**

```bash
# Test RTSP stream accessibility
echo "Testing RTSP streams..."

# Get list of active streams
STREAMS=$(curl -s http://localhost:9997/v3/paths/list | jq -r '.items[].name')

for stream in $STREAMS; do
    echo "Testing stream: $stream"

    # Test with FFmpeg
    timeout 10 ffmpeg -i "rtsp://localhost:8554/$stream" \
        -t 5 -acodec copy "test-${stream}.aac" -y 2>/dev/null

    if [[ -f "test-${stream}.aac" ]]; then
        size=$(stat -c%s "test-${stream}.aac")
        if [[ $size -gt 1000 ]]; then
            echo "PASS: Stream $stream is working (size: $size bytes)"
            rm "test-${stream}.aac"
        else
            echo "FAIL: Stream $stream has insufficient data"
            exit 1
        fi
    else
        echo "FAIL: Could not capture stream $stream"
        exit 1
    fi
done
```

**Audio Quality Test:**

```bash
# Test audio quality parameters
STREAM="lyrebird-test-mic"

echo "Testing audio quality for stream: $stream"

# Capture 30 seconds
ffmpeg -i "rtsp://localhost:8554/$STREAM" -t 30 \
    -acodec copy "quality-test.aac" -y 2>&1 | tee ffmpeg-output.log

# Extract audio parameters from FFmpeg output
SAMPLE_RATE=$(grep "Stream #0:0" ffmpeg-output.log | grep -oP '\d+ Hz' | head -1)
BITRATE=$(grep "bitrate:" ffmpeg-output.log | grep -oP '\d+ kb/s' | tail -1)

echo "Detected parameters:"
echo "  Sample Rate: $SAMPLE_RATE"
echo "  Bitrate: $BITRATE"

# Verify quality meets standards
if [[ "$SAMPLE_RATE" == "48000 Hz" ]] || [[ "$SAMPLE_RATE" == "44100 Hz" ]]; then
    echo "PASS: Sample rate acceptable"
else
    echo "FAIL: Unexpected sample rate: $SAMPLE_RATE"
    exit 1
fi

# Cleanup
rm -f quality-test.aac ffmpeg-output.log
```

## Stream Stability Test

**Short-Term Stability (5 minutes):**

```bash
# Test stream stability over 5 minutes
STREAM="lyrebird-test-mic"
DURATION=300  # 5 minutes

echo "Testing stream stability for $DURATION seconds..."

# Start capture
ffmpeg -i "rtsp://localhost:8554/$STREAM" \
    -t $DURATION -acodec copy "stability-test.aac" -y 2>&1 | tee stability.log &

FFMPEG_PID=$!

# Monitor for errors
sleep 5
ERRORS=0

while kill -0 $FFMPEG_PID 2>/dev/null; do
    # Check for buffer underruns
    if grep -q "buffer underrun\|DTS out of order" stability.log; then
        echo "WARN: Audio issues detected"
        ((ERRORS++))
    fi

    sleep 10
done

# Wait for completion
wait $FFMPEG_PID
EXIT_CODE=$?

# Verify results
if [[ $EXIT_CODE -eq 0 ]] && [[ $ERRORS -lt 5 ]]; then
    echo "PASS: Stream stable for $DURATION seconds"
    file_size=$(stat -c%s "stability-test.aac")
    echo "Captured: $file_size bytes"
else
    echo "FAIL: Stream instability detected"
    echo "Exit code: $EXIT_CODE, Errors: $ERRORS"
    exit 1
fi

# Cleanup
rm -f stability-test.aac stability.log
```

---

## Performance Testing

## Resource Usage Monitoring

**CPU and Memory Test:**

```bash
# Monitor resource usage during streaming
echo "Monitoring resource usage..."

# Start all streams
sudo ./mediamtx-stream-manager.sh start
sleep 10

# Monitor for 60 seconds
echo "Collecting metrics for 60 seconds..."

for i in {1..12}; do
    # CPU usage
    CPU_USAGE=$(ps aux | grep -E "ffmpeg|mediamtx" | grep -v grep | \
        awk '{sum+=$3} END {print sum}')

    # Memory usage (MB)
    MEM_USAGE=$(ps aux | grep -E "ffmpeg|mediamtx" | grep -v grep | \
        awk '{sum+=$6} END {print sum/1024}')

    # Process count
    PROC_COUNT=$(pgrep -c "ffmpeg\|mediamtx")

    echo "Iteration $i: CPU: ${CPU_USAGE}% | MEM: ${MEM_USAGE}MB | Processes: $PROC_COUNT"

    sleep 5
done

# Calculate averages
echo ""
echo "Performance test complete"
echo "Verify CPU usage is reasonable (<50% on multi-core systems)"
echo "Verify memory usage is stable (not continuously increasing)"

# Stop streams
sudo ./mediamtx-stream-manager.sh stop
```

**Bandwidth Test:**

```bash
# Test network bandwidth usage
STREAM="lyrebird-test-mic"

echo "Testing bandwidth usage..."

# Capture with statistics
ffmpeg -i "rtsp://localhost:8554/$STREAM" -t 60 \
    -acodec copy bandwidth-test.aac -y 2>&1 | \
    grep "bitrate=" | tail -1

# Calculate expected bandwidth
# AAC at 128kbps should be ~128kbps + RTSP overhead (~20%)
# Expected: ~150-160 kbps

echo "Verify bandwidth is appropriate for configured bitrate"

# Cleanup
rm -f bandwidth-test.aac
```

## Load Testing

**High-Load Test (5+ devices):**

```bash
# Test system under high load
echo "Starting high-load test with all devices..."

# Count devices
DEVICE_COUNT=$(ls -1 /dev/lyrebird-* | wc -l)

echo "Testing with $DEVICE_COUNT devices"

if [[ $DEVICE_COUNT -lt 5 ]]; then
    echo "WARN: Less than 5 devices available"
    echo "High-load testing requires 5+ USB microphones"
fi

# Start all streams
sudo ./mediamtx-stream-manager.sh start
sleep 10

# Monitor system under load
echo "Monitoring system under load for 5 minutes..."

START_TIME=$(date +%s)
DURATION=300  # 5 minutes

while [[ $(($(date +%s) - START_TIME)) -lt $DURATION ]]; do
    # Check stream health
    HEALTHY=$(sudo ./mediamtx-stream-manager.sh status | grep -c "running")

    # Check system load
    LOAD=$(uptime | awk -F'load average:' '{print $2}' | awk '{print $1}')

    echo "Time: $(($(date +%s) - START_TIME))s | Healthy streams: $HEALTHY/$DEVICE_COUNT | Load: $LOAD"

    if [[ $HEALTHY -lt $DEVICE_COUNT ]]; then
        echo "WARN: Some streams are unhealthy"
    fi

    sleep 30
done

echo "Load test complete"

# Stop streams
sudo ./mediamtx-stream-manager.sh stop
```

---

## Integration Testing

## Full System Integration Test

**Complete Workflow Test:**

```bash
#!/bin/bash
#
# Full system integration test
#

set -e

echo "========================================="
echo "LyreBirdAudio Integration Test"
echo "========================================="

# 1. Install MediaMTX
echo "Step 1: Installing MediaMTX..."
sudo ./install-mediamtx.sh
[[ -x /opt/mediamtx/mediamtx ]] || exit 1
echo "PASS: MediaMTX installed"

# 2. Map USB devices
echo "Step 2: Mapping USB devices..."
sudo ./lyrebird-usb-mapper.sh
[[ $(ls -1 /dev/lyrebird-* 2>/dev/null | wc -l) -gt 0 ]] || exit 1
echo "PASS: USB devices mapped"

# 3. Check device capabilities
echo "Step 3: Checking device capabilities..."
for device in /dev/lyrebird-*; do
    sudo ./lyrebird-capability-checker.sh "$device" || exit 1
done
echo "PASS: All devices checked"

# 4. Install systemd service
echo "Step 4: Installing systemd service..."
# Note: This would typically be done through orchestrator
echo "SKIP: Manual service installation in test environment"

# 5. Start streams
echo "Step 5: Starting streams..."
sudo ./mediamtx-stream-manager.sh start
sleep 10
echo "PASS: Streams started"

# 6. Verify stream health
echo "Step 6: Verifying stream health..."
HEALTHY=$(sudo ./mediamtx-stream-manager.sh status | grep -c "running")
TOTAL=$(ls -1 /dev/lyrebird-* | wc -l)

if [[ $HEALTHY -eq $TOTAL ]]; then
    echo "PASS: All $TOTAL streams healthy"
else
    echo "FAIL: Only $HEALTHY/$TOTAL streams healthy"
    exit 1
fi

# 7. Test stream playback
echo "Step 7: Testing stream playback..."
STREAM=$(ls -1 /dev/lyrebird-* | head -1 | xargs basename)
timeout 15 ffmpeg -i "rtsp://localhost:8554/$STREAM" \
    -t 10 -acodec copy integration-test.aac -y || exit 1
[[ -f integration-test.aac ]] || exit 1
echo "PASS: Stream playback successful"

# 8. Run diagnostics
echo "Step 8: Running diagnostics..."
sudo ./lyrebird-diagnostics.sh quick || exit 1
echo "PASS: Diagnostics successful"

# 9. Stop streams
echo "Step 9: Stopping streams..."
sudo ./mediamtx-stream-manager.sh stop
sleep 3
! pgrep -f "mediamtx" > /dev/null || exit 1
echo "PASS: Streams stopped"

# Cleanup
rm -f integration-test.aac

echo ""
echo "========================================="
echo "Integration Test: SUCCESS"
echo "========================================="
```

---

## Pre-Release Testing

### 72-Hour Stability Test

**Production-Grade Validation:**

```bash
#!/bin/bash
#
# 72-hour stability test for production release
#

# Test configuration
TEST_DURATION=$((72 * 3600))  # 72 hours in seconds
CHECK_INTERVAL=300  # 5 minutes
LOG_DIR="/var/log/lyrebird-stability-test"

# Setup
mkdir -p "$LOG_DIR"
START_TIME=$(date +%s)
END_TIME=$((START_TIME + TEST_DURATION))

echo "Starting 72-hour stability test"
echo "Start: $(date)"
echo "Expected end: $(date -d @$END_TIME)"

# Start all streams
sudo ./mediamtx-stream-manager.sh start

# Monitor loop
while [[ $(date +%s) -lt $END_TIME ]]; do
    CURRENT_TIME=$(date +%s)
    ELAPSED=$((CURRENT_TIME - START_TIME))
    REMAINING=$((END_TIME - CURRENT_TIME))

    # Collect metrics
    TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S')
    HEALTHY=$(sudo ./mediamtx-stream-manager.sh status | grep -c "running")
    TOTAL=$(ls -1 /dev/lyrebird-* | wc -l)
    CPU=$(ps aux | grep -E "ffmpeg|mediamtx" | grep -v grep | awk '{sum+=$3} END {print sum}')
    MEM=$(ps aux | grep -E "ffmpeg|mediamtx" | grep -v grep | awk '{sum+=$6} END {print sum/1024}')
    LOAD=$(uptime | awk -F'load average:' '{print $2}' | awk '{print $1}')

    # Log metrics
    echo "$TIMESTAMP,$ELAPSED,$HEALTHY,$TOTAL,$CPU,$MEM,$LOAD" >> "$LOG_DIR/metrics.csv"

    # Display progress
    HOURS_ELAPSED=$((ELAPSED / 3600))
    HOURS_REMAINING=$((REMAINING / 3600))
    echo "[$TIMESTAMP] Elapsed: ${HOURS_ELAPSED}h | Remaining: ${HOURS_REMAINING}h | Streams: $HEALTHY/$TOTAL | CPU: ${CPU}% | Mem: ${MEM}MB | Load: $LOAD"

    # Alert if streams are unhealthy
    if [[ $HEALTHY -lt $TOTAL ]]; then
        echo "ALERT: Unhealthy streams detected at $TIMESTAMP" | \
            tee -a "$LOG_DIR/alerts.log"

        # Restart failed streams
        sudo ./mediamtx-stream-manager.sh restart
    fi

    # Sleep until next check
    sleep $CHECK_INTERVAL
done

# Test complete
echo ""
echo "72-hour stability test complete"
echo "End: $(date)"

# Generate report
echo "Generating test report..."

TOTAL_CHECKS=$(wc -l < "$LOG_DIR/metrics.csv")
AVG_CPU=$(awk -F',' '{sum+=$5; count++} END {print sum/count}' "$LOG_DIR/metrics.csv")
AVG_MEM=$(awk -F',' '{sum+=$6; count++} END {print sum/count}' "$LOG_DIR/metrics.csv")
ALERT_COUNT=$(wc -l < "$LOG_DIR/alerts.log" 2>/dev/null || echo "0")

cat > "$LOG_DIR/report.txt" << EOF
LyreBirdAudio 72-Hour Stability Test Report
============================================

Test Duration: 72 hours
Total Checks: $TOTAL_CHECKS
Check Interval: ${CHECK_INTERVAL}s

Performance Metrics:
- Average CPU: ${AVG_CPU}%
- Average Memory: ${AVG_MEM}MB
- Alert Count: $ALERT_COUNT

Status: $([ $ALERT_COUNT -lt 10 ] && echo "PASS" || echo "NEEDS REVIEW")

Detailed metrics: $LOG_DIR/metrics.csv
Alert log: $LOG_DIR/alerts.log
EOF

cat "$LOG_DIR/report.txt"

# Stop streams
sudo ./mediamtx-stream-manager.sh stop
```

---

## Regression Testing

### Pre-Commit Regression Tests

**Quick Regression Test:**

```bash
#!/bin/bash
#
# Quick regression test before committing changes
#

echo "Running regression tests..."

FAILED=0

# Test 1: Shellcheck validation
echo "Test 1: Shellcheck validation..."
for script in *.sh; do
    shellcheck "$script" || FAILED=1
done

# Test 2: Basic functionality
echo "Test 2: Basic functionality..."
sudo ./lyrebird-usb-mapper.sh || FAILED=1

# Test 3: Stream start/stop
echo "Test 3: Stream management..."
sudo ./mediamtx-stream-manager.sh start || FAILED=1
sleep 5
sudo ./mediamtx-stream-manager.sh status | grep -q "running" || FAILED=1
sudo ./mediamtx-stream-manager.sh stop || FAILED=1

# Test 4: Diagnostics
echo "Test 4: Diagnostics..."
sudo ./lyrebird-diagnostics.sh quick || FAILED=1

# Results
if [[ $FAILED -eq 0 ]]; then
    echo "All regression tests PASSED"
    exit 0
else
    echo "Regression tests FAILED"
    exit 1
fi
```

---

## Test Documentation

## Test Results Template

```markdown
# Test Results: [Feature/Bug Fix Name]

## Test Environment
- System: Ubuntu 22.04 LTS
- Hardware: Intel N100
- USB Devices: 5x USB microphones
- Test Duration: 24 hours

## Tests Performed

## Functional Tests
- [x] Shellcheck validation passed
- [x] MediaMTX installation successful
- [x] USB device mapping works
- [x] Stream start/stop works
- [x] Stream playback verified

## Hardware Tests
- [x] Single device test passed
- [x] Multiple device test passed (5 devices)
- [x] USB persistence verified
- [x] Device reconnection handled correctly

## Performance Tests
- [x] CPU usage acceptable (<30%)
- [x] Memory stable (no leaks detected)
- [x] Bandwidth appropriate for bitrate
- [x] Load test passed (5 devices, 24 hours)

## Integration Tests
- [x] Full system workflow completed
- [x] Diagnostics successful
- [x] No errors in logs

## Issues Found
None

## Recommendations
- Ready for merge
- Consider additional testing with 10+ devices

## Test Artifacts
- Logs: /var/log/lyrebird-test/
- Recordings: /tmp/test-recordings/
- Metrics: /tmp/test-metrics.csv
```

---

## Related Documentation

- [Development Setup](development-setup.md) - Development environment setup
- [Code Standards](code-standards.md) - Coding standards and practices
- [Architecture](../advanced/architecture.md) - System architecture
- [Troubleshooting](../advanced/troubleshooting.md) - Common issues

---

## Next Steps

<div class="grid" markdown>

<div markdown>
## Development Setup
Setting up development environment

[Development Setup →](development-setup.md)
</div>

<div markdown>
## Contributing Overview
Contributing guidelines and workflows

[Contributing Overview →](index.md)
</div>

</div>
