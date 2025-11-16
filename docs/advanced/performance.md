# Performance Tuning

Comprehensive guide to optimizing LyreBirdAudio for resource usage, stream quality, and scalability.

---

## Overview

Performance tuning balances three critical factors: resource usage (CPU and memory), audio quality, and the number of concurrent streams. This guide provides strategies for optimizing each aspect based on your hardware platform and use case requirements.

This guide covers:

- Resource usage patterns and metrics
- Platform-specific optimization strategies
- Codec selection and quality tuning
- Multi-stream optimization
- Hardware recommendations
- Performance monitoring and troubleshooting

---

## Resource Usage Patterns

### Per-Stream Resource Consumption

Each RTSP audio stream consumes system resources based on encoding settings:

**Typical Resource Usage:**

| Quality Preset | CPU per Stream | Memory per Stream | Network Bandwidth |
|----------------|----------------|-------------------|-------------------|
| Low (16kHz) | 1-2% | 30-40 MB | 64 kbps |
| Normal (48kHz) | 2-4% | 40-60 MB | 128 kbps |
| High (48kHz) | 4-6% | 60-80 MB | 192 kbps |

**Factors Affecting Resource Usage:**

1. **Sample Rate**
   - 16 kHz: Voice-optimized, minimal resources
   - 44.1/48 kHz: Standard quality, moderate resources
   - 96 kHz: High fidelity, maximum resources

2. **Codec Selection**
   - Opus: Most efficient, excellent quality/performance ratio
   - AAC: Good efficiency, widely compatible
   - MP3: Moderate efficiency, legacy compatibility
   - PCM: No encoding overhead, very high bandwidth

3. **Bitrate**
   - Lower bitrate = less CPU for encoding, less network bandwidth
   - Higher bitrate = more CPU for encoding, more network bandwidth
   - Diminishing returns above optimal bitrate for codec

4. **Channel Configuration**
   - Mono: Half the data of stereo
   - Stereo: Double processing and bandwidth

## Base System Overhead

Beyond per-stream costs, the system requires base resources:

**Base Resource Requirements:**

| Component | CPU | Memory | File Descriptors |
|-----------|-----|--------|------------------|
| MediaMTX | 1-2% | 100-150 MB | 50-100 |
| System Scripts | <1% | 50 MB | 20-30 |
| **Total Base** | **2-3%** | **150-200 MB** | **70-130** |

## Total Resource Calculation

**Formula:**

```text
Total CPU = Base_CPU + (Num_Streams × Stream_CPU)
Total Memory = Base_Memory + (Num_Streams × Stream_Memory)
Total FDs = Base_FDs + (Num_Streams × FDs_per_Stream)
```

**Example Calculation (10 streams at normal quality):**

```text
CPU: 3% + (10 × 3%) = 33% total
Memory: 200 MB + (10 × 50 MB) = 700 MB total
FDs: 100 + (10 × 12) = 220 file descriptors
```

---

## Platform-Specific Optimization

## Raspberry Pi Limitations

**Platform:** Raspberry Pi 3/4

**Constraints:**

- Limited CPU power (ARM Cortex-A)
- Shared USB bus bandwidth
- Thermal throttling under sustained load
- Limited memory (1-8 GB depending on model)

**Recommended Configuration:**

```bash
# Maximum 2 USB microphones
# Use low to normal quality preset

# Example configuration for Raspberry Pi
DEVICE_Mic1_CODEC=opus
DEVICE_Mic1_SAMPLE_RATE=16000
DEVICE_Mic1_BITRATE=64k
DEVICE_Mic1_CHANNELS=1

DEVICE_Mic2_CODEC=opus
DEVICE_Mic2_SAMPLE_RATE=16000
DEVICE_Mic2_BITRATE=64k
DEVICE_Mic2_CHANNELS=1
```

**Optimization Tips:**

1. **Use Opus Codec**
   - Most efficient for ARM processors
   - Lower CPU usage than AAC

2. **Limit Sample Rate**
   - Use 16 kHz for voice applications
   - Maximum 48 kHz for music

3. **Use Powered USB Hub**
   - Reduces power draw from Pi
   - Prevents USB bus saturation
   - Improves device stability

4. **Enable CPU Governor Performance Mode**
   ```bash
   # Set CPU to performance mode
   echo performance | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
   ```

5. **Monitor Thermal Throttling**
   ```bash
   # Check for throttling
   vcgencmd get_throttled
   # 0x0 = no throttling
   # Other values indicate thermal or voltage issues
   ```

**Raspberry Pi Capacity:**

| Model | Max Streams (Low Quality) | Max Streams (Normal Quality) |
|-------|---------------------------|------------------------------|
| Pi 3B+ | 1-2 | 1 |
| Pi 4 (2GB) | 2-3 | 2 |
| Pi 4 (4/8GB) | 3-4 | 2 |

!!! warning "USB Bus Limitations"
    Raspberry Pi shares USB bandwidth with Ethernet (on Pi 3 and 4). Heavy network traffic can impact USB audio device performance. Use separate network interface (WiFi) for streaming if possible.

## Intel N100 Optimization

**Platform:** Intel N100 Mini PC

**Capabilities:**

- 4 cores with modern x86 architecture
- Dedicated USB controllers
- Efficient power consumption
- Adequate memory (8-16 GB typical)

**Recommended Configuration:**

```bash
# Support 4-8 devices with normal quality
# Can handle high quality for critical streams

# Example configuration for Intel N100
DEVICE_Studio_Mic_CODEC=aac
DEVICE_Studio_Mic_SAMPLE_RATE=48000
DEVICE_Studio_Mic_BITRATE=192k
DEVICE_Studio_Mic_CHANNELS=2

# Other devices use normal quality
DEVICE_Room1_CODEC=opus
DEVICE_Room1_SAMPLE_RATE=48000
DEVICE_Room1_BITRATE=128k
DEVICE_Room1_CHANNELS=1
```

**Optimization Tips:**

1. **Mix Quality Levels**
   - High quality for important streams
   - Normal quality for monitoring streams

2. **Leverage Multiple USB Controllers**
   - Distribute devices across USB ports
   - Check controller assignment: `lsusb -t`

3. **Enable Hardware Acceleration** (if available)
   ```bash
   # Check for QuickSync support
   vainfo

   # Use hardware acceleration if supported
   DEVICE_NAME_HWACCEL=qsv  # Intel QuickSync
   ```

**Intel N100 Capacity:**

| Quality Mix | Max Streams | CPU Usage | Memory Usage |
|-------------|-------------|-----------|--------------|
| All Low | 12-16 | 25-35% | 500-700 MB |
| All Normal | 8-10 | 30-40% | 600-800 MB |
| All High | 4-6 | 25-35% | 500-700 MB |
| Mixed | 8-10 | 25-40% | 600-800 MB |

## High-Performance Workstation

**Platform:** Desktop/Server with multi-core CPU

**Capabilities:**

- Many cores (8+)
- Large memory (32+ GB)
- Multiple USB controllers
- Dedicated NICs

**Recommended Configuration:**

```bash
# Support 20+ devices
# Use quality appropriate to use case
# Implement advanced features (multiplex, etc.)

# High-quality reference stream
DEVICE_Reference_CODEC=aac
DEVICE_Reference_SAMPLE_RATE=96000
DEVICE_Reference_BITRATE=256k
DEVICE_Reference_CHANNELS=2

# Standard monitoring streams
DEVICE_Monitor_*_CODEC=opus
DEVICE_Monitor_*_SAMPLE_RATE=48000
DEVICE_Monitor_*_BITRATE=128k
DEVICE_Monitor_*_CHANNELS=1
```

**Optimization Tips:**

1. **NUMA Awareness** (multi-socket systems)
   ```bash
   # Pin processes to NUMA nodes
   numactl --cpunodebind=0 --membind=0 ./mediamtx-stream-manager.sh start
   ```

2. **Use Process Priorities**
   ```bash
   # Increase priority for critical streams
   nice -n -10 ffmpeg [args for critical stream]
   ```

3. **Dedicated Network Interface**
   - Separate NIC for RTSP streaming
   - Prevents interference with other network services

4. **SSD Storage for Logs**
   - Reduces I/O bottlenecks
   - Improves diagnostic performance

**Workstation Capacity:**

| CPU Cores | Max Low Quality | Max Normal Quality | Max High Quality |
|-----------|-----------------|--------------------| -----------------|
| 8 cores | 40+ | 30+ | 15-20 |
| 12 cores | 60+ | 45+ | 25-30 |
| 16+ cores | 80+ | 60+ | 35-40 |

---

## Codec Selection and Tuning

## Codec Comparison

**Performance Characteristics:**

| Codec | CPU Efficiency | Quality | Bandwidth | Compatibility | Best For |
|-------|----------------|---------|-----------|---------------|----------|
| **Opus** | Excellent | Excellent | Very Low | Modern | Voice, general purpose |
| **AAC** | Good | Excellent | Medium | Universal | Music, high quality |
| **MP3** | Good | Good | Medium | Universal | Legacy compatibility |
| **PCM** | Excellent | Perfect | Very High | Universal | Local recording, no latency |

## Opus Optimization

**Best For:** Voice, podcasts, general audio streaming

**Advantages:**

- Lowest CPU usage for given quality
- Excellent quality at low bitrates
- Adaptive bitrate capability
- Low latency

**Recommended Settings:**

```bash
# Voice-optimized (minimal resources)
DEVICE_NAME_CODEC=opus
DEVICE_NAME_SAMPLE_RATE=16000
DEVICE_NAME_BITRATE=32k
DEVICE_NAME_CHANNELS=1

# Balanced quality (normal usage)
DEVICE_NAME_CODEC=opus
DEVICE_NAME_SAMPLE_RATE=48000
DEVICE_NAME_BITRATE=96k
DEVICE_NAME_CHANNELS=1

# High quality music
DEVICE_NAME_CODEC=opus
DEVICE_NAME_SAMPLE_RATE=48000
DEVICE_NAME_BITRATE=160k
DEVICE_NAME_CHANNELS=2
```

**Opus Bitrate Guidelines:**

| Use Case | Sample Rate | Bitrate | Quality Level |
|----------|-------------|---------|---------------|
| Voice (minimal) | 16 kHz | 24-32 kbps | Intelligible |
| Voice (good) | 16 kHz | 48-64 kbps | Clear |
| Music (normal) | 48 kHz | 96-128 kbps | Good |
| Music (high) | 48 kHz | 128-192 kbps | Excellent |

## AAC Optimization

**Best For:** Music, high-fidelity audio, universal compatibility

**Advantages:**

- Excellent quality at medium bitrates
- Universal device support
- Good for music and complex audio

**Recommended Settings:**

```bash
# Standard quality
DEVICE_NAME_CODEC=aac
DEVICE_NAME_SAMPLE_RATE=44100
DEVICE_NAME_BITRATE=128k
DEVICE_NAME_CHANNELS=2

# High quality
DEVICE_NAME_CODEC=aac
DEVICE_NAME_SAMPLE_RATE=48000
DEVICE_NAME_BITRATE=192k
DEVICE_NAME_CHANNELS=2

# Maximum quality
DEVICE_NAME_CODEC=aac
DEVICE_NAME_SAMPLE_RATE=48000
DEVICE_NAME_BITRATE=256k
DEVICE_NAME_CHANNELS=2
```

**AAC Bitrate Guidelines:**

| Use Case | Sample Rate | Bitrate | Quality Level |
|----------|-------------|---------|---------------|
| Voice | 24-48 kHz | 64-96 kbps | Good |
| Music (normal) | 44.1 kHz | 128-160 kbps | Good |
| Music (high) | 48 kHz | 192-256 kbps | Excellent |
| Music (reference) | 48 kHz | 256-320 kbps | Near-lossless |

## PCM (Uncompressed)

**Best For:** Local recording, minimal latency, maximum quality

**Advantages:**

- No encoding overhead (minimal CPU)
- Perfect quality preservation
- Zero latency from encoding

**Disadvantages:**

- Very high bandwidth (1.5 Mbps for stereo 48kHz)
- Large file sizes
- Network-intensive

**Recommended Settings:**

```bash
# Standard PCM
DEVICE_NAME_CODEC=pcm_s16le
DEVICE_NAME_SAMPLE_RATE=48000
DEVICE_NAME_CHANNELS=2

# High-resolution PCM
DEVICE_NAME_CODEC=pcm_s24le
DEVICE_NAME_SAMPLE_RATE=96000
DEVICE_NAME_CHANNELS=2
```

!!! note "PCM Use Cases"
    PCM is best for local networks or when recording for post-processing. Not recommended for internet streaming or resource-constrained systems.

---

## Multi-Stream Optimization

## Stream Prioritization

Assign resources based on stream importance:

**Priority Tiers:**

```bash
# Tier 1: Critical streams (high quality)
DEVICE_MainStage_CODEC=aac
DEVICE_MainStage_SAMPLE_RATE=48000
DEVICE_MainStage_BITRATE=192k

# Tier 2: Important streams (normal quality)
DEVICE_Backup_CODEC=opus
DEVICE_Backup_SAMPLE_RATE=48000
DEVICE_Backup_BITRATE=128k

# Tier 3: Monitoring streams (low quality)
DEVICE_Monitor_CODEC=opus
DEVICE_Monitor_SAMPLE_RATE=16000
DEVICE_Monitor_BITRATE=64k
```

## Staggered Startup

Prevent resource spikes by staggering stream starts:

```bash
# Increase startup delay between streams
STREAM_STARTUP_DELAY=15  # seconds between starts

# Launch streams in batches
sudo ./mediamtx-stream-manager.sh start
```

**Benefits:**

- Prevents CPU spikes
- Reduces memory allocation bursts
- Allows system to stabilize
- Easier to identify problematic devices

## Resource-Based Stream Limiting

Monitor resources and limit streams based on capacity:

```bash
# Set resource thresholds
MAX_CPU_PERCENT=80
MAX_MEMORY_MB=6000
MAX_FILE_DESCRIPTORS=800

# Stream manager respects these limits
# Refuses to start new streams if thresholds exceeded
```

**Monitoring Script Example:**

```bash
#!/bin/bash
# Check if resources available for new stream

current_cpu=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)
current_mem=$(free -m | awk 'NR==2{print $3}')

if (( $(echo "$current_cpu > $MAX_CPU_PERCENT" | bc -l) )); then
    echo "ERROR: CPU usage too high ($current_cpu%)"
    exit 1
fi

if (( current_mem > MAX_MEMORY_MB )); then
    echo "ERROR: Memory usage too high (${current_mem}MB)"
    exit 1
fi

echo "OK: Resources available for new stream"
```

## Buffer Size Tuning

Optimize thread queue sizes for stability:

```bash
# Default thread queue size
DEVICE_NAME_THREAD_QUEUE=1024

# Increase for unreliable USB connections
DEVICE_NAME_THREAD_QUEUE=2048

# Decrease for low-latency applications
DEVICE_NAME_THREAD_QUEUE=512
```

**Thread Queue Guidelines:**

| Scenario | Queue Size | Latency | Stability |
|----------|------------|---------|-----------|
| Low latency | 512 | Minimal | Lower |
| Balanced | 1024 | Low | Good |
| Stability priority | 2048 | Moderate | Excellent |
| Unreliable USB | 4096 | Higher | Maximum |

---

## Quality vs. Performance Tradeoffs

## Quality Presets

LyreBirdAudio provides three quality presets:

**Low Quality (Voice-Optimized):**

```bash
sudo ./lyrebird-mic-check.sh -g --quality=low
```

- Sample Rate: 16 kHz
- Codec: Opus
- Bitrate: 64 kbps
- Channels: Mono
- CPU: 1-2% per stream
- Memory: 30-40 MB per stream
- **Best for:** Voice, monitoring, resource-constrained systems

**Normal Quality (Balanced):**

```bash
sudo ./lyrebird-mic-check.sh -g --quality=normal
```

- Sample Rate: 48 kHz
- Codec: Opus
- Bitrate: 128 kbps
- Channels: Mono or Stereo (auto-detected)
- CPU: 2-4% per stream
- Memory: 40-60 MB per stream
- **Best for:** General purpose, music, podcasts

**High Quality (Maximum Fidelity):**

```bash
sudo ./lyrebird-mic-check.sh -g --quality=high
```

- Sample Rate: 48 kHz
- Codec: AAC
- Bitrate: 192 kbps
- Channels: Stereo
- CPU: 4-6% per stream
- Memory: 60-80 MB per stream
- **Best for:** Music, high-fidelity recording, critical applications

## Custom Tuning

Create custom quality profiles for specific needs:

**Example: Podcast Quality**

```bash
# Optimized for voice with good quality
DEVICE_NAME_CODEC=opus
DEVICE_NAME_SAMPLE_RATE=24000
DEVICE_NAME_BITRATE=96k
DEVICE_NAME_CHANNELS=1
DEVICE_NAME_THREAD_QUEUE=1024
```

**Example: Music Performance**

```bash
# Live music performance capture
DEVICE_NAME_CODEC=aac
DEVICE_NAME_SAMPLE_RATE=48000
DEVICE_NAME_BITRATE=256k
DEVICE_NAME_CHANNELS=2
DEVICE_NAME_THREAD_QUEUE=2048
```

**Example: Surveillance/Monitoring**

```bash
# Minimal resources for always-on monitoring
DEVICE_NAME_CODEC=opus
DEVICE_NAME_SAMPLE_RATE=12000
DEVICE_NAME_BITRATE=24k
DEVICE_NAME_CHANNELS=1
DEVICE_NAME_THREAD_QUEUE=512
```

---

## Performance Monitoring

### Real-Time Monitoring

Monitor stream performance continuously:

```bash
# Launch monitoring mode
sudo ./mediamtx-stream-manager.sh monitor
```

**Monitoring Output:**

```text
Stream Monitor - Updating every 5 seconds
===========================================
[10:30:15] Blue_Yeti: HEALTHY (CPU: 3.2%, MEM: 45 MB)
[10:30:15] USB_Microphone: HEALTHY (CPU: 2.8%, MEM: 42 MB)
[10:30:15] System: FD: 245/1024, Total CPU: 6.0%

Resource Summary:
  Total CPU: 6.0%
  Total Memory: 687 MB
  File Descriptors: 245 / 1024
  Active Streams: 2
```

## System Resource Monitoring

Use system tools for detailed analysis:

```bash
# Overall system load
top

# Per-process monitoring
top -p $(pgrep ffmpeg | tr '\n' ',')

# Memory details
free -h

# File descriptor usage
lsof | wc -l

# CPU per core
mpstat -P ALL 1

# Network bandwidth
iftop -i eth0
```

## Performance Metrics Collection

Collect metrics for analysis:

```bash
# Log performance metrics every 60 seconds
while true; do
    echo "$(date '+%Y-%m-%d %H:%M:%S')," \
         "$(top -bn1 | grep "Cpu(s)" | awk '{print $2}')," \
         "$(free -m | awk 'NR==2{print $3}')," \
         "$(lsof | wc -l)" >> /var/log/lyrebird-performance.csv
    sleep 60
done
```

**Metrics File Format:**

```text
Timestamp,CPU%,MemoryMB,FileDescriptors
2025-11-15 10:00:00,25.5,650,234
2025-11-15 10:01:00,26.1,652,236
2025-11-15 10:02:00,24.8,651,235
```

---

## Troubleshooting Performance Issues

## High CPU Usage

**Symptoms:**

- FFmpeg processes consuming excessive CPU
- System sluggishness
- Stream dropouts

**Solutions:**

1. **Lower Sample Rate**
   ```bash
   # Change from 48kHz to 16kHz
   DEVICE_NAME_SAMPLE_RATE=16000
   ```

2. **Switch to Opus Codec**
   ```bash
   DEVICE_NAME_CODEC=opus
   ```

3. **Reduce Bitrate**
   ```bash
   DEVICE_NAME_BITRATE=64k  # Reduced from 128k
   ```

4. **Use Mono Instead of Stereo**
   ```bash
   DEVICE_NAME_CHANNELS=1
   ```

5. **Reduce Number of Streams**
   - Disable monitoring streams
   - Use stream rotation
   - Distribute across multiple servers

## Memory Exhaustion

**Symptoms:**

- Out of memory errors
- System swap usage
- Process crashes

**Solutions:**

1. **Calculate Required Memory**
   ```
   Required = (Num_Streams × 50 MB) + 200 MB base
   ```

2. **Reduce Stream Count**
   - Disable unnecessary streams
   - Upgrade system RAM

3. **Lower Quality Settings**
   ```bash
   sudo ./lyrebird-mic-check.sh -g --quality=low -f
   ```

4. **Monitor Memory Usage**
   ```bash
   watch -n 1 free -h
   ```

## File Descriptor Exhaustion

**Symptoms:**

- "Too many open files" errors
- Streams fail to start
- MediaMTX connection errors

**Solutions:**

1. **Check Current Limit**
   ```bash
   ulimit -n  # Typically 1024
   ```

2. **Increase Limit Temporarily**
   ```bash
   ulimit -n 4096
   sudo ./mediamtx-stream-manager.sh restart
   ```

3. **Increase Limit Permanently**
   ```bash
   # Edit /etc/security/limits.conf
   * soft nofile 4096
   * hard nofile 8192

   # Reboot or re-login for changes to take effect
   ```

4. **Verify New Limit**
   ```bash
   ulimit -n  # Should show 4096
   ```

## Network Bandwidth Issues

**Symptoms:**

- Choppy playback
- Stream buffering
- Client disconnections

**Solutions:**

1. **Calculate Bandwidth Requirements**
   ```
   Total Bandwidth = Num_Streams × Bitrate × Num_Clients

   Example: 10 streams × 128 kbps × 5 clients = 6.4 Mbps
   ```

2. **Reduce Bitrate**
   ```bash
   DEVICE_NAME_BITRATE=64k  # Reduced from 128k
   ```

3. **Use More Efficient Codec**
   ```bash
   DEVICE_NAME_CODEC=opus  # More efficient than AAC or MP3
   ```

4. **Implement Quality of Service (QoS)**
   - Prioritize RTSP traffic on router
   - Use dedicated network interface

---

## Best Practices

## Capacity Planning

1. **Start Small, Scale Gradually**
   - Begin with 2-3 streams
   - Monitor resource usage
   - Add streams incrementally

2. **Calculate Before Deploying**
   - Estimate CPU: streams × 3%
   - Estimate Memory: (streams × 50 MB) + 200 MB
   - Validate against hardware specs

3. **Leave Headroom**
   - Target 70-80% max CPU usage
   - Reserve 20% memory for system
   - Don't max out file descriptors

## Quality Selection

1. **Match Quality to Use Case**
   - Voice: Low quality (16kHz, 64kbps)
   - General: Normal quality (48kHz, 128kbps)
   - Music: High quality (48kHz, 192kbps)

2. **Don't Over-Provision**
   - More quality than needed wastes resources
   - 192 kbps AAC sufficient for most music
   - 64 kbps Opus excellent for voice

3. **Test and Validate**
   - Listen to streams before production
   - Verify quality meets requirements
   - Monitor for artifacts or issues

## Platform Selection

1. **Raspberry Pi**
   - Limit to 2 devices maximum
   - Use low quality preset
   - Powered USB hub required

2. **Intel N100**
   - 4-8 devices recommended
   - Mix of quality levels acceptable
   - Good balance price/performance

3. **Workstation/Server**
   - 10+ devices easily
   - High quality streams supported
   - Best for production deployments

---

## Related Documentation

- [Architecture](architecture.md) - System design and components
- [Configuration](../user-guide/configuration.md) - Quality settings
- [Stream Management](../user-guide/stream-management.md) - Resource monitoring
- [Diagnostics & Monitoring](diagnostics-monitoring.md) - Health checks
- [Troubleshooting](troubleshooting.md) - Issue resolution

---

## Next Steps

<div class="grid" markdown>

<div markdown>
## Troubleshooting
Common issues and solutions

[Troubleshooting →](troubleshooting.md)
</div>

<div markdown>
### Diagnostics & Monitoring
System health monitoring

[Diagnostics & Monitoring →](diagnostics-monitoring.md)
</div>

</div>
