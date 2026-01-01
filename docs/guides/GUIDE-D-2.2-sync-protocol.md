# Implementation Guide: D-2.2 Implement Sync Protocol

## Overview
**Time Estimate:** 4-6 hours | **Cost:** $0 (software) | **Depends On:** D-2.1

---

## Problem
Multiple receivers have independent clocks that drift apart over time.
Even microseconds of drift causes audible phasing/echo.

## Solution: Sync Beacons
TX periodically broadcasts timestamp beacons. RX nodes adjust their
playback timing to match the master clock.

## Sync Beacon Structure
```cpp
struct SyncBeacon {
    uint8_t  type;        // 0x01 = sync beacon
    uint32_t masterTime;  // Microseconds since TX boot
    uint16_t sequence;    // Beacon sequence number
};
```

## TX Implementation
```cpp
// Add to TX loop - send beacon every 100 audio packets
if (packet.sequence % 100 == 0) {
    SyncBeacon beacon;
    beacon.type = 0x01;
    beacon.masterTime = micros();
    beacon.sequence = packet.sequence / 100;
    esp_now_send(broadcastMAC, (uint8_t*)&beacon, sizeof(beacon));
}
```

## RX Implementation
```cpp
int32_t clockOffset = 0;  // Difference from master clock

void onReceive(const uint8_t *mac, const uint8_t *data, int len) {
    // Check if sync beacon
    if (len == sizeof(SyncBeacon) && data[0] == 0x01) {
        SyncBeacon *beacon = (SyncBeacon*)data;
        uint32_t localTime = micros();
        
        // Calculate offset (with smoothing)
        int32_t newOffset = beacon->masterTime - localTime;
        clockOffset = (clockOffset * 7 + newOffset) / 8;  // Low-pass filter
        
        Serial.printf("Sync: offset=%ld us
", clockOffset);
        return;
    }
    
    // Handle audio packet...
}

// Use offset when scheduling playback
uint32_t getAdjustedTime() {
    return micros() + clockOffset;
}
```

## Jitter Buffer Adjustment
```cpp
// Adjust buffer depth based on sync
void adjustBufferDepth() {
    if (clockOffset > 1000) {
        // Were behind - reduce buffer
        targetBufferDepth = 3;
    } else if (clockOffset < -1000) {
        // Were ahead - increase buffer
        targetBufferDepth = 5;
    } else {
        targetBufferDepth = 4;  // Normal
    }
}
```

## Test Procedure
1. Flash updated TX and RX code
2. Monitor serial output for sync offsets
3. Offsets should stabilize within ±100us
4. Run for 10 minutes, verify no drift

## Success Criteria
- [ ] Sync beacons transmitted every 100 packets
- [ ] RX nodes calculate clock offset
- [ ] Offset stabilizes to <100us
- [ ] No audible phasing between speakers

**→ Proceed to D-2.3 for measurement verification**
