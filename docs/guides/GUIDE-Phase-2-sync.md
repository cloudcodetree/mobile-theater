# Implementation Guide: Phase 2 - Multi-Channel Sync

## Overview
**Phase Budget:** $55  
**Time Estimate:** 1-2 weeks  
**Objective:** Prove multiple receivers stay synchronized (<1ms drift)

---

## D-2.1: Add Second Receiver Node

### Parts Needed
| Item | Qty | Cost |
|------|-----|------|
| ESP32-S3-DevKitC-1 | 1 | $10 |
| PCM5102A DAC | 1 | $5 |
| Jumper wires | 10 | — |

### Setup
Wire second RX identical to first:
```
RX2: ESP32-S3 + PCM5102A (same wiring as D-1.2)
```

### Modified TX for Broadcast

```cpp
// audio_stream_tx_broadcast.ino
// Broadcasts to ALL receivers (no specific MAC)

#include <esp_now.h>
#include <WiFi.h>
#include <math.h>

#define SAMPLE_RATE 44100
#define TONE_FREQ 1000
#define AMPLITUDE 20000
#define SAMPLES_PER_PACKET 60

// Broadcast address (all 0xFF)
uint8_t broadcastMAC[] = {0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF};

struct AudioPacket {
  uint16_t sequence;
  uint16_t timestamp;
  uint8_t  flags;
  uint8_t  reserved;
  int16_t  samples[SAMPLES_PER_PACKET * 2];
};

AudioPacket packet;
uint16_t sequenceNum = 0;
float phase = 0;

void setup() {
  Serial.begin(115200);
  Serial.println("=== ESP-NOW Broadcast TX ===");
  
  WiFi.mode(WIFI_STA);
  esp_now_init();
  
  // Add broadcast peer
  esp_now_peer_info_t peerInfo = {};
  memcpy(peerInfo.peer_addr, broadcastMAC, 6);
  peerInfo.channel = 0;
  peerInfo.encrypt = false;
  esp_now_add_peer(&peerInfo);
  
  Serial.println("Broadcasting audio to all receivers...");
}

void loop() {
  float phaseIncrement = 2.0 * PI * TONE_FREQ / SAMPLE_RATE;
  
  for (int i = 0; i < SAMPLES_PER_PACKET; i++) {
    int16_t sample = (int16_t)(AMPLITUDE * sin(phase));
    packet.samples[i * 2] = sample;
    packet.samples[i * 2 + 1] = sample;
    phase += phaseIncrement;
    if (phase >= 2.0 * PI) phase -= 2.0 * PI;
  }
  
  packet.sequence = sequenceNum++;
  packet.timestamp = (uint16_t)(millis() & 0xFFFF);
  
  esp_now_send(broadcastMAC, (uint8_t*)&packet, sizeof(packet));
  
  delayMicroseconds(1360);
}
```

### Test
1. Flash broadcast TX to Board 1
2. Flash RX code to Board 2 and Board 3
3. Connect headphones/speakers to both RX DACs
4. Listen for simultaneous audio on both

---

## D-2.2: Sync Protocol Implementation

### Sync Beacon Structure

```cpp
// Add to packet structure
struct SyncBeacon {
  uint8_t  type;           // 0x01 = sync beacon
  uint32_t masterTime;     // Microseconds on TX
  uint16_t beaconSeq;      // Beacon sequence number
};

// TX sends sync beacon every 100 packets
if (sequenceNum % 100 == 0) {
  SyncBeacon beacon;
  beacon.type = 0x01;
  beacon.masterTime = micros();
  beacon.beaconSeq = sequenceNum / 100;
  esp_now_send(broadcastMAC, (uint8_t*)&beacon, sizeof(beacon));
}
```

### RX Clock Discipline

```cpp
// RX sync variables
int32_t clockOffset = 0;
uint32_t lastSyncTime = 0;

void handleSyncBeacon(SyncBeacon *beacon) {
  uint32_t localTime = micros();
  uint32_t masterTime = beacon->masterTime;
  
  // Calculate offset (simple version)
  int32_t newOffset = (int32_t)(masterTime - localTime);
  
  // Smooth the offset (low-pass filter)
  clockOffset = (clockOffset * 7 + newOffset) / 8;
  
  lastSyncTime = localTime;
  
  Serial.printf("Sync: offset = %ld us
", clockOffset);
}

// Use offset when scheduling playback
uint32_t getSyncedTime() {
  return micros() + clockOffset;
}
```

---

## D-2.3: Measure Sync Accuracy

### Test Setup

```
                    ┌─────────────┐
                    │ Logic       │
                    │ Analyzer    │
                    │             │
                    │ CH1  CH2    │
                    └──┬────┬─────┘
                       │    │
    ┌──────────────────┼────┼──────────────────┐
    │                  │    │                  │
    ▼                  │    │                  ▼
┌───────┐              │    │              ┌───────┐
│ RX1   │──GPIO──►─────┘    └─────◄──GPIO──│ RX2   │
│ DAC   │                                  │ DAC   │
└───┬───┘                                  └───┬───┘
    │                                          │
    ▼                                          ▼
  🔊 Speaker 1                            🔊 Speaker 2
```

### Add GPIO Pulse to RX

```cpp
// In RX code - pulse GPIO on each packet playback
#define SYNC_PIN 7

void setup() {
  // ... existing setup ...
  pinMode(SYNC_PIN, OUTPUT);
}

void playPacket() {
  digitalWrite(SYNC_PIN, HIGH);  // Pulse start
  
  // Play audio via I2S...
  i2s_write(...);
  
  digitalWrite(SYNC_PIN, LOW);   // Pulse end
}
```

### Measurement Procedure

1. Connect logic analyzer CH1 to RX1 SYNC_PIN
2. Connect logic analyzer CH2 to RX2 SYNC_PIN
3. Capture both channels
4. Measure time difference between rising edges
5. Record over 1 minute, calculate drift

### Expected Results

| Metric | Target | Acceptable |
|--------|--------|------------|
| Initial sync | <1ms | <2ms |
| Drift over 1 min | <1ms | <2ms |
| Drift over 10 min | <1ms | <5ms |

---

## Phase 2 Checkpoint

### Tests
- [ ] 3 nodes receive same audio
- [ ] Sync measured with logic analyzer
- [ ] Drift documented

### GO/NO-GO

✅ **GO** if:
- Sync accuracy <1ms
- Drift <1ms over 10 minutes
- All nodes stable

❌ **NO-GO** if:
- Sync >5ms
- Drift accumulates rapidly
- Nodes desync frequently

**→ If GO, proceed to Phase 3 (Command Module)**
