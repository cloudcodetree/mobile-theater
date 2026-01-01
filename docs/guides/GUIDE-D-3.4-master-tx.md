# Implementation Guide: D-3.4 ESP32 Master TX Integration

## Overview
**Time Estimate:** 4-6 hours | **Cost:** $40 | **Depends On:** D-3.3

---

## Objective
ESP32 receives 8-channel audio from Pi and broadcasts to satellites.

## Parts
| Item | Cost |
|------|------|
| ESP32-S3-DevKitC-1 | $10 |
| Level Shifter (3.3V-5V) | $5 |
| Custom PCB (optional) | $25 |

## Connection Diagram
```
Raspberry Pi 5              ESP32-S3 Master TX
┌────────────┐              ┌────────────┐
│            │              │            │
│  USB ──────┼──────────────┼─► USB      │
│  (Serial)  │   921600     │  (Serial)  │
│            │   baud       │            │
│            │              │     WiFi ══╪══► Broadcast
└────────────┘              │            │    to Satellites
                            └────────────┘
```

## Master TX Firmware
```cpp
// master_tx.ino

#include <esp_now.h>
#include <WiFi.h>

#define NUM_CHANNELS 8
#define SAMPLES_PER_PACKET 60

uint8_t broadcastMAC[] = {0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF};

struct ChannelPacket {
    uint16_t sequence;
    uint16_t timestamp;
    uint8_t  channelId;
    uint8_t  flags;
    int16_t  samples[SAMPLES_PER_PACKET];
};

uint8_t serialBuffer[512];
uint16_t globalSeq = 0;

void setup() {
    Serial.begin(921600);  // High speed for 8 channels
    
    WiFi.mode(WIFI_STA);
    esp_now_init();
    
    esp_now_peer_info_t peer = {};
    memcpy(peer.peer_addr, broadcastMAC, 6);
    esp_now_add_peer(&peer);
}

void loop() {
    // Read packet from Pi: seq(2) + channel(1) + samples(120)
    if (Serial.available() >= 123) {
        Serial.readBytes(serialBuffer, 123);
        
        ChannelPacket pkt;
        pkt.sequence = globalSeq++;
        pkt.timestamp = (uint16_t)(millis() & 0xFFFF);
        pkt.channelId = serialBuffer[2];
        pkt.flags = 0;
        memcpy(pkt.samples, &serialBuffer[3], 120);
        
        esp_now_send(broadcastMAC, (uint8_t*)&pkt, sizeof(pkt));
    }
    
    // Send sync beacon periodically
    static uint32_t lastSync = 0;
    if (millis() - lastSync > 100) {
        sendSyncBeacon();
        lastSync = millis();
    }
}

void sendSyncBeacon() {
    uint8_t beacon[6];
    beacon[0] = 0x01;  // Sync type
    uint32_t time = micros();
    memcpy(&beacon[1], &time, 4);
    beacon[5] = 0;
    esp_now_send(broadcastMAC, beacon, 6);
}
```

## Updated Satellite RX
```cpp
// Satellite filters by channel ID
#define MY_CHANNEL 0  // Set per satellite

void onReceive(const uint8_t *mac, const uint8_t *data, int len) {
    // Check for sync beacon
    if (len == 6 && data[0] == 0x01) {
        handleSyncBeacon(data);
        return;
    }
    
    // Check channel
    ChannelPacket *pkt = (ChannelPacket*)data;
    if (pkt->channelId != MY_CHANNEL) return;
    
    // Add to buffer and play
    addToJitterBuffer(pkt);
}
```

## Testing
1. Connect ESP32 to Pi via USB
2. Run audio_router.py on Pi
3. Monitor ESP32 serial output
4. Check satellites receive correct channels

## Success Criteria
- [ ] ESP32 receives data from Pi
- [ ] Broadcasts to all satellites
- [ ] Each satellite plays correct channel
- [ ] Sync beacons transmitted

**→ Proceed to D-3.5**
