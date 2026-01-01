# Implementation Guide: D-2.3 Measure Sync Accuracy

## Overview
**Time Estimate:** 3-4 hours | **Cost:** $20 | **Depends On:** D-2.2

---

## Objective
Quantitatively measure synchronization accuracy between receivers.

## Equipment
| Item | Cost | Notes |
|------|------|-------|
| Logic Analyzer | $15 | Saleae clone or similar |
| Jumper wires | $5 | For test points |

## Test Setup
```
Logic Analyzer
┌─────────────┐
│ CH1 ◄───────┼────── RX1 GPIO7 (sync pulse)
│ CH2 ◄───────┼────── RX2 GPIO7 (sync pulse)
│ GND ◄───────┼────── Common ground
└─────────────┘
```

## Add Sync Pulse to RX
```cpp
#define SYNC_PULSE_PIN 7

void setup() {
    pinMode(SYNC_PULSE_PIN, OUTPUT);
    // ... rest of setup
}

// In playback function - pulse on each packet
void playAudioPacket(AudioPacket *pkt) {
    digitalWrite(SYNC_PULSE_PIN, HIGH);
    
    // Play audio via I2S
    i2s_write(I2S_NUM_0, pkt->samples, ...);
    
    digitalWrite(SYNC_PULSE_PIN, LOW);
}
```

## Measurement Procedure
1. Connect logic analyzer to both RX sync pins
2. Set sample rate to 1MHz+
3. Capture 10 seconds of data
4. Measure time difference between CH1 and CH2 rising edges
5. Calculate statistics (mean, max, std dev)

## Analysis
```
Expected waveform:

CH1: ___╱‾‾‾╲___╱‾‾‾╲___╱‾‾‾╲___
CH2: ___╱‾‾‾╲___╱‾‾‾╲___╱‾‾‾╲___
         │       │       │
         └───────┴───────┴── Should align within 1ms
```

## Success Criteria
| Metric | Target | Acceptable |
|--------|--------|------------|
| Mean offset | <100us | <500us |
| Max offset | <500us | <1ms |
| Drift over 10 min | <1ms | <2ms |

## Recording Results
| Test | Time | Offset | Notes |
|------|------|--------|-------|
| Initial | 0:00 | ___us | |
| 1 min | 1:00 | ___us | |
| 5 min | 5:00 | ___us | |
| 10 min | 10:00 | ___us | |

## If Results Are Poor
- Check sync beacon interval (try every 50 packets)
- Adjust low-pass filter coefficient
- Verify both RX on same WiFi channel
- Check for interference sources

**→ If passing, complete Phase 2 checkpoint**
