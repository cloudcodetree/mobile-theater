# Implementation Guide: D-2.2 Implement Sync Protocol

## Overview
**Time Estimate:** 4-6 hours  
**Skill Level:** Intermediate  
**Cost:** $0 (software only)  
**Depends On:** D-2.1 complete

---

## The Sync Problem

Without synchronization, multiple receivers will drift apart over time:

```
Time ──────────────────────────────────────────►

RX1: ████████████████████████████████████████
RX2: ████████████████████████████████████████
         ↑ Start aligned, but...

After 1 minute:
RX1: ████████████████████████████████████████
RX2:  ████████████████████████████████████████
      ↑ Small drift accumulates

After 10 minutes:
RX1: ████████████████████████████████████████
RX2:     ████████████████████████████████████████
         ↑↑↑ Audible delay (>5ms sounds like echo)
```

---

## Sync Strategy

### Approach: Timestamp-Based Sync

1. TX sends periodic **sync beacons** with its timestamp
2. RX calculates **clock offset** from beacon
3. RX adjusts **playback timing** to match TX clock
4. All RX nodes align to same reference

```
┌─────────────────────────────────────────────────────────────┐
│                    SYNC PROTOCOL                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   TX                          RX1           RX2             │
│   ──                          ───           ───             │
│    │                           │             │              │
│    │── Sync Beacon ──────────►│             │              │
│    │   (masterTime=1000)      │             │              │
│    │                          │             │              │
│    │── Sync Beacon ─────────────────────────►              │
│    │   (masterTime=1000)                    │              │
│    │                           │             │              │
│    │                          RX1 calc:     RX2 calc:       │
│    │                          offset =      offset =        │
│    │                          1000-1002     1000-998        │
│    │                          = -2ms        = +2ms          │
│    │                           │             │              │
│    │── Audio Packet ─────────►│────────────►│              │
│    │   (playAt=1050)          │             │              │
│    │                          │             │              │
│    │                         Play at       Play at          │
│    │                         1050-2=1048   1050+2=1052      │
│    │                         (adjusted)    (adjusted)       │
│    │                           │             │              │
│    │                          Both play at same real time!  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Step 1: Define Sync Beacon

```cpp
// Add to packet types

#define PKT_TYPE_AUDIO 0x01
#define PKT_TYPE_SYNC  0x02

struct SyncBeacon {
  uint8_t  type;         // PKT_TYPE_SYNC
  uint8_t  reserved;
  uint16_t beaconSeq;    // Beacon sequence number
  uint32_t masterTime;   // TX micros() timestamp
};
```

---

## Step 2: TX with Sync Beacons

```cpp
// audio_tx_sync.ino

#include <esp_now.h>
#include <WiFi.h>
#include <math.h>

uint8_t broadcastMAC[] = {0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF};

#define SAMPLE_RATE 44100
#define TONE_FREQ 1000
#define AMPLITUDE 20000
#define SAMPLES_PER_PACKET 60
#define SYNC_INTERVAL 100  // Send sync every 100 audio packets

#define PKT_TYPE_AUDIO 0x01
#define PKT_TYPE_SYNC  0x02

struct AudioPacket {
  uint8_t  type;
  uint8_t  reserved;
  uint16_t sequence;
  uint32_t playTime;     // When RX should play this (master time)
  int16_t  samples[SAMPLES_PER_PACKET * 2];
};

struct SyncBeacon {
  uint8_t  type;
  uint8_t  reserved;
  uint16_t beaconSeq;
  uint32_t masterTime;
};

AudioPacket audioPkt;
SyncBeacon syncPkt;
uint16_t audioSeq = 0;
uint16_t syncSeq = 0;
float phase = 0;

// Timing
uint32_t nextPacketTime = 0;
const uint32_t PACKET_INTERVAL_US = 1360;  // ~735 packets/sec

void setup() {
  Serial.begin(115200);
  Serial.println("
=== SYNC TX ===");
  
  WiFi.mode(WIFI_STA);
  Serial.printf("TX MAC: %s
", WiFi.macAddress().c_str());
  
  esp_now_init();
  
  esp_now_peer_info_t peer = {};
  memcpy(peer.peer_addr, broadcastMAC, 6);
  peer.channel = 0;
  peer.encrypt = false;
  esp_now_add_peer(&peer);
  
  syncPkt.type = PKT_TYPE_SYNC;
  audioPkt.type = PKT_TYPE_AUDIO;
  
  nextPacketTime = micros();
  
  Serial.println("Broadcasting with sync beacons...");
}

void sendSyncBeacon() {
  syncPkt.beaconSeq = syncSeq++;
  syncPkt.masterTime = micros();
  esp_now_send(broadcastMAC, (uint8_t*)&syncPkt, sizeof(syncPkt));
}

void sendAudioPacket() {
  // Generate audio
  float phaseInc = 2.0 * PI * TONE_FREQ / SAMPLE_RATE;
  for (int i = 0; i < SAMPLES_PER_PACKET; i++) {
    int16_t s = (int16_t)(AMPLITUDE * sin(phase));
    audioPkt.samples[i * 2] = s;
    audioPkt.samples[i * 2 + 1] = s;
    phase += phaseInc;
    if (phase >= 2.0 * PI) phase -= 2.0 * PI;
  }
  
  audioPkt.sequence = audioSeq;
  // Tell RX when to play: current time + buffer delay
  audioPkt.playTime = micros() + 10000;  // 10ms in future
  
  esp_now_send(broadcastMAC, (uint8_t*)&audioPkt, sizeof(audioPkt));
  audioSeq++;
}

void loop() {
  uint32_t now = micros();
  
  if ((int32_t)(now - nextPacketTime) >= 0) {
    // Send sync beacon periodically
    if (audioSeq % SYNC_INTERVAL == 0) {
      sendSyncBeacon();
    }
    
    sendAudioPacket();
    nextPacketTime += PACKET_INTERVAL_US;
  }
  
  // Stats
  static uint32_t lastPrint = 0;
  if (millis() - lastPrint > 5000) {
    Serial.printf("Audio: %u | Sync: %u
", audioSeq, syncSeq);
    lastPrint = millis();
  }
}
```

---

## Step 3: RX with Clock Sync

```cpp
// audio_rx_sync.ino

#include <esp_now.h>
#include <WiFi.h>
#include "driver/i2s.h"

#define I2S_BCK  4
#define I2S_WS   5
#define I2S_DOUT 6

#define SAMPLE_RATE 44100
#define SAMPLES_PER_PACKET 60
#define BUFFER_SIZE 16

#define PKT_TYPE_AUDIO 0x01
#define PKT_TYPE_SYNC  0x02

struct AudioPacket {
  uint8_t  type;
  uint8_t  reserved;
  uint16_t sequence;
  uint32_t playTime;
  int16_t  samples[SAMPLES_PER_PACKET * 2];
};

struct SyncBeacon {
  uint8_t  type;
  uint8_t  reserved;
  uint16_t beaconSeq;
  uint32_t masterTime;
};

// Circular buffer
AudioPacket buffer[BUFFER_SIZE];
volatile int writeIdx = 0;
volatile int readIdx = 0;
volatile int buffered = 0;

// Clock sync
volatile int32_t clockOffset = 0;  // localTime + offset = masterTime
volatile uint32_t lastSyncTime = 0;
volatile bool synced = false;

// Stats
uint32_t rxAudio = 0;
uint32_t rxSync = 0;

void handleSync(SyncBeacon *beacon) {
  uint32_t localNow = micros();
  uint32_t masterNow = beacon->masterTime;
  
  // Calculate offset (master - local)
  int32_t newOffset = (int32_t)(masterNow - localNow);
  
  if (!synced) {
    // First sync - jump to offset
    clockOffset = newOffset;
    synced = true;
    Serial.printf("Initial sync: offset = %ld us
", clockOffset);
  } else {
    // Smooth updates (low-pass filter)
    // New offset = 87.5% old + 12.5% new
    clockOffset = (clockOffset * 7 + newOffset) / 8;
  }
  
  lastSyncTime = localNow;
  rxSync++;
}

void handleAudio(AudioPacket *pkt) {
  if (buffered < BUFFER_SIZE) {
    memcpy(&buffer[writeIdx], pkt, sizeof(AudioPacket));
    writeIdx = (writeIdx + 1) % BUFFER_SIZE;
    buffered++;
  }
  rxAudio++;
}

void onReceive(const uint8_t *mac, const uint8_t *data, int len) {
  uint8_t type = data[0];
  
  if (type == PKT_TYPE_SYNC && len == sizeof(SyncBeacon)) {
    handleSync((SyncBeacon*)data);
  } else if (type == PKT_TYPE_AUDIO && len == sizeof(AudioPacket)) {
    handleAudio((AudioPacket*)data);
  }
}

uint32_t getMasterTime() {
  return micros() + clockOffset;
}

void setupI2S() {
  i2s_config_t cfg = {
    .mode = (i2s_mode_t)(I2S_MODE_MASTER | I2S_MODE_TX),
    .sample_rate = SAMPLE_RATE,
    .bits_per_sample = I2S_BITS_PER_SAMPLE_16BIT,
    .channel_format = I2S_CHANNEL_FMT_RIGHT_LEFT,
    .communication_format = I2S_COMM_FORMAT_STAND_I2S,
    .intr_alloc_flags = ESP_INTR_FLAG_LEVEL1,
    .dma_buf_count = 8,
    .dma_buf_len = 64,
    .use_apll = false,
    .tx_desc_auto_clear = true
  };
  
  i2s_pin_config_t pins = {
    .bck_io_num = I2S_BCK,
    .ws_io_num = I2S_WS,
    .data_out_num = I2S_DOUT,
    .data_in_num = I2S_PIN_NO_CHANGE
  };
  
  i2s_driver_install(I2S_NUM_0, &cfg, 0, NULL);
  i2s_set_pin(I2S_NUM_0, &pins);
}

void setup() {
  Serial.begin(115200);
  Serial.println("
=== SYNC RX ===");
  
  WiFi.mode(WIFI_STA);
  Serial.printf("RX MAC: %s
", WiFi.macAddress().c_str());
  
  setupI2S();
  esp_now_init();
  esp_now_register_recv_cb(onReceive);
  
  Serial.println("Waiting for sync...");
}

void loop() {
  static uint32_t lastStats = 0;
  
  // Play when we have packets and are synced
  if (synced && buffered > 2) {
    AudioPacket *pkt = &buffer[readIdx];
    uint32_t masterNow = getMasterTime();
    
    // Wait until playTime
    if ((int32_t)(masterNow - pkt->playTime) >= 0) {
      size_t written;
      i2s_write(I2S_NUM_0, pkt->samples,
                SAMPLES_PER_PACKET * 2 * sizeof(int16_t),
                &written, portMAX_DELAY);
      
      readIdx = (readIdx + 1) % BUFFER_SIZE;
      buffered--;
    }
  }
  
  // Stats
  if (millis() - lastStats > 5000) {
    Serial.printf("Audio: %lu | Sync: %lu | Offset: %ld us | Buf: %d
",
                  rxAudio, rxSync, clockOffset, buffered);
    lastStats = millis();
  }
}
```

---

## Step 4: Test Sync

### What to Look For

1. **Both RX show similar clock offset** (within a few hundred microseconds)
2. **Offset stays stable** over time (doesn't drift wildly)
3. **Audio sounds synchronized** when listening to both speakers

### Serial Output Example

**RX1:**
```
=== SYNC RX ===
RX MAC: 11:22:33:44:55:66
Waiting for sync...
Initial sync: offset = -1523 us
Audio: 735 | Sync: 7 | Offset: -1498 us | Buf: 4
Audio: 1470 | Sync: 14 | Offset: -1502 us | Buf: 4
```

**RX2:**
```
=== SYNC RX ===
RX MAC: 77:88:99:AA:BB:CC
Waiting for sync...
Initial sync: offset = 823 us
Audio: 733 | Sync: 7 | Offset: 845 us | Buf: 4
Audio: 1468 | Sync: 14 | Offset: 839 us | Buf: 4
```

---

## Step 5: Verify Sync Accuracy

### Subjective Test

1. Place both speakers near each other
2. Listen for any "phasing" or "echo" effect
3. If sounds like ONE speaker = good sync
4. If sounds like echo = sync is off

### Objective Test (D-2.3)

Use GPIO pulses and logic analyzer to measure actual sync - covered in next deliverable.

---

## Troubleshooting

| Issue | Cause | Fix |
|-------|-------|-----|
| Offset drifts constantly | Crystal frequency mismatch | Increase sync beacon rate |
| Large offset jumps | Network jitter | Use median filter instead of average |
| Audio dropouts | Buffer underrun | Increase playTime offset |
| One RX delayed | Different initial offset | Both should stabilize after 5-10 beacons |

---

## Completion Checklist

- [ ] TX sends sync beacons every 100 packets
- [ ] Both RX calculate clock offset
- [ ] Offset stabilizes within ±500μs
- [ ] Audio plays synchronized (subjective)
- [ ] Offset doesn't drift significantly over 5 minutes

**→ Proceed to D-2.3 (Measure Sync Accuracy)**
