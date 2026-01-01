# Implementation Guide: D-2.3 Measure Sync Accuracy

## Overview
**Time Estimate:** 2-3 hours  
**Skill Level:** Intermediate  
**Cost:** $15-20 (logic analyzer)  
**Depends On:** D-2.2 complete

---

## Objective

Quantitatively measure synchronization accuracy between multiple receiver nodes using GPIO pulses and a logic analyzer.

---

## Equipment Needed

| Item | Cost | Notes |
|------|------|-------|
| Logic Analyzer | $10-15 | Saleae clone, 8ch, 24MHz |
| Jumper Wires | $2 | Connect GPIO to analyzer |

### Recommended Logic Analyzers

- **Budget:** "24MHz 8-Channel Logic Analyzer" (Amazon/AliExpress) ~$10
- **Better:** Saleae Logic 8 ~$400 (overkill but great software)
- **Alternative:** Oscilloscope with 2+ channels

---

## Step 1: Add GPIO Pulse to RX

Modify RX firmware to pulse a GPIO pin when playing each audio packet:

```cpp
// Add to audio_rx_sync.ino

#define SYNC_PULSE_PIN 7  // Use any free GPIO

void setup() {
  // ... existing setup ...
  pinMode(SYNC_PULSE_PIN, OUTPUT);
  digitalWrite(SYNC_PULSE_PIN, LOW);
}

void loop() {
  // ... existing code ...
  
  if (synced && buffered > 2) {
    AudioPacket *pkt = &buffer[readIdx];
    uint32_t masterNow = getMasterTime();
    
    if ((int32_t)(masterNow - pkt->playTime) >= 0) {
      // PULSE GPIO HIGH at start of playback
      digitalWrite(SYNC_PULSE_PIN, HIGH);
      
      size_t written;
      i2s_write(I2S_NUM_0, pkt->samples,
                SAMPLES_PER_PACKET * 2 * sizeof(int16_t),
                &written, portMAX_DELAY);
      
      // PULSE GPIO LOW after playback
      digitalWrite(SYNC_PULSE_PIN, LOW);
      
      readIdx = (readIdx + 1) % BUFFER_SIZE;
      buffered--;
    }
  }
}
```

---

## Step 2: Connect Logic Analyzer

```
Test Setup:

    ┌──────────────────────────────────────────┐
    │           Logic Analyzer                 │
    │                                          │
    │   CH0 ─────────┐    CH1 ─────────┐      │
    │   GND ────┐    │    GND ────┐    │      │
    └───────────┼────┼────────────┼────┼──────┘
                │    │            │    │
                │    │            │    │
           ┌────┴────┴───┐  ┌────┴────┴───┐
           │   ESP32     │  │   ESP32     │
           │   RX1       │  │   RX2       │
           │             │  │             │
           │  GPIO7 ─────┘  │  GPIO7 ─────┘
           │  GND ──────────┼─ GND        │
           │             │  │             │
           └─────────────┘  └─────────────┘
```

### Wiring

| Logic Analyzer | RX1 | RX2 |
|----------------|-----|-----|
| CH0 | GPIO7 | — |
| CH1 | — | GPIO7 |
| GND | GND | GND |

**Important:** Connect all GNDs together!

---

## Step 3: Capture and Measure

### Using PulseView (Free Software)

1. **Install PulseView**
   - https://sigrok.org/wiki/Downloads

2. **Configure**
   - Device: Your logic analyzer
   - Sample rate: 1MHz (plenty for ms-level timing)
   - Channels: Enable CH0 and CH1

3. **Capture**
   - Click Run
   - Let it capture for 5-10 seconds
   - Stop

4. **Measure**
   - Zoom to see individual pulses
   - Use cursors to measure time between:
     - CH0 rising edge
     - CH1 rising edge
   - This is your **sync error**

### Expected View

```
Time (ms):  0    1    2    3    4    5    6    7    8
            │    │    │    │    │    │    │    │    │
CH0 (RX1):  ─┐  ┌┐  ┌┐  ┌┐  ┌┐  ┌┐  ┌┐  ┌┐  ┌┐  ┌┐
             └──┘└──┘└──┘└──┘└──┘└──┘└──┘└──┘└──┘└──┘

CH1 (RX2):  ─┐  ┌┐  ┌┐  ┌┐  ┌┐  ┌┐  ┌┐  ┌┐  ┌┐  ┌┐
             └──┘└──┘└──┘└──┘└──┘└──┘└──┘└──┘└──┘└──┘
            ↑
            │ These should be nearly aligned
            │ Measure the offset between them
```

---

## Step 4: Record Measurements

### Data Collection

Take measurements at regular intervals:

| Time | CH0→CH1 Offset | Notes |
|------|----------------|-------|
| 0:00 | ______ μs | Initial |
| 1:00 | ______ μs | |
| 2:00 | ______ μs | |
| 5:00 | ______ μs | |
| 10:00 | ______ μs | |

### Calculate Statistics

```
Measurements (μs): _____, _____, _____, _____, _____

Average offset:    ______ μs
Max offset:        ______ μs
Min offset:        ______ μs
Standard deviation: ______ μs
Drift over 10min:  ______ μs
```

---

## Step 5: Interpret Results

### Sync Quality Levels

| Offset | Quality | Perception |
|--------|---------|------------|
| <100 μs | Excellent | Inaudible |
| 100-500 μs | Good | Inaudible |
| 500-1000 μs | Acceptable | Maybe audible |
| 1-3 ms | Poor | Slight phasing |
| >3 ms | Bad | Audible echo |

### GO/NO-GO Criteria

✅ **PASS** if:
- Average offset < 1ms
- Max offset < 2ms  
- Drift over 10 min < 1ms

❌ **FAIL** if:
- Average offset > 2ms
- Drift increases over time
- Offset jumps erratically

---

## Step 6: Troubleshooting Sync Issues

### If Offset Too Large (>2ms)

1. **Increase sync beacon rate**
   ```cpp
   #define SYNC_INTERVAL 50  // Was 100
   ```

2. **Reduce network jitter**
   - Change WiFi channel
   - Move devices closer

3. **Use median filter** instead of average:
   ```cpp
   // Keep last 5 offsets
   int32_t offsets[5];
   int offsetIdx = 0;
   
   void handleSync(SyncBeacon *beacon) {
     int32_t newOffset = (int32_t)(beacon->masterTime - micros());
     offsets[offsetIdx++ % 5] = newOffset;
     
     // Sort and take median
     int32_t sorted[5];
     memcpy(sorted, offsets, sizeof(sorted));
     // ... sort array ...
     clockOffset = sorted[2];  // Median
   }
   ```

### If Drift Over Time

1. **Crystal calibration issue**
   - ESP32 crystals vary ±20ppm
   - 20ppm = 20μs drift per second = 1.2ms per minute

2. **Solution: Increase sync rate or use PLL**
   ```cpp
   // Simple proportional correction
   void handleSync(SyncBeacon *beacon) {
     int32_t error = (int32_t)(beacon->masterTime - getMasterTime());
     clockOffset += error / 16;  // Gradual correction
   }
   ```

---

## Alternative: Oscilloscope Method

If you have a 2-channel oscilloscope:

1. Connect CH1 to RX1 GPIO7
2. Connect CH2 to RX2 GPIO7
3. Trigger on CH1
4. Measure delay to CH2

This gives real-time visual feedback.

---

## Completion Checklist

- [ ] Logic analyzer connected to both RX
- [ ] Pulses visible on both channels
- [ ] Offset measured: ______ μs average
- [ ] Drift measured: ______ μs over 10 min
- [ ] Results documented
- [ ] GO/NO-GO decision: ______

---

## Phase 2 Complete!

If all D-2.x deliverables pass:

- [ ] D-2.1: Second receiver works ✓
- [ ] D-2.2: Sync protocol implemented ✓
- [ ] D-2.3: Sync accuracy measured < 1ms ✓

**→ Proceed to Phase 3 (Command Module)**
