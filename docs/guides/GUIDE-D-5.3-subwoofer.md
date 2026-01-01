# D-5.3 Build Subwoofer Pod

## Overview
**Time:** 6-8 hours | **Cost:** $302 | **Skill:** Intermediate

## Subwoofer vs Satellite Differences
| Spec | Satellite | Subwoofer |
|------|-----------|-----------|
| Driver | 4" full-range | 8" subwoofer |
| Enclosure | 2.5L sealed | 15L ported |
| Amp | TPA3116 50W | TPA3255 300W |
| Battery | 3S LiFePO4 | 4S LiFePO4 |
| Filter | None | 80Hz low-pass |

## Bill of Materials
| Item | Cost |
|------|------|
| 8" Subwoofer driver (Dayton Audio) | $60 |
| TPA3255 Amp board (300W) | $40 |
| LiFePO4 32650 cells (×4) | $80 |
| 4S BMS (20A) | $15 |
| Enclosure (15L) | $80 |
| Port tube (2" × 6") | $5 |
| Misc (wire, connectors) | $22 |
| **TOTAL** | **$302** |

## Enclosure Design
```
┌─────────────────────────────┐
│  ┌─────────────────────┐   │
│  │                     │   │  Height: 350mm
│  │    8" DRIVER        │   │  Width: 300mm
│  │                     │   │  Depth: 200mm
│  └─────────────────────┘   │
│                             │  Volume: ~15L
│  ┌───────┐                 │
│  │ PORT  │  [Electronics]  │
│  │ 2"×6" │  [Battery 4S]   │
│  └───────┘                 │
└─────────────────────────────┘
```

## Low-Pass Filter (Firmware)
```cpp
// 80Hz 2nd-order Butterworth LPF
#define CUTOFF_HZ 80
#define SAMPLE_RATE 44100

float b0 = 0.00032, b1 = 0.00064, b2 = 0.00032;
float a1 = -1.9712, a2 = 0.9724;

float x1=0, x2=0, y1=0, y2=0;

int16_t lowPass(int16_t in) {
    float x0 = in;
    float y0 = b0*x0 + b1*x1 + b2*x2 - a1*y1 - a2*y2;
    x2=x1; x1=x0; y2=y1; y1=y0;
    return (int16_t)y0;
}
```

## Assembly Notes
1. Port tube tuned for ~35Hz (-3dB point)
2. Stuff enclosure loosely with polyfill
3. Amp needs good heatsinking (more power = more heat)
4. Use 4S battery for higher voltage headroom

## Testing
| Test | Target | Actual |
|------|--------|--------|
| Low freq response | 35-80Hz | |
| Max SPL | >95dB @ 1m | |
| Runtime | >2 hours | |
| No port noise | Clean bass | |

→ Proceed to Phase 6 (Cases)
