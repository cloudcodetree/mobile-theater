# Implementation Guide: D-3.5 Command Module Enclosure

## Overview
**Time Estimate:** 4-6 hours | **Cost:** $75 | **Depends On:** D-3.4

---

## Components to House
| Component | Size (approx) |
|-----------|---------------|
| Raspberry Pi 5 | 85×56mm |
| USB Audio Interface | 200×150mm |
| HDMI Switch | 100×60mm |
| Audio Extractor | 120×80mm |
| ESP32 Master TX | 70×25mm |
| Power supplies | Various |

## Enclosure Options

### Option 1: Project Box ($30)
- Hammond 1598GBK or similar
- Size: 250×180×100mm
- Pros: Cheap, available
- Cons: Requires drilling

### Option 2: 3D Printed ($50)
- Custom design in Fusion 360
- PETG for durability
- Pros: Perfect fit
- Cons: Time to print

### Option 3: 2U Rack Case ($80)
- Standard 19" rack mount
- Pros: Professional look
- Cons: Larger, heavier

## Layout Design
```
┌─────────────────────────────────────────────────┐
│ FRONT PANEL                                     │
│  [HDMI1] [HDMI2] [HDMI3] [HDMI4]  [PWR] [LED]  │
├─────────────────────────────────────────────────┤
│                                                 │
│  ┌─────────┐  ┌─────────────────┐  ┌────────┐ │
│  │   Pi 5  │  │  Audio Interface│  │ HDMI   │ │
│  │         │  │                 │  │ Switch │ │
│  └─────────┘  └─────────────────┘  └────────┘ │
│                                                 │
│  ┌─────────────────┐  ┌──────┐  ┌───────────┐ │
│  │ Audio Extractor │  │ESP32 │  │  Power    │ │
│  │                 │  │  TX  │  │  Supply   │ │
│  └─────────────────┘  └──────┘  └───────────┘ │
│                                                 │
├─────────────────────────────────────────────────┤
│ REAR PANEL                                      │
│  [HDMI OUT] [USB] [POWER] [ANTENNA]            │
└─────────────────────────────────────────────────┘
```

## Ventilation
- Intake vents on sides (mesh covered)
- 40mm fan near Pi 5 (quiet PWM)
- Exhaust near top/rear

## Assembly Steps
1. Mark and drill/cut all panel holes
2. Install panel-mount connectors
3. Mount components with standoffs
4. Route internal wiring
5. Install ventilation fan
6. Add cable management
7. Test fit lid/cover
8. Label all ports

## Bill of Materials
| Item | Cost |
|------|------|
| Enclosure | $30-80 |
| Panel connectors | $15 |
| 40mm Fan | $8 |
| Standoffs/screws | $5 |
| Wire/cable | $7 |

## Success Criteria
- [ ] All components fit
- [ ] Adequate ventilation (Pi <70°C under load)
- [ ] Clean cable routing
- [ ] All ports accessible
- [ ] Secure mounting (no rattling)

**→ Complete Phase 3 checkpoint**
