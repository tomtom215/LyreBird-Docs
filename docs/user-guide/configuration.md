# Configuration

Comprehensive guide to configuring LyreBirdAudio streams for optimal performance.

---

## Overview

LyreBirdAudio provides flexible configuration options for audio encoding and streaming. This guide covers:

- Audio device configuration
- Codec selection (Opus, AAC, MP3, PCM)
- Sample rate and channel configuration
- Bitrate optimization
- Quality tiers and recommendations
- Configuration file structure

---

## Configuration Files

## Audio Devices Configuration

**Location:** `/etc/mediamtx/audio-devices.conf`

This file contains device-specific audio settings using a dual-lookup system that supports both friendly names and full device IDs for guaranteed uniqueness.

### Configuration Format

Parameters are defined using the pattern:

```bash
DEVICE_<FRIENDLY_NAME>_<PARAMETER>=value
```

#### Supported Parameters

| Parameter | Description | Typical Values | Default |
|-----------|-------------|----------------|---------|
| `SAMPLE_RATE` | Audio frequency in Hz | 48000, 44100, 16000 | 48000 |
| `CHANNELS` | Audio channel count | 1 (mono), 2 (stereo) | 1 |
| `CODEC` | Audio encoding format | opus, aac, mp3, pcm | opus |
| `BITRATE` | Encoding bitrate | 64k, 128k, 256k | 128k |
| `THREAD_QUEUE` | FFmpeg buffer size | 1024, 8192, 16384 | 8192 |

#### Example Configuration

```bash
# Blue Yeti microphone settings
DEVICE_BLUE_YETI_SAMPLE_RATE=44100
DEVICE_BLUE_YETI_CHANNELS=2
DEVICE_BLUE_YETI_CODEC=aac
DEVICE_BLUE_YETI_BITRATE=192k
DEVICE_BLUE_YETI_THREAD_QUEUE=8192

# USB microphone with full device ID for uniqueness
DEVICE_usb_046d_0825_12345678_SAMPLE_RATE=48000
DEVICE_usb_046d_0825_12345678_CHANNELS=1
DEVICE_usb_046d_0825_12345678_CODEC=opus
DEVICE_usb_046d_0825_12345678_BITRATE=128k
```

---

## Automatic Configuration Generation

### Using the Capability Checker

The easiest way to generate optimal configuration is using the built-in capability checker:

```bash
sudo ./lyrebird-mic-check.sh -g --quality=normal
```

This automatically:

1. Detects all USB audio devices
2. Tests hardware capabilities
3. Generates optimal settings for each device
4. Writes configuration to `/etc/mediamtx/audio-devices.conf`

## Quality Tiers

LyreBirdAudio supports three predefined quality tiers:

### Low Quality (Speech/Monitoring)

```bash
sudo ./lyrebird-mic-check.sh -g --quality=low
```

- **Sample Rate:** 16000 Hz
- **Bitrate:** 64 kbps
- **Use Cases:** Voice monitoring, speech detection, bandwidth-constrained environments
- **Bandwidth:** ~8 KB/s per stream

#### Normal Quality (Balanced - Default)

```bash
sudo ./lyrebird-mic-check.sh -g --quality=normal
```

- **Sample Rate:** 48000 Hz
- **Bitrate:** 128 kbps
- **Use Cases:** General audio monitoring, bird calls, ambient sound
- **Bandwidth:** ~16 KB/s per stream

#### High Quality (Music/Professional)

```bash
sudo ./lyrebird-mic-check.sh -g --quality=high
```

- **Sample Rate:** 48000 Hz (or higher if supported)
- **Bitrate:** 256+ kbps
- **Use Cases:** Music recording, professional audio, high-fidelity capture
- **Bandwidth:** ~32 KB/s per stream

## Force Configuration Override

To force configuration generation without interactive prompts:

```bash
sudo ./lyrebird-mic-check.sh -g -f --quality=normal
```

The `-f` flag overwrites existing configuration without confirmation.

---

## Codec Selection

### Opus (Recommended - Default)

**Codec:** `opus`

**Advantages:**

- Superior quality at low bitrates
- Optimized for both speech and music
- Low latency
- Widely supported

**Best For:**

- General-purpose audio streaming
- Network streaming where bandwidth varies
- Real-time monitoring

**Configuration:**

```bash
DEVICE_MICROPHONE_CODEC=opus
DEVICE_MICROPHONE_BITRATE=128k
```

### AAC (High Compatibility)

**Codec:** `aac`

**Advantages:**

- Excellent compatibility with Apple devices
- Good quality at medium bitrates
- Wide software support

**Best For:**

- iOS/macOS client playback
- Recording for archival
- Maximum compatibility

**Configuration:**

```bash
DEVICE_MICROPHONE_CODEC=aac
DEVICE_MICROPHONE_BITRATE=192k
```

### MP3 (Legacy Support)

**Codec:** `mp3`

**Advantages:**

- Universal compatibility
- Proven reliability
- Supported by all playback devices

**Best For:**

- Legacy systems
- Maximum compatibility requirements
- Simple archival

**Configuration:**

```bash
DEVICE_MICROPHONE_CODEC=mp3
DEVICE_MICROPHONE_BITRATE=192k
```

### PCM (Uncompressed)

**Codec:** `pcm`

**Advantages:**

- No quality loss
- Zero encoding latency
- Professional recording quality

**Disadvantages:**

- Very high bandwidth usage
- Large file sizes

**Best For:**

- Professional audio capture
- Post-processing workflows
- LAN-only deployments

**Configuration:**

```bash
DEVICE_MICROPHONE_CODEC=pcm
# No bitrate needed for PCM
```

!!! warning "PCM Bandwidth Requirements"
    PCM audio uses approximately 1.4 Mbps per channel at 48 kHz sample rate. Ensure adequate network bandwidth before using uncompressed audio.

---

## Sample Rate Configuration

## Common Sample Rates

| Sample Rate | Use Case | Quality Level |
|-------------|----------|---------------|
| 8000 Hz | Telephony, minimal voice | Very Low |
| 16000 Hz | Speech recognition, monitoring | Low |
| 22050 Hz | AM radio quality | Medium-Low |
| 44100 Hz | CD quality audio | High |
| 48000 Hz | Professional audio (default) | High |
| 96000 Hz | High-resolution audio | Very High |

## Selecting Sample Rate

**General Recommendation:** Use 48000 Hz for most applications.

**Lower sample rates** (16000 Hz):

- Use when bandwidth is limited
- Adequate for speech/voice monitoring
- Reduces CPU usage
- Smaller file sizes

**Higher sample rates** (96000 Hz):

- Only if hardware supports it
- For professional music recording
- Increases bandwidth and CPU requirements

**Configuration:**

```bash
DEVICE_MICROPHONE_SAMPLE_RATE=48000
```

## Verifying Hardware Capabilities

Check what sample rates your device supports:

```bash
sudo ./lyrebird-mic-check.sh -V
```

This validates your configuration against actual hardware capabilities.

---

## Bitrate Optimization

### Bitrate vs. Quality vs. Bandwidth

Higher bitrates provide better audio quality but consume more bandwidth and CPU.

| Codec | Low (kbps) | Normal (kbps) | High (kbps) |
|-------|------------|---------------|-------------|
| Opus | 32-64 | 96-128 | 192-256 |
| AAC | 64-96 | 128-192 | 256-320 |
| MP3 | 96-128 | 192 | 256-320 |

## Bitrate Selection Guidelines

**64 kbps:**

- Speech monitoring
- Low bandwidth environments
- Multiple concurrent streams on limited network

**128 kbps (Recommended):**

- Balanced quality and bandwidth
- General audio monitoring
- Bird call recording
- Most use cases

**192-256 kbps:**

- High-quality music
- Professional recording
- Archival quality

**Configuration:**

```bash
DEVICE_MICROPHONE_BITRATE=128k
```

---

## Channel Configuration

### Mono vs. Stereo

#### Mono (1 Channel)

**Configuration:**

```bash
DEVICE_MICROPHONE_CHANNELS=1
```

**Advantages:**

- Lower bandwidth (half of stereo)
- Smaller file sizes
- Adequate for single-microphone capture

**Best For:**

- Single USB microphones
- Speech monitoring
- Point-source audio capture

#### Stereo (2 Channels)

**Configuration:**

```bash
DEVICE_MICROPHONE_CHANNELS=2
```

**Advantages:**

- Spatial audio information
- Left/right channel separation
- Professional recording quality

**Best For:**

- Stereo microphones
- Ambient environment capture
- Music recording

!!! note "USB Audio Adapters"
    For USB audio adapters with 3.5mm inputs, detected capabilities reflect the USB chip, NOT the connected microphone. Always verify physical connection and channel configuration.

---

## Thread Queue Size

The `THREAD_QUEUE` parameter configures FFmpeg's internal buffer for audio processing.

## Default Settings

```bash
# Default is 8192 if not specified
DEVICE_MICROPHONE_THREAD_QUEUE=8192
```

## When to Adjust

**Increase (16384, 32768):**

- Experiencing audio dropouts
- High CPU load environments
- Multiple concurrent streams
- USB devices with stability issues

**Decrease (512):**

- Minimize latency requirements
- Low-resource systems (Raspberry Pi)
- Stable USB connections

**Configuration:**

```bash
DEVICE_MICROPHONE_THREAD_QUEUE=2048
```

---

## Configuration Validation

## Validate Configuration

After making changes, validate your configuration:

```bash
sudo ./lyrebird-mic-check.sh -V
```

This checks:

- Configuration file syntax
- Hardware capability compatibility
- Sample rate support
- Channel configuration validity

## Test Configuration

Restart streams with new configuration:

```bash
sudo ./mediamtx-stream-manager.sh restart
```

## Monitor Stream Health

```bash
sudo ./mediamtx-stream-manager.sh monitor
```

Watch for errors or warnings related to configuration issues.

---

## Environment Variable Overrides

Override default configuration paths and parameters using environment variables:

## Common Overrides

```bash
# Override startup delay
STREAM_STARTUP_DELAY=20 ./mediamtx-stream-manager.sh start

# Override USB stabilization time
USB_STABILIZATION_DELAY=10 ./mediamtx-stream-manager.sh start

# Override max restart delay
MAX_RESTART_DELAY=600 ./mediamtx-stream-manager.sh start
```

## Timing Controls

| Variable | Default | Description |
|----------|---------|-------------|
| `STREAM_STARTUP_DELAY` | 10 seconds | Wait before stream validation |
| `USB_STABILIZATION_DELAY` | 5 seconds | USB device initialization time |
| `MAX_RESTART_DELAY` | 300 seconds | Maximum exponential backoff |

## Resource Thresholds

| Variable | Default | Description |
|----------|---------|-------------|
| `MAX_FD_WARNING` | 500 | File descriptor warning level |
| `MAX_CPU_WARNING` | 20% | CPU usage warning threshold |

---

## Advanced Configuration

## Custom FFmpeg Options

For advanced users, FFmpeg pipeline customization may be available through the stream manager. Consult the [Custom Integration](../advanced/custom-integration.md) guide for details.

## MediaMTX Server Settings

MediaMTX server configuration is stored in `/etc/mediamtx/mediamtx.yml`. See [MediaMTX Integration](mediamtx-integration.md) for details.

---

## Best Practices

## Production Deployments

1. **Use automatic configuration generation** for initial setup
2. **Test configuration thoroughly** before production deployment
3. **Monitor resource usage** after configuration changes
4. **Document custom settings** for future reference
5. **Backup configuration files** before making changes

## Performance Optimization

1. **Start with normal quality tier** and adjust as needed
2. **Use Opus codec** for best quality/bandwidth ratio
3. **Avoid PCM** unless specifically required
4. **Match sample rates** to actual hardware capabilities
5. **Monitor CPU and bandwidth** under full load

## Troubleshooting Configuration Issues

1. **Validate after every change** using `-V` flag
2. **Check logs** for configuration-related errors
3. **Test one device at a time** when troubleshooting
4. **Revert to generated configuration** if custom settings fail

---

## Related Documentation

- [USB Device Management](usb-device-management.md) - Device detection and mapping
- [Stream Management](stream-management.md) - Starting and managing streams
- [Performance Tuning](../advanced/performance.md) - Optimization strategies
- [Command Reference](../reference/command-reference.md) - All available commands

---

## Next Steps

<div class="grid" markdown>

<div markdown>
## USB Device Management
Learn about device detection and mapping

[USB Device Management →](usb-device-management.md)
</div>

<div markdown>
## Stream Management
Starting and managing streams

[Stream Management →](stream-management.md)
</div>

</div>
