# D-5.1 Production Jig & Process

## Overview
**Time:** 4-6 hours | **Cost:** $50 | **Skill:** Intermediate

## Objective
Create jigs and processes to efficiently build 7 more pods.

## Assembly Jig Design
```
┌────────────────────────────────────────┐
│         WIRING JIG                     │
│  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ │
│  │ESP32 │ │ DAC  │ │ Amp  │ │ Buck │ │
│  │holder│ │holder│ │holder│ │holder│ │
│  └──┬───┘ └──┬───┘ └──┬───┘ └──┬───┘ │
│  ───┴────────┴────────┴────────┴───   │
│        Wire routing channels           │
└────────────────────────────────────────┘
```

## QC Test Station
1. **Power test:** 9.6V from battery
2. **5V rail test:** Buck output
3. **Audio test:** Loopback tone
4. **Wireless test:** Receive from TX
5. **Full system:** 10 min burn-in

## Production Checklist (Per Pod)
```
□ Program ESP32 with channel ID
□ Wire DAC to ESP32
□ Wire amp to DAC
□ Assemble battery pack
□ Wire power distribution
□ Mount in enclosure
□ Connect speaker
□ Run QC tests
□ Label with channel ID
□ 1-hour burn-in
□ Final inspection
```

## Channel Programming
```bash
# Flash script per channel
./flash.sh 0  # FL - Front Left
./flash.sh 1  # FR - Front Right
./flash.sh 2  # C  - Center
./flash.sh 4  # SL - Surround Left
./flash.sh 5  # SR - Surround Right
./flash.sh 6  # RL - Rear Left
./flash.sh 7  # RR - Rear Right
./flash.sh 3  # SUB - Subwoofer (LFE)
```

## Time Estimate Per Pod
| Task | Time |
|------|------|
| Program ESP32 | 5 min |
| Wire electronics | 30 min |
| Assemble battery | 20 min |
| Mount in enclosure | 20 min |
| QC test | 15 min |
| Burn-in | 60 min |
| **Total** | ~2.5 hours |

→ Proceed to D-5.2 (Satellite Build)
