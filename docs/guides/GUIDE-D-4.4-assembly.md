# D-4.4 Prototype Pod Assembly & Test

## Overview
**Time:** 4-6 hours | **Cost:** $20 | **Skill:** Intermediate

## Final Assembly Steps

### 1. Pre-Assembly Checklist
- [ ] Enclosure complete (D-4.3)
- [ ] Battery tested (D-4.2)
- [ ] Amp tested (D-4.1)
- [ ] All firmware loaded

### 2. Wiring Diagram
```
[Battery] → [BMS] → [Switch] ─┬─→ [Buck 5V] → [ESP32] → [DAC]
                              │                            │
                              └─→ [TPA3116 Amp] ←──────────┘
                                        │
                                        ▼
                                   [Speaker]
```

### 3. Assembly Order
1. Install driver in enclosure
2. Mount amp board (thermal pad on case)
3. Mount ESP32 + DAC
4. Install battery pack (velcro secure)
5. Wire power distribution
6. Wire audio signal path
7. Install power switch + charge port
8. Close enclosure and seal

### 4. System Test
| Test | Method | Pass? |
|------|--------|-------|
| Power on | Switch on, LED lights | |
| WiFi connects | Check serial monitor | |
| Audio plays | Send test from TX | |
| Battery runtime | Play 3 hours | |
| Drop test | 1m onto grass | |

### 5. Measurements
| Metric | Target | Actual |
|--------|--------|--------|
| Weight | <3 kg | |
| SPL @ 1m | >85 dB | |
| Runtime | >3 hours | |
| Latency | <30ms | |

## Troubleshooting
| Issue | Likely Cause | Fix |
|-------|--------------|-----|
| No power | Switch or BMS | Check connections |
| No audio | Firmware/wiring | Verify ESP-NOW |
| Distortion | Gain too high | Reduce amp gain |
| Short runtime | Battery issue | Check cell balance |

## GO/NO-GO Decision
✅ GO if: All tests pass, runtime OK
🔄 ITERATE if: Minor issues to fix
❌ REDESIGN if: Major problems

→ If GO, proceed to Phase 5 (Production)
