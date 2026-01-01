# D-5.2 Build Satellite Pods (×6)

## Overview
**Time:** 15-20 hours total | **Cost:** $1,008 | **Skill:** Intermediate

## Bill of Materials (×6 satellites)
| Item | Qty | Unit | Total |
|------|-----|------|-------|
| ESP32-S3 | 6 | $10 | $60 |
| PCM5102A DAC | 6 | $5 | $30 |
| TPA3116 Amp | 6 | $12 | $72 |
| 4" Driver | 6 | $25 | $150 |
| LiFePO4 cells | 18 | $15 | $270 |
| 3S BMS | 6 | $8 | $48 |
| Buck converter | 6 | $5 | $30 |
| Enclosures | 6 | $40 | $240 |
| Misc/wire | 6 | $18 | $108 |
| **TOTAL** | | | **$1,008** |

## Channel Assignments
| Pod | Channel | Color Code |
|-----|---------|------------|
| #1 | FL (Front Left) | Red |
| #2 | FR (Front Right) | Green |
| #3 | C (Center) | Yellow |
| #4 | SL (Surround Left) | Blue |
| #5 | SR (Surround Right) | Purple |
| #6 | RL (Rear Left) | Orange |
| #7 | RR (Rear Right) | White |

## Production Sequence
Build in pairs for efficiency:
1. Week 1: FL + FR (front pair)
2. Week 2: SL + SR (surround pair)
3. Week 3: RL + RR (rear pair) + C (center)

## Quality Control Log
| Pod | Program | Wire | Battery | Assemble | QC | Burn-in |
|-----|---------|------|---------|----------|-----|---------|
| FL | □ | □ | □ | □ | □ | □ |
| FR | □ | □ | □ | □ | □ | □ |
| C | □ | □ | □ | □ | □ | □ |
| SL | □ | □ | □ | □ | □ | □ |
| SR | □ | □ | □ | □ | □ | □ |
| RL | □ | □ | □ | □ | □ | □ |
| RR | □ | □ | □ | □ | □ | □ |

## System Integration Test
After all satellites built:
- [ ] All 7 pods power on
- [ ] All receive from TX broadcast
- [ ] Sync sounds correct
- [ ] No interference between pods
- [ ] Runtime verified on all

→ Proceed to D-5.3 (Subwoofer)
