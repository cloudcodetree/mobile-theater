# Implementation Guide: Phase 5 - Full Production

## Overview
**Phase Budget:** $1,360  
**Time Estimate:** 4-6 weeks  
**Objective:** Build remaining 7 speaker pods (6 satellites + 1 subwoofer)

---

## D-5.1: Production Jig & Process

### Assembly Jig

Build a simple jig to speed up repetitive wiring:

```
┌─────────────────────────────────────────────┐
│           WIRING JIG                        │
│  ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐       │
│  │ESP32│  │ DAC │  │ Amp │  │Buck │       │
│  │Holder│ │Holder│ │Holder│ │Holder│       │
│  └──┬──┘  └──┬──┘  └──┬──┘  └──┬──┘       │
│     │        │        │        │           │
│  ───┴────────┴────────┴────────┴───        │
│            Wire Routing Channels            │
└─────────────────────────────────────────────┘
```

### QC Test Fixture

```cpp
// qc_test.ino - Flash to test each pod

void setup() {
  Serial.begin(115200);
  Serial.println("=== POD QC TEST ===");
  
  // Test 1: WiFi
  WiFi.mode(WIFI_STA);
  Serial.printf("MAC: %s
", WiFi.macAddress().c_str());
  
  // Test 2: I2S
  // Play test tone...
  
  // Test 3: ESP-NOW
  // Receive test packet...
  
  Serial.println("=== ALL TESTS PASSED ===");
}
```

### Assembly Checklist (Per Pod)

```
□ ESP32-S3 programmed with correct channel ID
□ DAC wired and tested
□ Amp wired (check polarity!)
□ Battery pack assembled
□ BMS wired correctly
□ Buck converter output = 5.0V ±0.1V
□ Driver mounted and sealed
□ All components secured
□ Wiring dressed and zip-tied
□ Power switch functional
□ Status LED works
□ QC test passed
□ 1-hour burn-in complete
```

---

## D-5.2: Satellite Pod Production (×6)

### Bill of Materials (×6)

| Item | Per Unit | ×6 Total |
|------|----------|----------|
| ESP32-S3-DevKitC-1 | $10 | $60 |
| PCM5102A DAC | $5 | $30 |
| TPA3116 Amp | $12 | $72 |
| 4" Driver | $25 | $150 |
| LiFePO4 Cells (×3) | $45 | $270 |
| 3S BMS | $8 | $48 |
| Buck Converter | $5 | $30 |
| Enclosure/Materials | $40 | $240 |
| Connectors/Wire/Misc | $18 | $108 |
| **Total** | **$168** | **$1,008** |

### Channel Assignment

| Pod # | Channel | ID in Code |
|-------|---------|------------|
| 1 | Front Left (FL) | 0 |
| 2 | Front Right (FR) | 1 |
| 3 | Center (C) | 2 |
| 4 | Surround Left (SL) | 4 |
| 5 | Surround Right (SR) | 5 |
| 6 | Rear Left (RL) | 6 |
| 7 | Rear Right (RR) | 7 |

### Firmware Flashing Script

```bash
#!/bin/bash
# flash_satellite.sh

CHANNEL_ID=$1
PORT=$2

if [ -z "$CHANNEL_ID" ] || [ -z "$PORT" ]; then
    echo "Usage: ./flash_satellite.sh <channel_id> <port>"
    echo "Channel IDs: 0=FL, 1=FR, 2=C, 4=SL, 5=SR, 6=RL, 7=RR"
    exit 1
fi

# Set channel ID in code
sed -i "s/#define MY_CHANNEL .*/#define MY_CHANNEL $CHANNEL_ID/" satellite_rx.ino

# Compile and flash
arduino-cli compile --fqbn esp32:esp32:esp32s3 satellite_rx.ino
arduino-cli upload -p $PORT --fqbn esp32:esp32:esp32s3 satellite_rx.ino

echo "Flashed channel $CHANNEL_ID to $PORT"
```

---

## D-5.3: Subwoofer Pod

### Differences from Satellite

| Spec | Satellite | Subwoofer |
|------|-----------|-----------|
| Driver | 4" full-range | 8" subwoofer |
| Amp | TPA3116 2×50W | TPA3255 2×300W |
| Battery | 3S LiFePO4 | 4S LiFePO4 |
| Enclosure | ~2.5L sealed | ~15L ported |
| Filter | None | 80Hz low-pass |

### Subwoofer BOM

| Item | Cost |
|------|------|
| 8" Subwoofer Driver | $60 |
| TPA3255 Amp Board | $40 |
| 4S LiFePO4 Pack | $80 |
| 4S BMS | $15 |
| Larger Enclosure | $80 |
| Misc | $27 |
| **Total** | **$302** |

### Low-Pass Filter

```cpp
// In subwoofer firmware
// Simple IIR low-pass filter at 80Hz

float lowPassCoeff = 0.01;  // Adjust for cutoff
float filteredSample = 0;

int16_t applyLowPass(int16_t sample) {
  filteredSample += lowPassCoeff * (sample - filteredSample);
  return (int16_t)filteredSample;
}
```

---

## Phase 5 Completion

### Pod Inventory

| # | Channel | Built | QC | Burn-in |
|---|---------|-------|-----|---------|
| 1 | FL | □ | □ | □ |
| 2 | FR | □ | □ | □ |
| 3 | C | □ | □ | □ |
| 4 | SL | □ | □ | □ |
| 5 | SR | □ | □ | □ |
| 6 | RL | □ | □ | □ |
| 7 | RR | □ | □ | □ |
| 8 | SUB | □ | □ | □ |

### System Test

- [ ] All 8 pods receive audio
- [ ] Sync verified across all pods
- [ ] No interference between pods
- [ ] Runtime >3 hours on all

**→ If complete, proceed to Phase 6 (Case Integration)**
