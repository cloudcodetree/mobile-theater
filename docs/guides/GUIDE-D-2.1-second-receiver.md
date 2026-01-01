# Implementation Guide: D-2.1 Add Second Receiver Node

## Overview
**Time Estimate:** 2-3 hours | **Cost:** $15 | **Depends On:** D-1.3

---

## Objective
Add a third ESP32-S3 as second receiver to test multi-node broadcast.

## Parts
| Item | Qty | Cost |
|------|-----|------|
| ESP32-S3-DevKitC-1 | 1 | $10 |
| PCM5102A DAC | 1 | $5 |

## Setup
Wire RX2 identical to RX1 (same as D-1.2).

```
┌──────────┐      ┌─────────┐
│    TX    │═════►│   RX1   │──► Speaker 1
│  ESP32   │      └─────────┘
│          │      ┌─────────┐
│ Broadcast│═════►│   RX2   │──► Speaker 2
└──────────┘      └─────────┘
```

## TX Code Change
Replace specific MAC with broadcast:
```cpp
// Old: uint8_t rxMAC[] = {0xAA, 0xBB, ...};
// New:
uint8_t broadcastMAC[] = {0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF};
```

## RX Code
No changes needed - same code from D-1.3 works.

## Test Procedure
1. Flash TX with broadcast code
2. Flash both RX boards with RX code
3. Power on RX1 and RX2
4. Power on TX
5. Verify both speakers play audio

## Success Criteria
- [ ] Both RX receive packets
- [ ] Both play audio simultaneously
- [ ] Packet loss <1% on both
- [ ] Audio sounds "in sync" (subjective)

**→ Proceed to D-2.2 for precise synchronization**
