# Implementation Guide: D-5.2 Build Satellite Pods (×6)

## Overview
**Time:** 2-3 days | **Cost:** $1,008

---

## BOM (×6 Satellites)

| Item | Per Unit | ×6 Total |
|------|----------|----------|
| ESP32-S3 | $10 | $60 |
| PCM5102A | $5 | $30 |
| TPA3116 Amp | $12 | $72 |
| 4" Driver | $25 | $150 |
| LiFePO4 (3 cells) | $45 | $270 |
| 3S BMS | $8 | $48 |
| Buck Converter | $5 | $30 |
| Enclosure | $40 | $240 |
| Misc | $18 | $108 |
| **Total** | | **$1,008** |

## Channel Programming

| Pod | Channel | MY_CHANNEL |
|-----|---------|------------|
| 1 | FL | 0 |
| 2 | FR | 1 |
| 3 | C | 2 |
| 4 | SL | 4 |
| 5 | SR | 5 |
| 6 | RL | 6 |
| 7 | RR | 7 |

## Build Tracking

| Pod | Assembled | QC Pass | Burn-in |
|-----|-----------|---------|---------|
| FL | [ ] | [ ] | [ ] |
| FR | [ ] | [ ] | [ ] |
| C | [ ] | [ ] | [ ] |
| SL | [ ] | [ ] | [ ] |
| SR | [ ] | [ ] | [ ] |
| RL | [ ] | [ ] | [ ] |
| RR | [ ] | [ ] | [ ] |

**→ Proceed to D-5.3**
