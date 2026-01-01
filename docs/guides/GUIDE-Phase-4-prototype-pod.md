# Implementation Guide: Phase 4 - Prototype Speaker Pod

## Overview
**Phase Budget:** $175  
**Time Estimate:** 2-3 weeks  
**Objective:** Build and validate one complete self-contained speaker pod

---

## Pod Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      SPEAKER POD INTERNALS                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐     │
│  │ ESP32   │───►│ PCM5102 │───►│ TPA3116 │───►│ 4" Full │     │
│  │ RX      │I2S │ DAC     │    │ Amp     │    │ Range   │     │
│  └────┬────┘    └─────────┘    └────┬────┘    │ Driver  │     │
│       │                             │         └─────────┘     │
│       │ 5V                          │ 12V                      │
│       │                             │                          │
│  ┌────┴─────────────────────────────┴────┐                    │
│  │           POWER SYSTEM                 │                    │
│  │  ┌─────────┐   ┌─────────┐   ┌─────┐  │                    │
│  │  │ 3S      │──►│  BMS    │──►│Buck │  │                    │
│  │  │ LiFePO4 │   │         │   │ 5V  │  │                    │
│  │  │ 9.6V    │   └────┬────┘   └─────┘  │                    │
│  │  └─────────┘        │                  │                    │
│  │                     │ 9.6V to Amp      │                    │
│  └─────────────────────┴──────────────────┘                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## D-4.1: Class-D Amplifier Integration

### Parts

| Item | Model | Cost | Link |
|------|-------|------|------|
| TPA3116D2 Amp Board | 2×50W | $12 | [Amazon](https://amazon.com/dp/B07Q2X8J5G) |
| 4" Full-Range Driver | Dayton Audio | $25 | [Parts Express](https://parts-express.com) |
| Binding Posts | — | $5 | Amazon |

### TPA3116 Pinout

```
TPA3116D2 Board:
┌─────────────────────────────────────────┐
│  [VCC+] [VCC-]        [R+] [R-]        │
│    12-24V DC          Right Speaker     │
│                                         │
│  [L IN] [R IN] [GND]  [L+] [L-]        │
│   Audio Input          Left Speaker     │
│                                         │
│  [MUTE] [STBY]        Volume Pot        │
└─────────────────────────────────────────┘
```

### Wiring

```
PCM5102A                 TPA3116D2
┌────────┐               ┌──────────┐
│   L OUT├───────────────┤ L IN     │
│   R OUT├───────────────┤ R IN     │
│   GND  ├───────────────┤ GND      │
└────────┘               │          │
                         │ L+ ──────┼──► Speaker +
Battery 9.6V ────────────┤ VCC+     │
Battery GND ─────────────┤ VCC-     │
                         │ L- ──────┼──► Speaker -
                         │          │
                         │ MUTE ────┼──► 3.3V (unmuted)
                         └──────────┘
```

### Test Procedure
1. Connect DAC output to amp input
2. Connect speaker to amp output
3. Apply 12V power (bench supply first!)
4. Play test tone from D-1.3
5. Verify clean audio, no oscillation

---

## D-4.2: Battery System Design

### Parts

| Item | Specs | Qty | Cost |
|------|-------|-----|------|
| LiFePO4 Cell | 32650, 3.2V, 6Ah | 3 | $15 each |
| 3S BMS | 10A continuous | 1 | $8 |
| Buck Converter | 5V 3A | 1 | $5 |
| XT60 Connector | — | 2 | $3 |
| Battery Holder | 32650 3S | 1 | $5 |

### Battery Pack Assembly

```
Cell Configuration: 3S1P (Series)
Nominal Voltage: 3.2V × 3 = 9.6V
Capacity: 6Ah
Energy: 57.6Wh

                    ┌─────────────┐
          ┌────────►│    BMS      │◄────────┐
          │  B-     │   3S 10A    │  B+     │
          │         └──────┬──────┘         │
          │                │ P- P+          │
          │                │                │
    ┌─────┴─────┐    ┌─────┴─────┐    ┌────┴────┐
    │   Cell 1  │────│   Cell 2  │────│  Cell 3 │
    │   3.2V    │    │   3.2V    │    │  3.2V   │
    │   6Ah     │    │   6Ah     │    │  6Ah    │
    └───────────┘    └───────────┘    └─────────┘
          -                                  +
```

### BMS Wiring

```
BMS Pin     Connect To
─────────   ──────────
B-          Cell 1 negative
B1          Cell 1 positive / Cell 2 negative
B2          Cell 2 positive / Cell 3 negative  
B+          Cell 3 positive
P-          Output negative (to load)
P+          Output positive (to load)
```

### Power Distribution

```
Battery Pack (9.6V)
        │
        ├────────────────► TPA3116 Amp (9.6V direct)
        │
        ▼
  ┌──────────┐
  │ Buck     │
  │ 5V 3A    │───────────► ESP32 + PCM5102A
  └──────────┘
```

### Runtime Calculation

```
ESP32 + DAC:  ~0.3A @ 5V  = 1.5W
TPA3116:      ~2A avg @ 9.6V = 19W (at medium volume)
Total:        ~21W

Battery:      57.6Wh
Runtime:      57.6Wh / 21W = 2.7 hours (conservative)
              At low volume: 4+ hours
```

---

## D-4.3: Speaker Pod Enclosure

### Design Requirements
- Internal volume: ~2-3 liters (for 4" driver)
- Sealed or ported design
- Space for: Battery, ESP32, DAC, Amp
- Weather resistant (outdoor use)
- Mounting point for stand

### Recommended Dimensions

```
┌─────────────────────────────┐
│                             │  Height: 8" (200mm)
│      ┌───────────┐          │  Width:  6" (150mm)
│      │  Driver   │          │  Depth:  6" (150mm)
│      │    4"     │          │
│      └───────────┘          │  Internal Volume: ~2.5L
│                             │
│  ┌─────┐ ┌─────┐ ┌─────┐   │
│  │Batt │ │ESP32│ │ Amp │   │
│  └─────┘ └─────┘ └─────┘   │
│                             │
│    [Power] [Status LED]     │
└─────────────────────────────┘
```

### Build Options

1. **3D Print** (Recommended for prototype)
   - Material: PETG (weather resistant)
   - Wall thickness: 3mm minimum
   - Infill: 30%+ for rigidity

2. **Wood/MDF**
   - 1/2" MDF
   - Seal with paint/lacquer

3. **Modify Existing**
   - Ammo can
   - Pelican case
   - Commercial speaker cabinet

---

## D-4.4: Prototype Assembly & Test

### Assembly Order

1. **Test all components separately first**
2. Mount driver to enclosure (seal with gasket)
3. Install battery pack (secure firmly)
4. Mount BMS and buck converter
5. Wire power distribution
6. Mount ESP32 and DAC
7. Mount amplifier (with heatsink clearance!)
8. Connect all wiring
9. Add status LED and power switch
10. Close enclosure

### Final Wiring Diagram

```
[Battery]──►[BMS]──┬──►[Buck 5V]──►[ESP32]──I2S──►[PCM5102A]
                   │                                  │
                   │                              [Audio Out]
                   │                                  │
                   └──►[TPA3116]◄─────────────────────┘
                          │
                          ▼
                      [Speaker]
```

### Test Checklist

- [ ] Battery charges correctly
- [ ] BMS protects against over-discharge
- [ ] ESP32 boots and connects to TX
- [ ] Audio plays wirelessly
- [ ] Volume acceptable
- [ ] No overheating after 1 hour
- [ ] Runtime >3 hours confirmed

### Drop Test
- Drop from 1 meter onto grass
- Verify still functional
- Check for loose components

---

## Phase 4 Checkpoint

### Measurements

| Metric | Target | Actual |
|--------|--------|--------|
| Weight | <3 kg | |
| Runtime | >3 hours | |
| Max SPL | >90 dB @ 1m | |
| Frequency response | 80Hz-18kHz | |

### GO/NO-GO

✅ **GO** if prototype performs well  
🔄 **ITERATE** if minor issues  
❌ **REDESIGN** if major problems

**→ If GO, proceed to Phase 5 (Production)**
