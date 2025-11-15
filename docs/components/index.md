# Components Overview

LyreBirdAudio is built from specialized components that work together.

---

## Core Components

<div class="grid" markdown>

<div markdown>
### [Orchestrator](orchestrator.md)
Unified management interface for all operations
</div>

<div markdown>
### [Stream Manager](stream-manager.md)
Stream lifecycle management and automatic recovery
</div>

<div markdown>
### [USB Audio Mapper](usb-audio-mapper.md)
Device persistence via udev rules
</div>

<div markdown>
### [Capability Checker](capability-checker.md)
Hardware detection and capability testing
</div>

<div markdown>
### [Diagnostics Tool](diagnostics.md)
System health monitoring and reporting
</div>

<div markdown>
### [Version Manager](version-manager.md)
Git-based update and rollback management
</div>

<div markdown>
### [MediaMTX Installer](installer.md)
MediaMTX server installation and setup
</div>

</div>

---

## Component Interaction

```mermaid
graph LR
    A[Orchestrator] --> B[USB Mapper]
    A --> C[Stream Manager]
    A --> D[Version Manager]
    A --> E[Installer]
    C --> F[Capability Checker]
    C --> G[Diagnostics]
```

[Return to Main Documentation](../index.md)
