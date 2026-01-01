# Implementation Guide: D-1.3 ESP-NOW Audio Streaming

## Overview
**Time Estimate:** 4-6 hours  
**Skill Level:** Intermediate  
**Cost:** $15 (optional equipment)  
**Depends On:** D-1.1 and D-1.2 complete

⚠️ **CRITICAL:** This is the proof-of-concept that validates the entire project. Results here determine GO/NO-GO.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     WIRELESS AUDIO STREAMING                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  TX (Board 1)                           RX (Board 2)                    │
│  ───────────                           ───────────                      │
│                                                                         │
│  ┌─────────────┐                      ┌─────────────┐                   │
│  │ Generate    │                      │ Receive     │                   │
│  │ Audio       │                      │ Packets     │                   │
│  │ (sine wave) │                      │             │                   │
│  └──────┬──────┘                      └──────┬──────┘                   │
│         │                                    │                          │
│         ▼                                    ▼                          │
│  ┌─────────────┐     ESP-NOW          ┌─────────────┐                   │
│  │ Packetize   │    ════════►         │ Jitter      │                   │
│  │ + Sequence  │     2.4 GHz          │ Buffer      │                   │
│  │ + Timestamp │                      │             │                   │
│  └──────┬──────┘                      └──────┬──────┘                   │
│         │                                    │                          │
│         ▼                                    ▼                          │
│  ┌─────────────┐                      ┌─────────────┐                   │
│  │ ESP-NOW     │                      │ I2S Output  │                   │
│  │ Send        │                      │ to DAC      │                   │
│  └─────────────┘                      └──────┬──────┘                   │
│                                              │                          │
│                                              ▼                          │
│                                        ┌───────────┐                    │
│                                        │ PCM5102A  │──► 🔊              │
│                                        └───────────┘                    │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Step 1: Understand the Constraints

### ESP-NOW Limits
| Parameter | Limit |
|-----------|-------|
| Max payload | **250 bytes** |
| Max peers | 20 (encrypted) / 10 (unencrypted) |
| Typical latency | 1-3 ms |
| Range | 200m+ (line of sight) |

### Audio Math
```
Sample rate:     44100 Hz
Bits per sample: 16 bits (2 bytes)
Channels:        2 (stereo)
Bytes/second:    44100 × 2 × 2 = 176,400 bytes/sec

ESP-NOW payload: 250 bytes max
Usable payload:  240 bytes (leave room for headers)
Samples/packet:  240 / 4 = 60 stereo samples

Packets/second:  44100 / 60 = 735 packets/sec
Packet interval: 1.36 ms
```

---

## Step 2: Packet Structure

### Audio Packet Format

```cpp
// Must fit in 250 bytes
struct AudioPacket {
  uint16_t sequence;     // 2 bytes - packet counter
  uint16_t timestamp;    // 2 bytes - ms since boot (wraps)
  uint8_t  flags;        // 1 byte  - future use
  uint8_t  reserved;     // 1 byte  - alignment
  int16_t  samples[120]; // 240 bytes - 60 stereo samples
};
// Total: 246 bytes ✓
```

---

## Step 3: TX Firmware

### Complete TX Code

```cpp
// File: audio_stream_tx.ino

#include <esp_now.h>
#include <WiFi.h>
#include <math.h>

// ========== CONFIGURATION ==========
// REPLACE with your RX board's MAC address!
uint8_t rxMAC[] = {0xXX, 0xXX, 0xXX, 0xXX, 0xXX, 0xXX};

#define SAMPLE_RATE    44100
#define TONE_FREQ      1000
#define AMPLITUDE      20000
#define SAMPLES_PER_PACKET 60

// ========== PACKET STRUCTURE ==========
struct AudioPacket {
  uint16_t sequence;
  uint16_t timestamp;
  uint8_t  flags;
  uint8_t  reserved;
  int16_t  samples[SAMPLES_PER_PACKET * 2]; // Stereo
};

AudioPacket packet;
uint16_t sequenceNum = 0;
uint32_t packetsSent = 0;
uint32_t sendErrors = 0;
float phase = 0;

// ========== CALLBACKS ==========
void onSent(const uint8_t *mac, esp_now_send_status_t status) {
  if (status != ESP_NOW_SEND_SUCCESS) {
    sendErrors++;
  }
}

// ========== SETUP ==========
void setup() {
  Serial.begin(115200);
  Serial.println("
=== ESP-NOW Audio TX ===");
  
  // Init WiFi in station mode
  WiFi.mode(WIFI_STA);
  Serial.print("TX MAC: ");
  Serial.println(WiFi.macAddress());
  
  // Init ESP-NOW
  if (esp_now_init() != ESP_OK) {
    Serial.println("ESP-NOW init failed!");
    while(1) delay(1000);
  }
  
  esp_now_register_send_cb(onSent);
  
  // Add peer
  esp_now_peer_info_t peerInfo = {};
  memcpy(peerInfo.peer_addr, rxMAC, 6);
  peerInfo.channel = 0;
  peerInfo.encrypt = false;
  
  if (esp_now_add_peer(&peerInfo) != ESP_OK) {
    Serial.println("Failed to add peer!");
    while(1) delay(1000);
  }
  
  Serial.println("TX Ready - Streaming audio...");
  Serial.printf("Tone: %d Hz, Sample Rate: %d Hz
", TONE_FREQ, SAMPLE_RATE);
  Serial.printf("Packets/sec: %d
", SAMPLE_RATE / SAMPLES_PER_PACKET);
}

// ========== MAIN LOOP ==========
void loop() {
  static uint32_t lastStats = 0;
  
  // Generate sine wave samples
  float phaseIncrement = 2.0 * PI * TONE_FREQ / SAMPLE_RATE;
  
  for (int i = 0; i < SAMPLES_PER_PACKET; i++) {
    int16_t sample = (int16_t)(AMPLITUDE * sin(phase));
    packet.samples[i * 2] = sample;     // Left
    packet.samples[i * 2 + 1] = sample; // Right
    phase += phaseIncrement;
    if (phase >= 2.0 * PI) phase -= 2.0 * PI;
  }
  
  // Fill packet header
  packet.sequence = sequenceNum++;
  packet.timestamp = (uint16_t)(millis() & 0xFFFF);
  packet.flags = 0;
  packet.reserved = 0;
  
  // Send packet
  esp_err_t result = esp_now_send(rxMAC, (uint8_t*)&packet, sizeof(packet));
  if (result == ESP_OK) {
    packetsSent++;
  }
  
  // Print stats every 5 seconds
  if (millis() - lastStats > 5000) {
    float errorRate = (float)sendErrors / packetsSent * 100;
    Serial.printf("Sent: %lu | Errors: %lu (%.2f%%)
", 
                  packetsSent, sendErrors, errorRate);
    lastStats = millis();
  }
  
  // Timing: send packet every 1.36ms for 44.1kHz
  delayMicroseconds(1360);
}
```

---

## Step 4: RX Firmware with Jitter Buffer

### Complete RX Code

```cpp
// File: audio_stream_rx.ino

#include <esp_now.h>
#include <WiFi.h>
#include "driver/i2s.h"

// ========== I2S PINS ==========
#define I2S_BCK_PIN  4
#define I2S_WS_PIN   5
#define I2S_DATA_PIN 6

// ========== CONFIGURATION ==========
#define SAMPLE_RATE        44100
#define SAMPLES_PER_PACKET 60
#define JITTER_BUFFER_SIZE 8  // Number of packets to buffer

// ========== PACKET STRUCTURE ==========
struct AudioPacket {
  uint16_t sequence;
  uint16_t timestamp;
  uint8_t  flags;
  uint8_t  reserved;
  int16_t  samples[SAMPLES_PER_PACKET * 2];
};

// ========== JITTER BUFFER ==========
AudioPacket jitterBuffer[JITTER_BUFFER_SIZE];
volatile int writeIndex = 0;
volatile int readIndex = 0;
volatile int bufferedPackets = 0;
volatile bool bufferReady = false;

// ========== STATS ==========
uint32_t packetsReceived = 0;
uint32_t packetsLost = 0;
uint16_t lastSequence = 0;
bool firstPacket = true;

// ========== ESP-NOW CALLBACK ==========
void onReceive(const uint8_t *mac, const uint8_t *data, int len) {
  if (len != sizeof(AudioPacket)) return;
  
  AudioPacket *pkt = (AudioPacket*)data;
  
  // Track packet loss
  if (!firstPacket) {
    uint16_t expected = lastSequence + 1;
    if (pkt->sequence != expected) {
      uint16_t lost = pkt->sequence - expected;
      packetsLost += lost;
    }
  }
  firstPacket = false;
  lastSequence = pkt->sequence;
  packetsReceived++;
  
  // Add to jitter buffer
  if (bufferedPackets < JITTER_BUFFER_SIZE) {
    memcpy(&jitterBuffer[writeIndex], pkt, sizeof(AudioPacket));
    writeIndex = (writeIndex + 1) % JITTER_BUFFER_SIZE;
    bufferedPackets++;
    
    // Start playback when buffer is half full
    if (bufferedPackets >= JITTER_BUFFER_SIZE / 2) {
      bufferReady = true;
    }
  }
}

// ========== I2S SETUP ==========
void setupI2S() {
  i2s_config_t i2s_config = {
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
  
  i2s_pin_config_t pin_config = {
    .bck_io_num = I2S_BCK_PIN,
    .ws_io_num = I2S_WS_PIN,
    .data_out_num = I2S_DATA_PIN,
    .data_in_num = I2S_PIN_NO_CHANGE
  };
  
  i2s_driver_install(I2S_NUM_0, &i2s_config, 0, NULL);
  i2s_set_pin(I2S_NUM_0, &pin_config);
}

// ========== SETUP ==========
void setup() {
  Serial.begin(115200);
  Serial.println("
=== ESP-NOW Audio RX ===");
  
  // Init WiFi
  WiFi.mode(WIFI_STA);
  Serial.print("RX MAC: ");
  Serial.println(WiFi.macAddress());
  
  // Init I2S
  setupI2S();
  Serial.println("I2S initialized");
  
  // Init ESP-NOW
  if (esp_now_init() != ESP_OK) {
    Serial.println("ESP-NOW init failed!");
    while(1) delay(1000);
  }
  
  esp_now_register_recv_cb(onReceive);
  
  Serial.println("RX Ready - Waiting for audio...");
  Serial.printf("Jitter buffer: %d packets
", JITTER_BUFFER_SIZE);
}

// ========== MAIN LOOP ==========
void loop() {
  static uint32_t lastStats = 0;
  
  // Play audio from jitter buffer
  if (bufferReady && bufferedPackets > 0) {
    AudioPacket *pkt = &jitterBuffer[readIndex];
    
    size_t bytesWritten;
    i2s_write(I2S_NUM_0, pkt->samples, 
              SAMPLES_PER_PACKET * 2 * sizeof(int16_t),
              &bytesWritten, portMAX_DELAY);
    
    readIndex = (readIndex + 1) % JITTER_BUFFER_SIZE;
    bufferedPackets--;
  }
  
  // Print stats every 5 seconds
  if (millis() - lastStats > 5000) {
    float lossRate = 0;
    if (packetsReceived > 0) {
      lossRate = (float)packetsLost / (packetsReceived + packetsLost) * 100;
    }
    Serial.printf("Received: %lu | Lost: %lu (%.2f%%) | Buffer: %d
",
                  packetsReceived, packetsLost, lossRate, bufferedPackets);
    lastStats = millis();
  }
}
```

---

## Step 5: Testing Procedure

### 5.1 Hardware Setup

```
TX Board (Board 1)          RX Board (Board 2)
┌───────────────┐           ┌───────────────┐
│   ESP32-S3    │           │   ESP32-S3    │
│               │           │               │
│           USB │           │      GPIO4 ───┼──► BCK
│               │ ~~~2.4GHz~~│      GPIO5 ───┼──► LCK  ──► PCM5102A
│               │           │      GPIO6 ───┼──► DIN
└───────────────┘           │               │
                            │           USB │
                            └───────────────┘
                                    │
                                    ▼
                               [Headphones]
```

### 5.2 Test Sequence

1. **Flash RX first** (so it's ready to receive)
2. **Open RX Serial Monitor**
3. **Copy RX MAC address**
4. **Update TX code** with RX MAC
5. **Flash TX**
6. **Open TX Serial Monitor** (second window)
7. **Connect headphones** to PCM5102A
8. **Listen** for audio tone

### 5.3 Expected Serial Output

**TX:**
```
=== ESP-NOW Audio TX ===
TX MAC: AA:BB:CC:DD:EE:FF
TX Ready - Streaming audio...
Tone: 1000 Hz, Sample Rate: 44100 Hz
Packets/sec: 735
Sent: 3675 | Errors: 0 (0.00%)
Sent: 7350 | Errors: 2 (0.03%)
```

**RX:**
```
=== ESP-NOW Audio RX ===
RX MAC: 11:22:33:44:55:66
I2S initialized
RX Ready - Waiting for audio...
Jitter buffer: 8 packets
Received: 3670 | Lost: 3 (0.08%) | Buffer: 4
Received: 7342 | Lost: 5 (0.07%) | Buffer: 4
```

---

## Step 6: Measure Latency

### Method 1: Oscilloscope (Accurate)

1. Generate tone on TX that toggles every 100ms
2. Measure time between TX GPIO toggle and RX audio output
3. Requires oscilloscope with 2 channels

### Method 2: Audible Test (Approximate)

1. Modify TX to play tones with gaps
2. Estimate latency subjectively (<50ms is imperceptible for most)

### Method 3: Loopback (If you have audio input)

1. Connect TX audio output to RX audio input
2. Measure round-trip time / 2

### Expected Latency Breakdown

| Stage | Time |
|-------|------|
| TX: Generate + Packetize | <1ms |
| ESP-NOW transmission | 1-3ms |
| RX: Receive callback | <1ms |
| Jitter buffer (4 packets) | ~5.4ms |
| I2S DMA buffer | ~3ms |
| **Total** | **~10-15ms** |

---

## Step 7: Range Testing

Test at increasing distances:

| Distance | Expected | Your Result |
|----------|----------|-------------|
| 1 meter | 100% packets | |
| 5 meters | 99%+ packets | |
| 10 meters | 98%+ packets | |
| 20 meters | 95%+ packets | |
| 50 meters (LOS) | 90%+ packets | |

---

## Step 8: GO/NO-GO Decision

### Success Criteria

| Metric | Target | Actual | Pass? |
|--------|--------|--------|-------|
| Audio plays wirelessly | Yes | | |
| Latency | <30ms | | |
| Packet loss | <1% | | |
| Range (LOS) | >10m | | |
| Audio quality | No constant artifacts | | |

### Decision Matrix

✅ **GO to Phase 2** if ALL criteria pass

🔄 **ITERATE** if:
- Latency 30-50ms (try smaller jitter buffer)
- Packet loss 1-3% (try different WiFi channel)
- Range 5-10m (acceptable for indoor use)

❌ **NO-GO / PIVOT** if:
- Latency >50ms consistently
- Packet loss >5%
- Range <5m
- Constant audio artifacts

### Pivot Options
1. **WiFi UDP Multicast** - Higher bandwidth, more latency
2. **nRF24L01+** - Lower latency, more complex
3. **Commercial modules** - WiSA, proprietary wireless

---

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| No audio | MAC address wrong | Double-check RX MAC in TX code |
| | WiFi not in STA mode | Ensure WiFi.mode(WIFI_STA) |
| Choppy audio | Buffer underrun | Increase JITTER_BUFFER_SIZE |
| | High packet loss | Move closer, check interference |
| High latency | Buffer too large | Decrease JITTER_BUFFER_SIZE |
| Clicks/pops | Buffer overflow | Increase buffer or reduce rate |
| Distorted | Amplitude too high | Reduce AMPLITUDE |

---

## Completion Checklist

- [ ] TX streaming packets successfully
- [ ] RX receiving and playing audio
- [ ] Latency measured: ______ ms
- [ ] Packet loss measured: ______ %
- [ ] Range tested: ______ meters
- [ ] GO/NO-GO decision made: ______

**→ If GO, close Issue #3 and Issue #4 (checkpoint), proceed to Phase 2**
