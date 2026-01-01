# Implementation Guide: Phase 3 - Command Module

## Overview
**Phase Budget:** $620  
**Time Estimate:** 3-4 weeks  
**Objective:** Build central brain with HDMI input, 7.1 decode, and wireless TX

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         COMMAND MODULE                                  │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                 │
│  │   HDMI      │    │   Audio     │    │   Pi 5      │                 │
│  │   Switch    │───►│   Extract   │───►│   Decode    │                 │
│  │   4×1       │    │   7.1 PCM   │    │   Route     │                 │
│  └─────────────┘    └─────────────┘    └──────┬──────┘                 │
│        ▲                                       │                        │
│        │                                       │ I2S / SPI              │
│   [PS5][Xbox]                                  ▼                        │
│   [ATV][Blu-ray]                        ┌─────────────┐                │
│                                         │   ESP32     │                │
│                          HDMI Out ◄─────│   TX        │───► Wireless   │
│                          to Projector   └─────────────┘      Broadcast │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## D-3.1: HDMI Switch + Audio Extractor

### Shopping List

| Item | Model | Cost | Link |
|------|-------|------|------|
| HDMI Switch 4×1 | SGEYR 4K@60Hz | $40 | [Amazon](https://amazon.com/dp/B07RK6BNFT) |
| Audio Extractor | Monoprice Blackbird | $80 | [Amazon](https://amazon.com/dp/B01B5FR722) |
| HDMI Cables (×5) | 6ft High Speed | $30 | Amazon |

### Wiring

```
[PS5]─────HDMI─────┐
[Xbox]────HDMI─────┤
[Apple TV]─HDMI────┼──►[4×1 Switch]──►[Audio Extractor]──┬──►[Projector]
[Blu-ray]─HDMI─────┘                         │           │
                                             │           └──HDMI Out
                                             ▼
                                        [7.1 PCM Out]
                                        (8× RCA or HDMI ARC)
```

### Verification
- [ ] All 4 sources switch correctly
- [ ] Video passes to projector
- [ ] Audio extractor outputs 7.1 PCM
- [ ] No audio/video sync issues

---

## D-3.2: Raspberry Pi 5 Setup

### Parts

| Item | Cost |
|------|------|
| Raspberry Pi 5 (8GB) | $80 |
| Pi 5 Active Cooler | $10 |
| 27W USB-C Power Supply | $15 |
| MicroSD 64GB | $15 |
| USB Audio Interface (8ch) | $100 |

### Recommended Audio Interfaces
- **Behringer UMC1820** (~$250) - 8 inputs, proven
- **Focusrite Scarlett 18i8** (~$350) - Better quality
- **MOTU M4** (~$200) - 4 inputs (need 2)

### OS Setup

```bash
# Flash Raspberry Pi OS Lite (64-bit)
# Use Raspberry Pi Imager

# First boot - enable SSH, set hostname
sudo raspi-config

# Update system
sudo apt update && sudo apt upgrade -y

# Install audio tools
sudo apt install -y alsa-utils pulseaudio jackd2

# Low-latency kernel (optional but recommended)
sudo apt install -y linux-image-rt-arm64
```

### ALSA Configuration

```bash
# List audio devices
arecord -l
aplay -l

# Test recording
arecord -D hw:1,0 -f S16_LE -r 48000 -c 8 test.wav

# Test playback
aplay -D hw:1,0 test.wav
```

---

## D-3.3: Channel Routing Software

### Python Audio Router

```python
#!/usr/bin/env python3
# audio_router.py

import numpy as np
import sounddevice as sd
import queue

SAMPLE_RATE = 48000
CHANNELS = 8
BLOCK_SIZE = 256

# Channel mapping: input channel -> speaker
CHANNEL_MAP = {
    0: 'FL',   # Front Left
    1: 'FR',   # Front Right
    2: 'C',    # Center
    3: 'LFE',  # Subwoofer
    4: 'SL',   # Surround Left
    5: 'SR',   # Surround Right
    6: 'RL',   # Rear Left
    7: 'RR',   # Rear Right
}

audio_queue = queue.Queue()

def audio_callback(indata, frames, time, status):
    if status:
        print(f"Audio status: {status}")
    audio_queue.put(indata.copy())

def main():
    print(f"Starting 7.1 audio router...")
    print(f"Sample rate: {SAMPLE_RATE}, Channels: {CHANNELS}")
    
    # List devices
    print(sd.query_devices())
    
    # Start input stream
    with sd.InputStream(
        samplerate=SAMPLE_RATE,
        channels=CHANNELS,
        blocksize=BLOCK_SIZE,
        callback=audio_callback
    ):
        print("Routing audio... Press Ctrl+C to stop")
        while True:
            data = audio_queue.get()
            # Send each channel to ESP32
            for ch in range(CHANNELS):
                channel_data = data[:, ch]
                send_to_esp32(ch, channel_data)

if __name__ == '__main__':
    main()
```

---

## D-3.4: ESP32 Master TX

### Multi-Channel TX Code

```cpp
// master_tx.ino
// Receives 8 channels from Pi, broadcasts to satellites

#include <esp_now.h>
#include <WiFi.h>
#include <SPI.h>

#define NUM_CHANNELS 8
#define SAMPLES_PER_PACKET 60

uint8_t broadcastMAC[] = {0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF};

struct ChannelPacket {
  uint16_t sequence;
  uint16_t timestamp;
  uint8_t  channelId;    // 0-7 for each speaker
  uint8_t  reserved;
  int16_t  samples[SAMPLES_PER_PACKET];
};

void setup() {
  Serial.begin(115200);
  WiFi.mode(WIFI_STA);
  esp_now_init();
  
  esp_now_peer_info_t peer = {};
  memcpy(peer.peer_addr, broadcastMAC, 6);
  esp_now_add_peer(&peer);
  
  // Setup SPI to receive from Pi
  // ... SPI slave config ...
}

void loop() {
  // Receive audio from Pi via SPI
  // Packetize each channel
  // Broadcast with channel ID
  
  for (int ch = 0; ch < NUM_CHANNELS; ch++) {
    ChannelPacket pkt;
    pkt.channelId = ch;
    // ... fill samples from SPI buffer ...
    esp_now_send(broadcastMAC, (uint8_t*)&pkt, sizeof(pkt));
  }
}
```

### Satellite RX (Channel Filter)

```cpp
// satellite_rx.ino
// Each satellite only plays its assigned channel

#define MY_CHANNEL 0  // Set per satellite: 0=FL, 1=FR, etc.

void onReceive(const uint8_t *mac, const uint8_t *data, int len) {
  ChannelPacket *pkt = (ChannelPacket*)data;
  
  // Only process packets for my channel
  if (pkt->channelId != MY_CHANNEL) return;
  
  // Add to jitter buffer and play
  addToBuffer(pkt);
}
```

---

## D-3.5: Command Module Enclosure

### Requirements
- Ventilation for Pi 5 + audio interface
- Front panel: HDMI inputs, power switch, status LEDs
- Rear panel: HDMI output, antenna for ESP32
- Dimensions: ~12" × 10" × 4"

### Options
1. **Project Box** - Modify plastic enclosure ($30)
2. **3D Print** - Custom design ($50 filament)
3. **Rack Mount** - 2U rack case ($80)

---

## Phase 3 Checkpoint

- [ ] All 4 HDMI sources work
- [ ] 7.1 audio extracted successfully
- [ ] Pi 5 processes all 8 channels
- [ ] ESP32 broadcasts to satellites
- [ ] End-to-end latency acceptable
- [ ] Enclosure complete

**→ If GO, proceed to Phase 4 (Prototype Pod)**
