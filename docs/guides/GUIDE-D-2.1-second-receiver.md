# Implementation Guide: D-2.1 Add Second Receiver Node

## Overview
**Time Estimate:** 2-3 hours  
**Skill Level:** Beginner  
**Cost:** $15  
**Depends On:** D-1.3 complete (working TX→RX audio)

---

## Objective

Add a third ESP32-S3 as a second receiver to test multi-node audio broadcast. Both receivers should play the same audio simultaneously.

---

## Step 1: Order Parts

| Item | Qty | Cost | Notes |
|------|-----|------|-------|
| ESP32-S3-DevKitC-1 | 1 | $10 | Third board |
| PCM5102A DAC | 1 | $5 | Second DAC |

**Total:** $15

---

## Step 2: Wire Second Receiver

Identical wiring to first RX (D-1.2):

```
ESP32-S3 (RX2)              PCM5102A (#2)
┌──────────────┐           ┌──────────────┐
│         3V3 ─┼───────────┼─ VCC         │
│              │     ┌─────┼─ XMT (3V3)   │
│         GND ─┼─────┼─────┼─ GND         │
│              │     │ ┌───┼─ SCK (GND)   │
│              │     │ │ ┌─┼─ FMT (GND)   │
│       GPIO4 ─┼─────┼─┼─┼─┼─ BCK         │
│       GPIO5 ─┼─────┼─┼─┼─┼─ LCK         │
│       GPIO6 ─┼─────┼─┼─┼─┼─ DIN         │
└──────────────┘     │ │ │ └──────────────┘
                     └─┴─┘
```

---

## Step 3: Modify TX for Broadcast

Change TX to broadcast to ALL receivers instead of one specific MAC:

```cpp
// audio_tx_broadcast.ino

#include <esp_now.h>
#include <WiFi.h>
#include <math.h>

// Broadcast address - sends to ALL ESP-NOW devices
uint8_t broadcastMAC[] = {0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF};

#define SAMPLE_RATE 44100
#define TONE_FREQ 1000
#define AMPLITUDE 20000
#define SAMPLES_PER_PACKET 60

struct AudioPacket {
  uint16_t sequence;
  uint16_t timestamp;
  uint8_t  flags;
  uint8_t  reserved;
  int16_t  samples[SAMPLES_PER_PACKET * 2];
};

AudioPacket packet;
uint16_t seqNum = 0;
float phase = 0;

uint32_t packetsSent = 0;
uint32_t lastStats = 0;

void setup() {
  Serial.begin(115200);
  Serial.println("
=== ESP-NOW BROADCAST TX ===");
  
  WiFi.mode(WIFI_STA);
  Serial.printf("TX MAC: %s
", WiFi.macAddress().c_str());
  
  if (esp_now_init() != ESP_OK) {
    Serial.println("ESP-NOW init failed!");
    while(1);
  }
  
  // Add broadcast peer
  esp_now_peer_info_t peerInfo = {};
  memcpy(peerInfo.peer_addr, broadcastMAC, 6);
  peerInfo.channel = 0;
  peerInfo.encrypt = false;
  
  if (esp_now_add_peer(&peerInfo) != ESP_OK) {
    Serial.println("Failed to add broadcast peer!");
    while(1);
  }
  
  Serial.println("Broadcasting to ALL receivers...");
  Serial.printf("Tone: %dHz @ %dHz sample rate
", TONE_FREQ, SAMPLE_RATE);
}

void loop() {
  // Generate sine wave
  float phaseInc = 2.0 * PI * TONE_FREQ / SAMPLE_RATE;
  
  for (int i = 0; i < SAMPLES_PER_PACKET; i++) {
    int16_t sample = (int16_t)(AMPLITUDE * sin(phase));
    packet.samples[i * 2] = sample;      // L
    packet.samples[i * 2 + 1] = sample;  // R
    phase += phaseInc;
    if (phase >= 2.0 * PI) phase -= 2.0 * PI;
  }
  
  packet.sequence = seqNum++;
  packet.timestamp = (uint16_t)(millis() & 0xFFFF);
  packet.flags = 0;
  
  esp_now_send(broadcastMAC, (uint8_t*)&packet, sizeof(packet));
  packetsSent++;
  
  // Stats every 5 sec
  if (millis() - lastStats > 5000) {
    Serial.printf("Packets sent: %lu (%.1f/sec)
", 
                  packetsSent, packetsSent / (millis() / 1000.0));
    lastStats = millis();
  }
  
  delayMicroseconds(1360);  // ~735 packets/sec
}
```

---

## Step 4: RX Firmware (Same for Both)

Use the same RX code from D-1.3 on BOTH receivers:

```cpp
// audio_rx_node.ino
// Flash this to BOTH RX boards

#include <esp_now.h>
#include <WiFi.h>
#include "driver/i2s.h"

#define I2S_BCK  4
#define I2S_WS   5
#define I2S_DOUT 6

#define SAMPLE_RATE 44100
#define SAMPLES_PER_PACKET 60
#define BUFFER_SIZE 8

struct AudioPacket {
  uint16_t sequence;
  uint16_t timestamp;
  uint8_t  flags;
  uint8_t  reserved;
  int16_t  samples[SAMPLES_PER_PACKET * 2];
};

AudioPacket buffer[BUFFER_SIZE];
volatile int writeIdx = 0;
volatile int readIdx = 0;
volatile int buffered = 0;
volatile bool ready = false;

uint32_t rxCount = 0;
uint32_t lostCount = 0;
uint16_t lastSeq = 0;
bool firstPkt = true;

void onReceive(const uint8_t *mac, const uint8_t *data, int len) {
  if (len != sizeof(AudioPacket)) return;
  
  AudioPacket *pkt = (AudioPacket*)data;
  
  // Track loss
  if (!firstPkt && pkt->sequence != (uint16_t)(lastSeq + 1)) {
    lostCount += (uint16_t)(pkt->sequence - lastSeq - 1);
  }
  firstPkt = false;
  lastSeq = pkt->sequence;
  rxCount++;
  
  // Buffer packet
  if (buffered < BUFFER_SIZE) {
    memcpy(&buffer[writeIdx], pkt, sizeof(AudioPacket));
    writeIdx = (writeIdx + 1) % BUFFER_SIZE;
    buffered++;
    if (buffered >= BUFFER_SIZE / 2) ready = true;
  }
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
=== ESP-NOW RX NODE ===");
  
  WiFi.mode(WIFI_STA);
  Serial.printf("RX MAC: %s
", WiFi.macAddress().c_str());
  
  setupI2S();
  
  esp_now_init();
  esp_now_register_recv_cb(onReceive);
  
  Serial.println("Waiting for broadcast...");
}

void loop() {
  static uint32_t lastStats = 0;
  
  // Play from buffer
  if (ready && buffered > 0) {
    size_t written;
    i2s_write(I2S_NUM_0, buffer[readIdx].samples,
              SAMPLES_PER_PACKET * 2 * sizeof(int16_t),
              &written, portMAX_DELAY);
    readIdx = (readIdx + 1) % BUFFER_SIZE;
    buffered--;
  }
  
  // Stats
  if (millis() - lastStats > 5000) {
    float loss = (rxCount > 0) ? 
                 (float)lostCount / (rxCount + lostCount) * 100 : 0;
    Serial.printf("RX: %lu | Lost: %lu (%.2f%%) | Buf: %d
",
                  rxCount, lostCount, loss, buffered);
    lastStats = millis();
  }
}
```

---

## Step 5: Test Setup

```
Physical Layout:

        [TX]
       /    \
      /      \
   [RX1]    [RX2]
     │        │
   [DAC]    [DAC]
     │        │
    🔊       🔊
  Speaker  Speaker
```

### Test Procedure

1. Flash `audio_tx_broadcast.ino` to TX board
2. Flash `audio_rx_node.ino` to RX1 board
3. Flash `audio_rx_node.ino` to RX2 board
4. Connect DACs and speakers to both RX boards
5. Power all three boards
6. Open Serial Monitors for all three
7. Listen for audio on BOTH speakers

---

## Step 6: Verify Results

### Expected Output

**TX Serial:**
```
=== ESP-NOW BROADCAST TX ===
TX MAC: AA:BB:CC:DD:EE:FF
Broadcasting to ALL receivers...
Tone: 1000Hz @ 44100Hz sample rate
Packets sent: 3675 (735.0/sec)
```

**RX1 Serial:**
```
=== ESP-NOW RX NODE ===
RX MAC: 11:22:33:44:55:66
Waiting for broadcast...
RX: 3670 | Lost: 2 (0.05%) | Buf: 4
```

**RX2 Serial:**
```
=== ESP-NOW RX NODE ===
RX MAC: 77:88:99:AA:BB:CC
Waiting for broadcast...
RX: 3668 | Lost: 4 (0.11%) | Buf: 4
```

### Success Criteria

| Test | Expected | Actual | Pass? |
|------|----------|--------|-------|
| TX broadcasts | ✓ | | |
| RX1 receives | ✓ | | |
| RX2 receives | ✓ | | |
| Both play audio | ✓ | | |
| Audio sounds same | ✓ | | |
| Packet loss <1% | ✓ | | |

---

## Troubleshooting

| Issue | Cause | Fix |
|-------|-------|-----|
| Only one RX works | Bad wiring | Check second DAC |
| Neither RX works | TX not broadcasting | Verify broadcast MAC |
| Audio out of sync | Different buffer states | Proceed to D-2.2 |
| High packet loss | Interference | Change WiFi channel |

---

## Completion Checklist

- [ ] Second RX wired and working
- [ ] TX broadcasting to all
- [ ] Both RX receiving packets
- [ ] Both speakers playing audio
- [ ] Packet loss acceptable on both

**→ Proceed to D-2.2 (Sync Protocol)**
