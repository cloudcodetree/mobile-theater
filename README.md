# 🎬 Pelican Cinema

> Military-grade portable 7.1 wireless surround sound theater system

[![Project Status](https://img.shields.io/badge/status-Phase_1-yellow)]()
[![Budget](https://img.shields.io/badge/budget-$2,655-green)]()
[![License](https://img.shields.io/badge/license-MIT-blue)]()

## Overview

A fully portable, battery-powered 7.1 surround sound theater system featuring:

- **Wireless speaker pods** with ESP32 mesh networking
- **Self-contained batteries** in each speaker (3+ hour runtime)
- **Multi-source input** (PS5, Xbox, Apple TV, Blu-ray)
- **Sub-80ms latency** end-to-end
- **Outdoor optimized** (no ceiling required)

```
┌─────────────────────────────────────────────────────────────────┐
│                      SYSTEM OVERVIEW                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│    [Sources] → [Command Module] → [Wireless] → [Speaker Pods]  │
│                                                                 │
│    • PS5           • HDMI Switch      ESP-NOW      • FL  • FR   │
│    • Xbox          • Audio Extractor  2.4GHz       • C   • SUB  │
│    • Apple TV      • Raspberry Pi 5   Mesh         • SL  • SR   │
│    • Blu-ray       • ESP32 Master TX               • RL  • RR   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Quick Stats

| Metric | Target |
|--------|--------|
| Channels | 7.1 (expandable to 7.1.4) |
| Speaker Pods | 8 (7 satellites + 1 sub) |
| Total Budget | ~$2,655 |
| Latency | <80ms |
| Runtime | 3+ hours |

## Project Phases

| Phase | Description | Budget | Status |
|-------|-------------|--------|--------|
| 1 | Wireless Proof of Concept | $45 | 🟡 Active |
| 2 | Multi-Channel Sync Validation | $55 | ⚪ Ready |
| 3 | Command Module Build | $620 | ⚪ Pending |
| 4 | Prototype Speaker Pod | $175 | ⚪ Pending |
| 5 | Full Production (7 pods) | $1,360 | ⚪ Pending |
| 6 | Case Integration | $350 | ⚪ Pending |
| 7 | Calibration & Testing | $50 | ⚪ Pending |

## Repository Structure

```
mobile-theater/
├── docs/
│   ├── phases/          # Phase milestone documents
│   ├── deliverables/    # Individual work items
│   ├── specs/           # Component specifications
│   └── tests/           # Test reports
├── bom/                 # Bills of materials
├── builds/              # Build logs per unit
├── firmware/            # ESP32 source code
│   ├── src/
│   └── lib/
├── hardware/            # Schematics, PCB designs
│   ├── schematics/
│   └── pcb/
├── cad/                 # Enclosure designs
│   ├── enclosures/
│   └── mounts/
├── templates/           # Document templates
├── resources/           # Reference materials
└── diagrams/            # System diagrams
```

## Getting Started

### Phase 1: Wireless Proof of Concept

1. Order parts from [Phase 1 BOM](bom/BOM-phase-1.md)
2. Set up ESP32 development environment
3. Flash TX/RX firmware
4. Test audio streaming latency

See [D-1.1: Dev Environment Setup](docs/deliverables/D-1.1-dev-environment.md) for details.

## Documentation

- [Master Wiki](docs/WIKI.md) — Central documentation hub
- [Video Tutorials](resources/VIDEO_TUTORIALS.md) — Curated learning resources
- [Chat Log](docs/CHAT_LOG.md) — Project discussion history

## Hardware

### Speaker Pod Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│  ESP32-S3 RX → I2S → PCM5102A DAC → TPA3116 Amp → 4" Driver    │
│      ▲                                                          │
│      └── 3S LiFePO4 Battery (3+ hrs) + BMS + Buck Converter    │
└─────────────────────────────────────────────────────────────────┘
```

### Command Module

```
[HDMI Sources] → [4×1 Switch] → [Audio Extractor] → [Pi 5] → [ESP32 TX]
                      │
                      └──→ [4K Projector]
```

## License

MIT License - See [LICENSE](LICENSE) for details.

## Contributing

This is a personal build project. Issues and suggestions welcome!
