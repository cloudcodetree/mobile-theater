# Implementation Guide: D-5.3 Build Subwoofer Pod

## Overview
**Time Estimate:** 6-8 hours | **Cost:** $302 | **Depends On:** D-5.2

---

## Subwoofer vs Satellite Differences
| Spec | Satellite | Subwoofer |
|------|-----------|-----------|
| Driver | 4" full-range | 8" subwoofer |
| Amp | TPA3116 2×50W | TPA3255 2×300W |
| Battery | 3S LiFePO4 | 4S LiFePO4 |
| Enclosure | ~2.5L sealed | ~15L ported |
| Filter | None | 80Hz low-pass |
| Channel ID | 0-2, 4-7 | 3 (LFE) |

## Bill of Materials
| Item | Cost |
|------|------|
| 8" Subwoofer Driver (Dayton Audio) | $60 |
| TPA3255 Amp Board | $40 |
| 4S LiFePO4 Pack (32650 cells ×4) | $80 |
| 4S BMS (20A) | $15 |
| Buck Converter 5V | $5 |
| Large Enclosure | $80 |
| Port Tube (2" × 6") | $5 |
| Misc (wire, connectors) | $17 |
| **Total** | **$302** |

## Enclosure Design
```
┌─────────────────────────────────┐
│                                 │
│   Height: 350mm (14")           │
│   Width:  250mm (10")           │
│   Depth:  300mm (12")           │
│                                 │
│   Internal Volume: ~15L         │
│   Port: 2" × 6" (tuned ~40Hz)   │
│                                 │
└─────────────────────────────────┘
```

## 4S Battery Configuration
```
4S1P = 4 cells in Series
Voltage: 3.2V × 4 = 12.8V nominal
Capacity: 6Ah
Energy: 76.8Wh
```

## Low-Pass Filter (Firmware)
```cpp
// Simple IIR low-pass filter at 80Hz
// In subwoofer satellite_rx.ino

#define CUTOFF_HZ 80
#define SAMPLE_RATE 48000

// Precomputed coefficient for 80Hz @ 48kHz
const float alpha = 0.0104;  // 2*pi*fc/fs

float filteredL = 0, filteredR = 0;

void applyLowPass(int16_t* samples, int count) {
    for (int i = 0; i < count; i += 2) {
        // Left channel
        filteredL += alpha * (samples[i] - filteredL);
        samples[i] = (int16_t)filteredL;
        
        // Right channel (sum to mono for sub)
        filteredR += alpha * (samples[i+1] - filteredR);
        samples[i+1] = (int16_t)filteredR;
        
        // Mix to mono (optional)
        int16_t mono = (samples[i] + samples[i+1]) / 2;
        samples[i] = mono;
        samples[i+1] = mono;
    }
}
```

## Ported Enclosure Tuning
```
Port Calculation:
- Box volume: 15L
- Tuning freq: 40Hz
- Port diameter: 2" (51mm)
- Port length: ~6" (152mm)

Use online port calculator to verify.
```

## Assembly Notes
1. Subwoofer needs MORE reinforcement (heavier driver)
2. Seal port tube with silicone
3. More polyfill needed for larger volume
4. Consider adding handles (heavy!)
5. Use thicker speaker wire (14 AWG)

## Testing
- [ ] Low frequency response (30-80Hz)
- [ ] No port chuffing at high volume
- [ ] Integration with satellites (crossover)
- [ ] Extended runtime test (larger battery)

## Success Criteria
- [ ] Plays down to 35Hz cleanly
- [ ] No distortion at moderate volume
- [ ] Runtime >4 hours
- [ ] Integrates well with satellites

**→ Complete Phase 5 checkpoint**
