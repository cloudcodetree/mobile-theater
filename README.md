# 🎬 Pelican Cinema

> Military-grade portable 7.1 wireless surround sound theater system

[![Status](https://img.shields.io/badge/status-Phase_1-yellow)]()
[![Budget](https://img.shields.io/badge/budget-$2,655-green)]()
[![License](https://img.shields.io/badge/license-MIT-blue)]()

## Overview

A fully portable, battery-powered 7.1 surround sound theater featuring wireless ESP32 speaker pods, self-contained batteries, and sub-80ms latency.

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

## Project Structure

```
mobile-theater/
├── .github/           # Issue templates, workflows
├── docs/
│   ├── phases/        # Phase milestone docs
│   ├── deliverables/  # Individual work items
│   ├── specs/         # Component specifications
│   └── tests/         # Test reports
├── bom/               # Bills of materials
├── builds/            # Build logs per unit
├── firmware/
│   ├── tx/            # Master transmitter code
│   └── rx/            # Satellite receiver code
├── hardware/          # Schematics, PCB designs
├── cad/               # Enclosure designs
├── templates/         # Document templates
├── resources/         # Reference materials
└── diagrams/          # System diagrams
```

## Phases

| Phase | Description | Budget | Status |
|-------|-------------|--------|--------|
| 1 | Wireless Proof of Concept | $45 | 🟡 Active |
| 2 | Multi-Channel Sync Validation | $55 | ⚪ Ready |
| 3 | Command Module Build | $620 | ⚪ Pending |
| 4 | Prototype Speaker Pod | $175 | ⚪ Pending |
| 5 | Full Production (7 pods) | $1,360 | ⚪ Pending |
| 6 | Case Integration | $350 | ⚪ Pending |
| 7 | Calibration & Testing | $50 | ⚪ Pending |

## Getting Started

1. Review [Phase 1 Documentation](docs/phases/PHASE-1.md)
2. Order parts from [Phase 1 BOM](bom/BOM-phase-1.md)
3. Follow deliverables in order (D-1.1 → D-1.2 → D-1.3)

## Speaker Pod Architecture

Each wireless speaker pod contains:

```
┌─────────────────────────────────────────────────────────────────┐
│  [ESP32-S3] → [I2S] → [PCM5102A DAC] → [TPA3116 Amp] → [Driver] │
│       ▲                                                         │
│       └── [3S LiFePO4 9.6V] → [BMS] → [5V Buck]                │
└─────────────────────────────────────────────────────────────────┘
```

## License

MIT License - See [LICENSE](LICENSE) for details.
