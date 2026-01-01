# Implementation Guide: D-5.3 Build Subwoofer Pod

## Overview
**Time:** 6-8 hours | **Cost:** $302

---

## Subwoofer BOM

| Item | Cost |
|------|------|
| 8" Subwoofer Driver | $60 |
| TPA3255 Amp (300W) | $40 |
| 4S LiFePO4 Pack | $80 |
| 4S BMS (20A) | $15 |
| Large Enclosure | $80 |
| Misc | $27 |
| **Total** | **$302** |

## Differences from Satellite

| Spec | Satellite | Subwoofer |
|------|-----------|-----------|
| Driver | 4" | 8" |
| Amp | TPA3116 50W | TPA3255 300W |
| Battery | 3S 9.6V | 4S 12.8V |
| Volume | 2.5L | 15L |
| Channel | 0-7 | 3 (LFE) |

## Low-Pass Filter (80Hz)

```cpp
// In subwoofer firmware
#define LPF_COEFF 0.02  // 80Hz @ 48kHz

float lpfState = 0;
int16_t applyLPF(int16_t in) {
  lpfState += LPF_COEFF * (in - lpfState);
  return (int16_t)lpfState;
}
```

## Completion
- [ ] 8" driver mounted
- [ ] TPA3255 wired (12.8V)
- [ ] 4S battery assembled
- [ ] Low-pass filter enabled
- [ ] QC test passed

**→ Phase 5 Complete! Proceed to Phase 6**
