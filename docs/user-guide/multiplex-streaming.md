# Multiplex Streaming

Advanced guide to combining multiple audio sources into single streams using FFmpeg audio filters.

---

## Overview

Multiplex streaming allows you to combine multiple USB microphones into a single RTSP stream. This is useful for applications requiring synchronized multi-microphone capture, such as:

- Office environments with multiple microphones
- Meeting rooms with distributed audio pickup
- Bird monitoring stations with microphone arrays
- Audio surveillance with multiple capture points

This guide covers:

- Multiplex modes (amix vs amerge)
- Creating multiplex streams
- Channel separation and mixing
- Use cases and examples
- Troubleshooting multiplex configurations

---

## Multiplex Modes

LyreBirdAudio supports two primary multiplex modes using FFmpeg audio filters:

### 1. amix (Audio Mixing)

**Filter:** `amix`

**Purpose:** Combines all microphone inputs into a single mixed audio stream.

**Behavior:**

- All audio sources are summed together
- Output has the same number of channels as inputs (typically mono or stereo)
- All microphones contribute to the same audio output
- Cannot distinguish between individual sources in playback

**Best For:**

- General ambient monitoring
- Office background audio
- Situations where individual source identification isn't needed
- Reducing bandwidth (single stream vs multiple streams)

**Example Use Case:**

Office with 4 microphones capturing general ambient sound for presence detection.

### 2. amerge (Audio Merging)

**Filter:** `amerge`

**Purpose:** Merges inputs while preserving each microphone as a separate channel.

**Behavior:**

- Each input becomes a distinct output channel
- Channel count equals number of inputs
- Individual sources remain separate
- Requires multi-channel capable playback

**Best For:**

- Multi-track recording
- Post-processing separation
- Identifying individual sound sources
- Professional audio analysis

**Example Use Case:**

Bird monitoring with 4 microphones positioned around a habitat, maintaining spatial information for analysis.

---

## Creating Multiplex Streams

### Basic Multiplex Command

**Using amix (mixed output):**

```bash
sudo ./mediamtx-stream-manager.sh start -m multiplex -f amix
```

**Using amerge (separate channels):**

```bash
sudo ./mediamtx-stream-manager.sh start -m multiplex -f amerge
```

### Custom Stream Naming

Specify a custom name for the multiplex stream:

```bash
sudo ./mediamtx-stream-manager.sh start -m multiplex -f amix -n Office_Audio
```

This creates a stream accessible at:

```
rtsp://hostname:8554/Office_Audio
```

### Selecting Specific Devices

To multiplex only specific devices (not all configured devices):

```bash
# Edit /etc/mediamtx/audio-devices.conf to include only desired devices
# Then start multiplex stream
sudo ./mediamtx-stream-manager.sh start -m multiplex -f amix
```

---

## amix: Mixed Audio Streams

### How amix Works

The `amix` filter combines multiple audio inputs into a single output by summing the audio samples:

**Input:**

- Microphone 1: Bird chirping
- Microphone 2: Wind noise
- Microphone 3: Distant traffic

**Output:**

- Single stream containing all three sounds mixed together

### Configuration Example

**Creating a mixed stream from 3 microphones:**

```bash
sudo ./mediamtx-stream-manager.sh start -m multiplex -f amix -n Garden_Audio
```

**FFmpeg command generated (conceptual):**

```bash
ffmpeg \
  -f alsa -i hw:CARD=Mic1 \
  -f alsa -i hw:CARD=Mic2 \
  -f alsa -i hw:CARD=Mic3 \
  -filter_complex "[0:a][1:a][2:a]amix=inputs=3:duration=longest" \
  -c:a opus -b:a 128k \
  -f rtsp rtsp://localhost:8554/Garden_Audio
```

### amix Parameters

**Filter syntax:**

```
amix=inputs=N:duration=MODE:dropout_transition=TIME
```

| Parameter | Description | Options |
|-----------|-------------|---------|
| `inputs` | Number of input streams | Integer (e.g., 3, 4, 5) |
| `duration` | Output duration mode | longest, shortest, first |
| `dropout_transition` | Fade time when input ends | Seconds (e.g., 2, 5) |

**Common configurations:**

```bash
# Standard mixing (3 inputs)
amix=inputs=3:duration=longest

# With dropout transition
amix=inputs=4:duration=longest:dropout_transition=2
```

### Volume Balancing

When mixing multiple sources, you may need to adjust relative volumes:

```bash
# Equal mix of 3 inputs
-filter_complex "[0:a][1:a][2:a]amix=inputs=3:duration=longest"

# Weighted mix (first input louder)
-filter_complex "[0:a]volume=1.5[a1];[1:a][2:a][a1]amix=inputs=3"
```

### Use Case: Office Monitoring

**Scenario:** Monitor general office ambient audio with 4 microphones

**Configuration:**

```bash
# Create mixed stream
sudo ./mediamtx-stream-manager.sh start -m multiplex -f amix -n Office_Ambient
```

**Result:**

- Single RTSP stream: `rtsp://server:8554/Office_Ambient`
- All 4 microphones contribute to one audio output
- Bandwidth: ~16 KB/s (single stream instead of 4 × 16 KB/s)
- Use case: General presence detection, background noise monitoring

---

## amerge: Channel-Separated Streams

### How amerge Works

The `amerge` filter combines inputs while keeping them as separate channels:

**Input:**

- Microphone 1 (mono): Ch 0
- Microphone 2 (mono): Ch 1
- Microphone 3 (mono): Ch 2

**Output:**

- Multi-channel stream with 3 channels
- Channel 0: Microphone 1
- Channel 1: Microphone 2
- Channel 2: Microphone 3

### Configuration Example

**Creating a merged stream from 3 microphones:**

```bash
sudo ./mediamtx-stream-manager.sh start -m multiplex -f amerge -n Bird_Array
```

**FFmpeg command generated (conceptual):**

```bash
ffmpeg \
  -f alsa -i hw:CARD=Mic1 \
  -f alsa -i hw:CARD=Mic2 \
  -f alsa -i hw:CARD=Mic3 \
  -filter_complex "[0:a][1:a][2:a]amerge=inputs=3[out]" \
  -map "[out]" \
  -c:a opus -b:a 256k \
  -f rtsp rtsp://localhost:8554/Bird_Array
```

### Channel Mapping

Each input microphone is mapped to a specific channel:

| Microphone | Channel Index | Description |
|------------|---------------|-------------|
| First device | 0 | Left-most channel |
| Second device | 1 | Second channel |
| Third device | 2 | Third channel |
| Nth device | N-1 | Nth channel |

### Extracting Individual Channels

**During playback, extract specific channels:**

```bash
# Extract channel 0 (first microphone)
ffplay -af "pan=mono|c0=c0" rtsp://localhost:8554/Bird_Array

# Extract channel 1 (second microphone)
ffplay -af "pan=mono|c0=c1" rtsp://localhost:8554/Bird_Array

# Extract channel 2 (third microphone)
ffplay -af "pan=mono|c0=c2" rtsp://localhost:8554/Bird_Array
```

**Recording to separate files:**

```bash
# Record all channels
ffmpeg -i rtsp://localhost:8554/Bird_Array -c copy recording.mkv

# Split into separate files
ffmpeg -i rtsp://localhost:8554/Bird_Array \
  -map 0:a -af "pan=mono|c0=c0" mic1.opus \
  -map 0:a -af "pan=mono|c0=c1" mic2.opus \
  -map 0:a -af "pan=mono|c0=c2" mic3.opus
```

### Bitrate Considerations

Multi-channel streams require higher bitrates:

**Bitrate calculation:**

```
Total Bitrate = Base Bitrate × Number of Channels
```

**Example:**

```
Single channel: 128 kbps
3 channels: 128 × 3 = 384 kbps (or use 256-320 kbps with compression)
```

**Recommended bitrates for amerge:**

| Channels | Minimum | Recommended | High Quality |
|----------|---------|-------------|--------------|
| 2 | 128 kbps | 192 kbps | 256 kbps |
| 3 | 192 kbps | 256 kbps | 384 kbps |
| 4 | 256 kbps | 320 kbps | 512 kbps |
| 5+ | 320 kbps | 384 kbps | 640 kbps |

### Use Case: Bird Monitoring Array

**Scenario:** Record birds from 4 directional microphones for spatial analysis

**Configuration:**

```bash
# Create channel-separated stream
sudo ./mediamtx-stream-manager.sh start -m multiplex -f amerge -n Bird_Array
```

**Result:**

- Single RTSP stream: `rtsp://server:8554/Bird_Array`
- 4 separate audio channels preserved
- Can analyze which direction bird calls originated
- Post-processing can separate individual sound sources
- Bandwidth: ~32 KB/s (4 channels at 256 kbps total)

---

## Comparison: amix vs amerge

### Feature Comparison

| Feature | amix | amerge |
|---------|------|--------|
| **Output Channels** | Same as input (typically mono) | Sum of all inputs |
| **Source Separation** | No | Yes |
| **Bitrate** | Lower (single channel) | Higher (multiple channels) |
| **Bandwidth** | ~16 KB/s (128 kbps) | ~32-64 KB/s (256-512 kbps) |
| **Post-Processing** | Limited | Full separation |
| **Use Case** | General monitoring | Professional analysis |
| **Complexity** | Simple | Moderate |
| **Client Compatibility** | Universal | Multi-channel support required |

### When to Use Each

**Use amix when:**

- General ambient monitoring is sufficient
- Individual source identification not needed
- Bandwidth is limited
- Simple playback required
- Detecting presence/activity

**Use amerge when:**

- Need to identify individual sources
- Post-processing/analysis required
- Spatial information is important
- Professional recording quality needed
- Client supports multi-channel audio

---

## Advanced Configurations

### Custom Filter Chains

For advanced users, combine multiple filters:

**amix with normalization:**

```bash
-filter_complex "[0:a][1:a][2:a]amix=inputs=3:duration=longest,volume=0.8[out]"
```

**amerge with per-channel processing:**

```bash
-filter_complex "[0:a]volume=1.2[a0];[1:a]volume=1.0[a1];[2:a]volume=0.8[a2]; \
                 [a0][a1][a2]amerge=inputs=3[out]"
```

### Synchronization

Multiplex streams automatically synchronize inputs, but you can fine-tune:

**Add delay to compensate for mic positioning:**

```bash
-filter_complex "[0:a]adelay=0[a0];[1:a]adelay=100[a1];[2:a]adelay=200[a2]; \
                 [a0][a1][a2]amix=inputs=3[out]"
```

Delays in milliseconds compensate for physical distance between microphones.

### Sample Rate Matching

All inputs must have matching sample rates. LyreBirdAudio handles this automatically by:

1. Reading device configurations from `/etc/mediamtx/audio-devices.conf`
2. Resampling if necessary to match common rate
3. Applying filters after resampling

**To ensure consistent quality:**

```bash
# Set all devices to same sample rate
DEVICE_MIC1_SAMPLE_RATE=48000
DEVICE_MIC2_SAMPLE_RATE=48000
DEVICE_MIC3_SAMPLE_RATE=48000
```

---

## Accessing Multiplex Streams

### Stream URL

Multiplex streams are accessed like any other RTSP stream:

```
rtsp://hostname:8554/stream-name
```

**Examples:**

```bash
# Default multiplex stream
rtsp://localhost:8554/multiplex

# Custom named stream
rtsp://localhost:8554/Office_Audio
rtsp://localhost:8554/Bird_Array
```

### Playback

**VLC (automatic multi-channel support):**

```bash
vlc rtsp://localhost:8554/Bird_Array
```

**FFplay (specify channels):**

```bash
# Play all channels
ffplay rtsp://localhost:8554/Bird_Array

# Play specific channel
ffplay -af "pan=mono|c0=c1" rtsp://localhost:8554/Bird_Array
```

**Audacity (multi-track recording):**

1. File -> Import -> Audio
2. Enter URL: `rtsp://localhost:8554/Bird_Array`
3. Each channel appears as separate track

---

## Monitoring Multiplex Streams

### Check Stream Status

```bash
# List all streams including multiplex
sudo ./mediamtx-stream-manager.sh list
```

**Example output:**

```
Available RTSP Streams:
=======================
Blue_Yeti - rtsp://localhost:8554/Blue_Yeti
USB_Microphone - rtsp://localhost:8554/USB_Microphone
Office_Audio - rtsp://localhost:8554/Office_Audio (multiplex: 4 channels)

Total: 3 streams
```

### Verify Channel Count

Query MediaMTX API for stream details:

```bash
curl http://localhost:9997/v3/paths/get/Office_Audio
```

Look for `tracks` field indicating number of audio channels.

### Resource Usage

**Multiplex streams use more resources:**

- **CPU:** 2-3× single stream (filter processing)
- **Memory:** 1.5-2× single stream
- **Bandwidth:** Depends on channel count and bitrate

**Monitor resources:**

```bash
sudo ./mediamtx-stream-manager.sh monitor
```

---

## Troubleshooting

### Multiplex Stream Won't Start

**Symptom:** Multiplex stream fails to initialize

**Solutions:**

1. **Verify all devices are available:**
   ```bash
   cat /proc/asound/cards
   ```

2. **Check device configurations match:**
   ```bash
   # Ensure consistent sample rates
   grep SAMPLE_RATE /etc/mediamtx/audio-devices.conf
   ```

3. **Test devices individually first:**
   ```bash
   sudo ./mediamtx-stream-manager.sh start
   # Then try multiplex
   ```

4. **Check FFmpeg filter syntax:**
   ```bash
   sudo journalctl -u mediamtx-stream-manager -n 50
   ```

### Audio Out of Sync

**Symptom:** Different microphone inputs not synchronized

**Solutions:**

1. **Ensure matching sample rates:**
   ```bash
   # Set all to same rate in audio-devices.conf
   DEVICE_*_SAMPLE_RATE=48000
   ```

2. **Add buffer time:**
   ```bash
   DEVICE_*_THREAD_QUEUE=2048
   ```

3. **Check USB hub quality:**
   - Use powered USB hub
   - Avoid daisy-chaining hubs
   - Connect to same USB controller when possible

### High CPU Usage

**Symptom:** FFmpeg using excessive CPU for multiplex

**Solutions:**

1. **Reduce number of inputs:**
   - Limit to 4-6 microphones maximum
   - Split into multiple multiplex streams

2. **Lower sample rate:**
   ```bash
   DEVICE_*_SAMPLE_RATE=16000  # For voice monitoring
   ```

3. **Use more efficient codec:**
   ```bash
   DEVICE_*_CODEC=opus
   ```

4. **Reduce bitrate:**
   ```bash
   # For amix
   DEVICE_*_BITRATE=96k

   # For amerge (total for all channels)
   DEVICE_*_BITRATE=256k
   ```

### Cannot Hear All Channels

**Symptom:** Only some microphones audible in amix output

**Solutions:**

1. **Check microphone levels:**
   ```bash
   alsamixer
   # Verify all inputs have similar levels
   ```

2. **Add volume normalization:**
   ```bash
   # Per-channel volume adjustment in filter
   -filter_complex "[0:a]volume=1.5[a0];[1:a]volume=1.0[a1]; \
                    [a0][a1]amix=inputs=2"
   ```

3. **Verify all devices streaming:**
   ```bash
   ps aux | grep ffmpeg
   # Check all expected FFmpeg processes running
   ```

### Channel Separation Lost (amerge)

**Symptom:** Cannot distinguish channels in amerge stream

**Solutions:**

1. **Verify client supports multi-channel:**
   - Use VLC or Audacity
   - Check audio device configuration

2. **Explicitly specify channels in playback:**
   ```bash
   ffplay -af "pan=mono|c0=c2" rtsp://localhost:8554/stream
   ```

3. **Check stream metadata:**
   ```bash
   ffprobe rtsp://localhost:8554/stream
   # Verify channel count matches expectation
   ```

---

## Best Practices

### Planning Multiplex Deployments

1. **Start with individual streams** - Verify each device works alone first
2. **Match configurations** - Use identical sample rates and formats
3. **Calculate bandwidth** - Plan for increased bandwidth with amerge
4. **Test under load** - Verify performance with all devices active
5. **Monitor resources** - Watch CPU and memory during operation

### Performance Optimization

1. **Limit input count** - Use 4-6 microphones maximum per multiplex stream
2. **Use consistent hardware** - Same microphone models when possible
3. **Optimize sample rates** - 48000 Hz for most applications, 16000 Hz for voice
4. **Right-size bitrate** - Balance quality vs bandwidth based on use case
5. **Monitor continuously** - Track resource usage over time

### Audio Quality

1. **Match microphone sensitivity** - Use similar microphones for consistent levels
2. **Physical positioning** - Space microphones appropriately for coverage
3. **Test acoustics** - Verify no echo or interference between microphones
4. **Adjust individual levels** - Use volume filters to balance inputs
5. **Record test samples** - Verify quality before production deployment

### Reliability

1. **Use powered USB hubs** - Prevent power issues with multiple devices
2. **Implement monitoring** - Track multiplex stream health
3. **Plan for failures** - Individual mic failure shouldn't break entire stream
4. **Document configuration** - Keep records of device assignments and filter settings
5. **Regular testing** - Periodically verify all channels working correctly

---

## Use Case Examples

### Example 1: Office Ambient Monitoring (amix)

**Requirement:** Monitor general office noise with 4 ceiling-mounted microphones

**Configuration:**

```bash
# In /etc/mediamtx/audio-devices.conf
DEVICE_OFFICE_NE_SAMPLE_RATE=16000
DEVICE_OFFICE_NE_CODEC=opus
DEVICE_OFFICE_NE_BITRATE=64k

DEVICE_OFFICE_NW_SAMPLE_RATE=16000
DEVICE_OFFICE_NW_CODEC=opus
DEVICE_OFFICE_NW_BITRATE=64k

DEVICE_OFFICE_SE_SAMPLE_RATE=16000
DEVICE_OFFICE_SE_CODEC=opus
DEVICE_OFFICE_SE_BITRATE=64k

DEVICE_OFFICE_SW_SAMPLE_RATE=16000
DEVICE_OFFICE_SW_CODEC=opus
DEVICE_OFFICE_SW_BITRATE=64k
```

**Create stream:**

```bash
sudo ./mediamtx-stream-manager.sh start -m multiplex -f amix -n Office_Ambient
```

**Result:**

- Stream: `rtsp://server:8554/Office_Ambient`
- Bandwidth: ~8 KB/s
- Purpose: Activity detection, presence monitoring

### Example 2: Bird Habitat Monitoring (amerge)

**Requirement:** Record bird calls from 3 directional microphones for species identification

**Configuration:**

```bash
# In /etc/mediamtx/audio-devices.conf
DEVICE_BIRD_NORTH_SAMPLE_RATE=48000
DEVICE_BIRD_NORTH_CODEC=opus
DEVICE_BIRD_NORTH_BITRATE=128k

DEVICE_BIRD_EAST_SAMPLE_RATE=48000
DEVICE_BIRD_EAST_CODEC=opus
DEVICE_BIRD_EAST_BITRATE=128k

DEVICE_BIRD_WEST_SAMPLE_RATE=48000
DEVICE_BIRD_WEST_CODEC=opus
DEVICE_BIRD_WEST_BITRATE=128k
```

**Create stream:**

```bash
sudo ./mediamtx-stream-manager.sh start -m multiplex -f amerge -n Bird_Habitat
```

**Result:**

- Stream: `rtsp://server:8554/Bird_Habitat`
- Channels: 3 (separate)
- Bandwidth: ~32 KB/s
- Purpose: Species identification, directional analysis

### Example 3: Meeting Room (amix with custom volumes)

**Requirement:** Capture meeting audio with 2 table microphones and 1 presenter microphone

**Advanced configuration** (requires custom FFmpeg integration):

```bash
# Presenter mic louder, table mics quieter
-filter_complex "[0:a]volume=1.5[presenter]; \
                 [1:a]volume=0.8[table1]; \
                 [2:a]volume=0.8[table2]; \
                 [presenter][table1][table2]amix=inputs=3[out]"
```

**Result:**

- Balanced audio with emphasis on presenter
- Single stream for simple recording/playback

---

## Related Documentation

- [Stream Management](stream-management.md) - Managing stream lifecycle
- [Configuration](configuration.md) - Audio encoding settings
- [MediaMTX Integration](mediamtx-integration.md) - RTSP server configuration
- [USB Device Management](usb-device-management.md) - Device detection and setup
- [Performance Tuning](../advanced/performance.md) - Optimization strategies

---

## Next Steps

<div class="grid" markdown>

<div markdown>
### Stream Management
Managing stream lifecycle and health

[Stream Management →](stream-management.md)
</div>

<div markdown>
### MediaMTX Integration
RTSP server configuration

[MediaMTX Integration →](mediamtx-integration.md)
</div>

</div>
