# Implementation Guide: D-5.2 Build Satellite Pods (×6)

## Overview
**Time Estimate:** 20-30 hours total | **Cost:** $1,008 | **Depends On:** D-5.1

---

## Bill of Materials (×6 Satellites)
| Item | Per Unit | ×6 Total |
|------|----------|----------|
| ESP32-S3-DevKitC-1 | $10 | $60 |
| PCM5102A DAC | $5 | $30 |
| TPA3116D2 Amp | $12 | $72 |
| 4" Full-Range Driver | $25 | $150 |
| LiFePO4 Cells (×3) | $45 | $270 |
| 3S BMS | $8 | $48 |
| Buck Converter | $5 | $30 |
| Enclosure Materials | $40 | $240 |
| Wire/Connectors/Misc | $18 | $108 |
| **Total** | **$168** | **$1,008** |

## Channel Assignments
| Pod # | Channel | Firmware ID | Notes |
|-------|---------|-------------|-------|
| 1 | Front Left | MY_CHANNEL=0 | |
| 2 | Front Right | MY_CHANNEL=1 | |
| 3 | Center | MY_CHANNEL=2 | |
| 4 | Surround Left | MY_CHANNEL=4 | |
| 5 | Surround Right | MY_CHANNEL=5 | |
| 6 | Rear Left | MY_CHANNEL=6 | |
| 7 | Rear Right | MY_CHANNEL=7 | |

## Firmware Flashing Script
```bash
#!/bin/bash
# flash_pod.sh <channel_id> <port>

CHANNEL=$1
PORT=$2

# Update channel in code
sed -i "s/#define MY_CHANNEL.*/#define MY_CHANNEL $CHANNEL/" satellite_rx.ino

# Compile and upload
arduino-cli compile --fqbn esp32:esp32:esp32s3 satellite_rx.ino
arduino-cli upload -p $PORT --fqbn esp32:esp32:esp32s3 satellite_rx.ino

echo "✓ Pod flashed: Channel $CHANNEL on $PORT"
```

## Build Schedule
Recommend building 2 pods per day:

| Day | Build | Notes |
|-----|-------|-------|
| 1 | FL, FR | Start with critical front channels |
| 2 | C, SL | |
| 3 | SR, RL | |
| 4 | RR + System Test | Test all 7 together |

## Assembly Process (Per Pod)
1. □ Label enclosure with channel name
2. □ Flash ESP32 with correct channel
3. □ Wire electronics on jig
4. □ Test electronics before installing
5. □ Install in enclosure
6. □ Final wiring
7. □ QC test
8. □ Burn-in test (1 hour)
9. □ Record in tracking sheet

## Batch Testing
After all 6 satellites built:
1. Power on all pods
2. Power on command module
3. Play test content with all 7 channels
4. Walk around room, verify correct channels
5. Check sync between all speakers
6. Run 30-minute burn-in all together

## Tracking Sheet
| Pod | Built | QC Pass | Burn-in | Notes |
|-----|-------|---------|---------|-------|
| FL | □ | □ | □ | |
| FR | □ | □ | □ | |
| C | □ | □ | □ | |
| SL | □ | □ | □ | |
| SR | □ | □ | □ | |
| RL | □ | □ | □ | |
| RR | □ | □ | □ | |

**→ Proceed to D-5.3 (Subwoofer)**
