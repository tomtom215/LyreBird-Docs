# System Overview

High-level overview of LyreBirdAudio architecture.

---

## System Architecture

LyreBirdAudio transforms USB microphones into reliable RTSP streams through a layered architecture:

1. **Hardware Layer** - USB microphones
2. **Device Layer** - udev persistent device naming
3. **Processing Layer** - FFmpeg audio capture and encoding
4. **Streaming Layer** - MediaMTX RTSP server
5. **Management Layer** - Stream lifecycle management
6. **Client Layer** - RTSP client applications

---

## Key Components

See the [Components Overview](../components/index.md) for detailed information.

[Architecture Details](../advanced/architecture.md) | [Main Documentation](../index.md)

---

## Get Started

Learn more about the system:

[Architecture Guide →](../advanced/architecture.md){ .md-button .md-button--primary }
[Quick Start →](../getting-started/quick-start.md){ .md-button }
