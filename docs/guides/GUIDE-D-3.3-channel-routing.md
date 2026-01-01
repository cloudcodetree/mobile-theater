# Implementation Guide: D-3.3 Channel Decode & Routing

## Overview
**Time Estimate:** 4-6 hours  
**Skill Level:** Intermediate  
**Cost:** $0 (software only)  
**Depends On:** D-3.2 complete

---

## Objective

Create software that captures 8-channel audio from USB interface, processes it, and sends each channel to the ESP32 transmitter.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      AUDIO ROUTING SOFTWARE                             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  USB Audio Interface                    Serial/SPI to ESP32             │
│  ┌─────────────────┐                    ┌─────────────────┐             │
│  │ CH0: FL ────────┼──►[Process]──────►│ Channel 0       │             │
│  │ CH1: FR ────────┼──►[Process]──────►│ Channel 1       │             │
│  │ CH2: C  ────────┼──►[Process]──────►│ Channel 2       │             │
│  │ CH3: LFE ───────┼──►[Process]──────►│ Channel 3       │             │
│  │ CH4: SL ────────┼──►[Process]──────►│ Channel 4       │             │
│  │ CH5: SR ────────┼──►[Process]──────►│ Channel 5       │             │
│  │ CH6: RL ────────┼──►[Process]──────►│ Channel 6       │             │
│  │ CH7: RR ────────┼──►[Process]──────►│ Channel 7       │             │
│  └─────────────────┘                    └─────────────────┘             │
│                                                                         │
│  [Process] = Level adjust, delay compensation, optional EQ              │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Step 1: Install Dependencies

```bash
# On Pi 5
pip3 install numpy sounddevice pyserial

# Or create virtual environment
python3 -m venv ~/pelican-env
source ~/pelican-env/bin/activate
pip install numpy sounddevice pyserial
```

---

## Step 2: Audio Router Script

Create `/home/pi/pelican-cinema/audio_router.py`:

```python
#!/usr/bin/env python3
"""
Pelican Cinema Audio Router
Captures 8-channel audio and sends to ESP32 TX
"""

import numpy as np
import sounddevice as sd
import serial
import struct
import time
import threading
from queue import Queue

# ============== CONFIGURATION ==============
SAMPLE_RATE = 48000
CHANNELS = 8
BLOCK_SIZE = 256  # Samples per block
SERIAL_PORT = '/dev/ttyUSB0'  # ESP32 TX serial port
SERIAL_BAUD = 921600

# Channel mapping: USB input channel -> Speaker
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

# Per-channel level adjustment (1.0 = unity gain)
CHANNEL_LEVELS = {
    0: 1.0,  # FL
    1: 1.0,  # FR
    2: 1.0,  # C
    3: 1.0,  # LFE
    4: 1.0,  # SL
    5: 1.0,  # SR
    6: 1.0,  # RL
    7: 1.0,  # RR
}

# ============== GLOBALS ==============
audio_queue = Queue(maxsize=32)
running = True
stats = {'blocks': 0, 'overruns': 0, 'underruns': 0}

# ============== AUDIO CALLBACK ==============
def audio_callback(indata, frames, time_info, status):
    """Called by sounddevice for each audio block"""
    global stats
    
    if status.input_overflow:
        stats['overruns'] += 1
    if status.input_underflow:
        stats['underruns'] += 1
    
    # Copy data to queue
    try:
        audio_queue.put_nowait(indata.copy())
        stats['blocks'] += 1
    except:
        stats['overruns'] += 1

# ============== SERIAL SENDER ==============
def serial_sender(ser):
    """Thread that sends audio to ESP32"""
    global running
    
    sequence = 0
    
    while running:
        try:
            # Get audio block from queue
            data = audio_queue.get(timeout=1.0)
            
            # Process each channel
            for ch in range(CHANNELS):
                # Extract channel data
                channel_data = data[:, ch]
                
                # Apply level adjustment
                channel_data = channel_data * CHANNEL_LEVELS[ch]
                
                # Convert to int16
                samples = (channel_data * 32767).astype(np.int16)
                
                # Create packet
                # Format: [SYNC][SEQ][CH][LEN][DATA...]
                packet = struct.pack('<BBHH',
                    0xAA,           # Sync byte
                    ch,             # Channel ID
                    sequence,       # Sequence number
                    len(samples)    # Sample count
                )
                packet += samples.tobytes()
                
                # Send to ESP32
                ser.write(packet)
            
            sequence = (sequence + 1) & 0xFFFF
            
        except Exception as e:
            if running:
                print(f"Sender error: {e}")

# ============== STATS PRINTER ==============
def print_stats():
    """Print statistics periodically"""
    global running
    
    while running:
        time.sleep(5)
        print(f"Blocks: {stats['blocks']} | "
              f"Overruns: {stats['overruns']} | "
              f"Underruns: {stats['underruns']} | "
              f"Queue: {audio_queue.qsize()}")

# ============== MAIN ==============
def main():
    global running
    
    print("=" * 50)
    print("Pelican Cinema Audio Router")
    print("=" * 50)
    
    # List audio devices
    print("
Audio Devices:")
    print(sd.query_devices())
    
    # Find USB audio interface
    devices = sd.query_devices()
    usb_device = None
    for i, dev in enumerate(devices):
        if 'USB' in dev['name'] and dev['max_input_channels'] >= 8:
            usb_device = i
            print(f"
Using device {i}: {dev['name']}")
            break
    
    if usb_device is None:
        print("ERROR: No 8-channel USB audio device found!")
        return
    
    # Open serial connection to ESP32
    try:
        ser = serial.Serial(SERIAL_PORT, SERIAL_BAUD, timeout=1)
        print(f"Serial connected: {SERIAL_PORT} @ {SERIAL_BAUD}")
    except Exception as e:
        print(f"Serial error: {e}")
        print("Running without serial output (test mode)")
        ser = None
    
    # Start sender thread
    if ser:
        sender_thread = threading.Thread(target=serial_sender, args=(ser,))
        sender_thread.start()
    
    # Start stats thread
    stats_thread = threading.Thread(target=print_stats)
    stats_thread.daemon = True
    stats_thread.start()
    
    # Start audio stream
    print(f"
Starting audio capture...")
    print(f"Sample rate: {SAMPLE_RATE} Hz")
    print(f"Channels: {CHANNELS}")
    print(f"Block size: {BLOCK_SIZE}")
    print("
Press Ctrl+C to stop
")
    
    try:
        with sd.InputStream(
            device=usb_device,
            samplerate=SAMPLE_RATE,
            channels=CHANNELS,
            blocksize=BLOCK_SIZE,
            dtype=np.float32,
            callback=audio_callback
        ):
            while True:
                time.sleep(0.1)
                
    except KeyboardInterrupt:
        print("
Shutting down...")
    finally:
        running = False
        if ser:
            sender_thread.join(timeout=2)
            ser.close()

if __name__ == '__main__':
    main()
```

---

## Step 3: Test Audio Capture

```bash
# Make executable
chmod +x /home/pi/pelican-cinema/audio_router.py

# Test run (without ESP32 connected)
python3 /home/pi/pelican-cinema/audio_router.py
```

### Expected Output

```
==================================================
Pelican Cinema Audio Router
==================================================

Audio Devices:
   0 HDA Intel: ALC892 (hw:0,0), ALSA (0 in, 8 out)
>  1 UMC1820: USB Audio (hw:1,0), ALSA (18 in, 18 out)

Using device 1: UMC1820: USB Audio (hw:1,0)
Serial error: [Errno 2] could not open port /dev/ttyUSB0
Running without serial output (test mode)

Starting audio capture...
Sample rate: 48000 Hz
Channels: 8
Block size: 256

Press Ctrl+C to stop

Blocks: 937 | Overruns: 0 | Underruns: 0 | Queue: 0
Blocks: 1874 | Overruns: 0 | Underruns: 0 | Queue: 0
```

---

## Step 4: Level Calibration

### Adjust Per-Channel Levels

Play test tone through each channel, measure with SPL meter, adjust `CHANNEL_LEVELS`:

```python
# Example: Center channel is too loud
CHANNEL_LEVELS = {
    0: 1.0,   # FL
    1: 1.0,   # FR
    2: 0.85,  # C - reduced 15%
    3: 1.0,   # LFE
    4: 1.0,   # SL
    5: 1.0,   # SR
    6: 1.0,   # RL
    7: 1.0,   # RR
}
```

---

## Step 5: Enable Service

```bash
# Enable auto-start
sudo systemctl enable pelican-audio.service

# Start service
sudo systemctl start pelican-audio.service

# Check status
sudo systemctl status pelican-audio.service
```

---

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| No audio devices | USB not detected | Check `lsusb`, reconnect |
| High latency | Large buffer | Decrease BLOCK_SIZE |
| Overruns | CPU overload | Increase BLOCK_SIZE |
| Wrong channels | Mapping incorrect | Verify CHANNEL_MAP |

---

## Completion Checklist

- [ ] Dependencies installed
- [ ] Audio router script created
- [ ] 8-channel capture tested
- [ ] Levels calibrated
- [ ] Service enabled

**→ Proceed to D-3.4 (ESP32 Master TX)**
