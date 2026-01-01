# Implementation Guide: D-3.4 ESP32 Master TX Integration

## Overview
**Time Estimate:** 4-6 hours  
**Skill Level:** Intermediate  
**Cost:** $40  
**Depends On:** D-3.3 complete

---

## Objective

Integrate ESP32-S3 as master transmitter, receiving 8-channel audio from Pi 5 via serial/USB and broadcasting to all satellite speakers via ESP-NOW.

---

## Shopping List

| Item | Cost | Notes |
|------|------|-------|
| ESP32-S3-DevKitC-1 | $10 | Master TX |
| USB-C Cable | $5 | Data cable to Pi |
| Logic Level Shifter | $5 | Optional, for GPIO |
| Proto Board | $10 | For permanent mount |
| Antenna (2.4GHz) | $10 | External for range |

**Total:** $40

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      MASTER TX SYSTEM                                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   Raspberry Pi 5                         ESP32-S3 Master TX             │
│   ┌─────────────────┐                   ┌─────────────────┐             │
│   │                 │                   │                 │             │
│   │  Audio Router   │───USB Serial────►│  Receive        │             │
│   │  (Python)       │    921600 baud   │  8 channels     │             │
│   │                 │                   │                 │             │
│   └─────────────────┘                   │       │         │             │
│                                         │       ▼         │             │
│                                         │  ┌──────────┐   │             │
│                                         │  │ Packetize│   │             │
│                                         │  │ + Sync   │   │             │
│                                         │  └────┬─────┘   │             │
│                                         │       │         │             │
│                                         │       ▼         │             │
│                                         │  ┌──────────┐   │             │
│                                         │  │ ESP-NOW  │───┼──► 📡       │
│                                         │  │ Broadcast│   │  Wireless   │
│                                         │  └──────────┘   │             │
│                                         │                 │             │
│                                         └─────────────────┘             │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Step 1: Connect ESP32 to Pi

### USB Connection

Simply connect ESP32-S3 to Pi 5 via USB-C. The ESP32 will appear as a serial port.

```bash
# On Pi, check for device
ls /dev/ttyUSB* /dev/ttyACM*

# Should show something like:
# /dev/ttyUSB0
# or
# /dev/ttyACM0
```

### Alternative: Hardware Serial (Faster)

For lower latency, use direct UART:

```
Pi 5 GPIO           ESP32-S3
─────────           ────────
GPIO14 (TX) ───────► GPIO44 (RX)
GPIO15 (RX) ◄─────── GPIO43 (TX)
GND ──────────────── GND
```

---

## Step 2: Master TX Firmware

```cpp
// master_tx.ino
// Receives audio from Pi via serial, broadcasts via ESP-NOW

#include <esp_now.h>
#include <WiFi.h>

// ============== CONFIGURATION ==============
#define SERIAL_BAUD 921600
#define NUM_CHANNELS 8
#define SAMPLES_PER_PACKET 60

// Broadcast to all receivers
uint8_t broadcastMAC[] = {0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF};

// ============== PACKET STRUCTURES ==============
#define PKT_TYPE_AUDIO 0x01
#define PKT_TYPE_SYNC  0x02

struct AudioPacket {
  uint8_t  type;         // PKT_TYPE_AUDIO
  uint8_t  channelId;    // 0-7
  uint16_t sequence;
  uint32_t playTime;     // When to play (master clock)
  int16_t  samples[SAMPLES_PER_PACKET];
};

struct SyncBeacon {
  uint8_t  type;         // PKT_TYPE_SYNC
  uint8_t  reserved;
  uint16_t beaconSeq;
  uint32_t masterTime;
};

// ============== GLOBALS ==============
AudioPacket audioPkt;
SyncBeacon syncPkt;
uint16_t audioSeq[NUM_CHANNELS] = {0};
uint16_t syncSeq = 0;

// Serial receive buffer
uint8_t rxBuffer[1024];
int rxIndex = 0;

// Stats
uint32_t packetsReceived = 0;
uint32_t packetsSent = 0;
uint32_t lastStats = 0;

// ============== SETUP ==============
void setup() {
  // Debug serial (USB)
  Serial.begin(115200);
  Serial.println("
=== MASTER TX ===");
  
  // Audio data serial (or use Serial for both via USB)
  Serial1.begin(SERIAL_BAUD, SERIAL_8N1, 44, 43);  // RX=44, TX=43
  // Or just use Serial for USB connection:
  // Serial.begin(SERIAL_BAUD);
  
  // Init WiFi for ESP-NOW
  WiFi.mode(WIFI_STA);
  Serial.printf("TX MAC: %s
", WiFi.macAddress().c_str());
  
  // Init ESP-NOW
  if (esp_now_init() != ESP_OK) {
    Serial.println("ESP-NOW init failed!");
    while(1);
  }
  
  // Add broadcast peer
  esp_now_peer_info_t peer = {};
  memcpy(peer.peer_addr, broadcastMAC, 6);
  peer.channel = 0;
  peer.encrypt = false;
  esp_now_add_peer(&peer);
  
  // Init packet types
  audioPkt.type = PKT_TYPE_AUDIO;
  syncPkt.type = PKT_TYPE_SYNC;
  
  Serial.println("Waiting for audio from Pi...");
}

// ============== SEND SYNC BEACON ==============
void sendSyncBeacon() {
  syncPkt.beaconSeq = syncSeq++;
  syncPkt.masterTime = micros();
  esp_now_send(broadcastMAC, (uint8_t*)&syncPkt, sizeof(syncPkt));
}

// ============== PROCESS SERIAL DATA ==============
void processSerialData() {
  // Read available data
  while (Serial.available()) {  // or Serial1.available()
    uint8_t b = Serial.read();
    rxBuffer[rxIndex++] = b;
    
    // Check for sync byte at start
    if (rxIndex == 1 && rxBuffer[0] != 0xAA) {
      rxIndex = 0;  // Reset, not a valid packet
      continue;
    }
    
    // Check if we have header (SYNC + CH + SEQ + LEN = 6 bytes)
    if (rxIndex >= 6) {
      uint8_t channelId = rxBuffer[1];
      uint16_t seq = rxBuffer[2] | (rxBuffer[3] << 8);
      uint16_t sampleCount = rxBuffer[4] | (rxBuffer[5] << 8);
      
      // Calculate expected packet size
      int expectedSize = 6 + (sampleCount * 2);
      
      if (rxIndex >= expectedSize) {
        // Complete packet received
        packetsReceived++;
        
        // Extract samples
        int16_t* samples = (int16_t*)&rxBuffer[6];
        
        // Build and send ESP-NOW packet
        audioPkt.channelId = channelId;
        audioPkt.sequence = audioSeq[channelId]++;
        audioPkt.playTime = micros() + 15000;  // 15ms in future
        
        int samplesToSend = min((int)sampleCount, SAMPLES_PER_PACKET);
        memcpy(audioPkt.samples, samples, samplesToSend * sizeof(int16_t));
        
        esp_now_send(broadcastMAC, (uint8_t*)&audioPkt, 
                     8 + samplesToSend * sizeof(int16_t));
        packetsSent++;
        
        // Send sync beacon every 100 packets
        if (packetsSent % 100 == 0) {
          sendSyncBeacon();
        }
        
        // Reset buffer
        rxIndex = 0;
      }
    }
    
    // Prevent buffer overflow
    if (rxIndex >= sizeof(rxBuffer)) {
      rxIndex = 0;
    }
  }
}

// ============== MAIN LOOP ==============
void loop() {
  processSerialData();
  
  // Print stats
  if (millis() - lastStats > 5000) {
    Serial.printf("RX: %lu | TX: %lu | Sync: %u
",
                  packetsReceived, packetsSent, syncSeq);
    lastStats = millis();
  }
}
```

---

## Step 3: Update Pi Audio Router

Update the Python script to send to the correct serial port:

```python
# In audio_router.py, update:

SERIAL_PORT = '/dev/ttyACM0'  # or /dev/ttyUSB0
SERIAL_BAUD = 921600
```

---

## Step 4: Test Integration

### 1. Flash ESP32

```bash
# On your development machine
arduino-cli compile --fqbn esp32:esp32:esp32s3 master_tx.ino
arduino-cli upload -p /dev/ttyUSB0 --fqbn esp32:esp32:esp32s3 master_tx.ino
```

### 2. Connect to Pi

```bash
# Connect ESP32 to Pi 5 via USB
# Check detection
ls /dev/ttyACM*
```

### 3. Start Audio Router

```bash
# On Pi
python3 /home/pi/pelican-cinema/audio_router.py
```

### 4. Monitor ESP32

```bash
# Open serial monitor on Pi
screen /dev/ttyACM0 115200

# Should see:
# === MASTER TX ===
# TX MAC: AA:BB:CC:DD:EE:FF
# Waiting for audio from Pi...
# RX: 735 | TX: 735 | Sync: 7
```

---

## Step 5: Test with Satellite RX

1. Power on a satellite RX (from Phase 2)
2. Verify it receives packets from Master TX
3. Audio should play through satellite speaker

---

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| No serial data | Wrong port | Check `ls /dev/tty*` |
| Garbled data | Baud mismatch | Verify both ends at 921600 |
| ESP-NOW fails | WiFi not init | Check WiFi.mode(WIFI_STA) |
| High latency | Buffer too large | Reduce playTime offset |

---

## Completion Checklist

- [ ] ESP32 connected to Pi via USB
- [ ] Serial communication working
- [ ] ESP-NOW broadcasting
- [ ] Sync beacons sending
- [ ] Satellite RX receives audio

**→ Proceed to D-3.5 (Command Module Enclosure)**
