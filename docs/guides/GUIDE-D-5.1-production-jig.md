# Implementation Guide: D-5.1 Production Jig & Process

## Overview
**Time Estimate:** 4-6 hours | **Cost:** $50 | **Depends On:** Phase 4

---

## Objective
Create assembly jigs and document repeatable production process.

## Wiring Jig
Build a simple fixture to hold components during wiring:

```
┌─────────────────────────────────────────────┐
│           ASSEMBLY JIG (Plywood Base)       │
│                                             │
│  ┌───────┐  ┌───────┐  ┌───────┐  ┌──────┐ │
│  │ ESP32 │  │  DAC  │  │  Amp  │  │ Buck │ │
│  │ Clip  │  │ Clip  │  │ Clip  │  │ Clip │ │
│  └───┬───┘  └───┬───┘  └───┬───┘  └───┬──┘ │
│      │          │          │          │     │
│  ────┴──────────┴──────────┴──────────┴──── │
│           Wire Routing Channels             │
│                                             │
│  [Labels] [Wire Lengths] [Crimping Guide]  │
└─────────────────────────────────────────────┘
```

## Jig Materials
| Item | Cost |
|------|------|
| Plywood base 12"x18" | $10 |
| 3D printed clips/holders | $5 |
| Labels | $3 |
| Wire guides | $2 |

## Pre-Cut Wire Lengths
| Connection | Length | Color |
|------------|--------|-------|
| Battery → BMS | 50mm | Red/Black |
| BMS → Buck | 80mm | Red/Black |
| Buck → ESP32 | 60mm | Red/Black |
| ESP32 → DAC (I2S) | 100mm | Yellow/Green/Blue |
| DAC → Amp | 80mm | White (L/R) |
| Amp → Driver | 150mm | Speaker wire |

## QC Test Fixture
```cpp
// qc_test.ino - Flash to each pod for testing

#include <esp_now.h>
#include <WiFi.h>
#include "driver/i2s.h"

void setup() {
    Serial.begin(115200);
    Serial.println("=== POD QC TEST ===");
    
    // Test 1: WiFi/ESP-NOW
    WiFi.mode(WIFI_STA);
    Serial.printf("✓ MAC: %s
", WiFi.macAddress().c_str());
    
    if (esp_now_init() == ESP_OK) {
        Serial.println("✓ ESP-NOW: OK");
    } else {
        Serial.println("✗ ESP-NOW: FAIL");
    }
    
    // Test 2: I2S
    // [I2S init code]
    Serial.println("✓ I2S: OK");
    
    // Test 3: Play tone
    Serial.println("Playing 1kHz tone for 3 seconds...");
    // [Tone generation code]
    
    Serial.println("=== QC COMPLETE ===");
}

void loop() {}
```

## Production Checklist (Per Pod)
```
POD #: ______  CHANNEL: ______  DATE: ______

□ 1. ESP32 programmed (Channel ID: ___)
□ 2. MAC recorded: ___:___:___:___:___:___
□ 3. DAC wired and tested
□ 4. Amp wired (check polarity)
□ 5. Battery assembled
    □ Cell 1: ___V  Cell 2: ___V  Cell 3: ___V
□ 6. BMS wired correctly
□ 7. Buck output: ___V (target: 5.0V)
□ 8. Driver mounted and sealed
□ 9. All wiring secured (zip ties)
□ 10. QC test passed
□ 11. 1-hour burn-in complete
□ 12. Final inspection

Tested by: ____________  Date: ______
```

## Batch Organization
Track all pods in a spreadsheet:
| Pod# | Channel | MAC | QC Date | Notes |
|------|---------|-----|---------|-------|
| 1 | FL (0) | | | |
| 2 | FR (1) | | | |
| 3 | C (2) | | | |
| 4 | SL (4) | | | |
| 5 | SR (5) | | | |
| 6 | RL (6) | | | |
| 7 | RR (7) | | | |
| 8 | SUB (3) | | | |

**→ Proceed to D-5.2**
