# Implementation Guide: D-3.3 7.1 Channel Decode & Routing

## Overview
**Time Estimate:** 6-8 hours | **Cost:** $0 (software) | **Depends On:** D-3.2

---

## Objective
Software to receive 8-channel audio and route to ESP32 TX.

## Channel Mapping
| Channel | Index | Speaker |
|---------|-------|---------|
| FL | 0 | Front Left |
| FR | 1 | Front Right |
| C | 2 | Center |
| LFE | 3 | Subwoofer |
| SL | 4 | Surround Left |
| SR | 5 | Surround Right |
| RL | 6 | Rear Left |
| RR | 7 | Rear Right |

## Python Audio Router
```python
#!/usr/bin/env python3
# audio_router.py

import numpy as np
import sounddevice as sd
import serial
import struct
import threading
import queue

SAMPLE_RATE = 48000
CHANNELS = 8
BLOCK_SIZE = 256
SERIAL_PORT = "/dev/ttyUSB0"  # ESP32

audio_queues = [queue.Queue() for _ in range(CHANNELS)]

def audio_callback(indata, frames, time, status):
    if status:
        print(f"Status: {status}")
    for ch in range(CHANNELS):
        audio_queues[ch].put(indata[:, ch].copy())

def esp32_sender():
    ser = serial.Serial(SERIAL_PORT, 921600)
    seq = 0
    while True:
        for ch in range(CHANNELS):
            if not audio_queues[ch].empty():
                samples = audio_queues[ch].get()
                # Pack: seq(2) + channel(1) + samples
                packet = struct.pack("<HB", seq, ch)
                packet += samples.astype(np.int16).tobytes()
                ser.write(packet)
        seq = (seq + 1) % 65536

def main():
    print("Starting 7.1 Audio Router...")
    
    # Start ESP32 sender thread
    sender = threading.Thread(target=esp32_sender, daemon=True)
    sender.start()
    
    # Start audio input
    with sd.InputStream(
        samplerate=SAMPLE_RATE,
        channels=CHANNELS,
        blocksize=BLOCK_SIZE,
        callback=audio_callback
    ):
        print("Routing... Ctrl+C to stop")
        while True:
            sd.sleep(1000)

if __name__ == "__main__":
    main()
```

## Service Setup
```bash
# Create systemd service
sudo nano /etc/systemd/system/audio-router.service

[Unit]
Description=Pelican Cinema Audio Router
After=sound.target

[Service]
ExecStart=/usr/bin/python3 /home/pi/audio_router.py
Restart=always
User=pi

[Install]
WantedBy=multi-user.target

# Enable and start
sudo systemctl enable audio-router
sudo systemctl start audio-router
```

## Testing
```bash
# Run manually first
python3 audio_router.py

# Check logs
journalctl -u audio-router -f
```

## Success Criteria
- [ ] Receives 8-channel audio input
- [ ] Separates into discrete channels
- [ ] Sends to ESP32 via serial
- [ ] Runs as system service

**→ Proceed to D-3.4**
