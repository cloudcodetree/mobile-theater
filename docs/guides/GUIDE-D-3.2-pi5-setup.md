# Implementation Guide: D-3.2 Raspberry Pi 5 Audio Platform

## Overview
**Time Estimate:** 4-6 hours  
**Skill Level:** Intermediate  
**Cost:** $220  
**Depends On:** D-3.1 complete

---

## Shopping List

| Item | Model | Cost | Link |
|------|-------|------|------|
| Raspberry Pi 5 | 8GB RAM | $80 | [RPi Store](https://raspberrypi.com) |
| Active Cooler | Official Pi 5 | $10 | RPi Store |
| Power Supply | 27W USB-C | $15 | RPi Store |
| MicroSD | 64GB A2 | $15 | Amazon |
| USB Audio Interface | 8+ channels | $100 | See below |

**Total:** $220

---

## Step 1: Choose Audio Interface

### Requirement: 8 Analog Inputs

You need to capture 8 channels (7.1) from the audio extractor.

### Option A: Single 8-Channel Interface

| Model | Inputs | Cost | Notes |
|-------|--------|------|-------|
| Behringer UMC1820 | 8 | $250 | Proven, USB 2.0 |
| Focusrite Scarlett 18i8 | 8 | $350 | Better preamps |
| MOTU M4 | 4 | $200 | Need 2 units |

### Option B: Dual 4-Channel Interfaces

| Model | Inputs | Cost | Notes |
|-------|--------|------|-------|
| Behringer UMC404HD | 4 | $100 | Need 2 = $200 |
| Focusrite Scarlett 4i4 | 4 | $180 | Need 2 = $360 |

### Option C: Budget Multi-Channel

| Model | Inputs | Cost | Notes |
|-------|--------|------|-------|
| TASCAM US-16x08 | 16 | $300 | Overkill but works |
| Zoom LiveTrak L-8 | 8 | $300 | Mixer+interface |

### Recommendation

**Behringer UMC1820** - Best value for 8 channels, well-supported on Linux.

---

## Step 2: Flash Raspberry Pi OS

### Download

1. Get **Raspberry Pi Imager**: https://raspberrypi.com/software/
2. Select: **Raspberry Pi OS Lite (64-bit)** (no desktop needed)

### Configure Before Flashing

Click gear icon in Imager:

```
☑ Set hostname: pelican-pi
☑ Enable SSH: Use password authentication
☑ Set username: pi
☑ Set password: [your password]
☑ Configure WiFi: [your network]  (optional, can use ethernet)
☑ Set locale: [your timezone]
```

### Flash

1. Insert SD card
2. Select SD card as target
3. Click WRITE
4. Wait for completion

---

## Step 3: First Boot Setup

### Connect

```bash
# SSH into Pi (find IP from router or use hostname)
ssh pi@pelican-pi.local
# or
ssh pi@192.168.x.x
```

### Update System

```bash
sudo apt update && sudo apt full-upgrade -y
sudo reboot
```

### Install Audio Tools

```bash
# Reconnect after reboot
sudo apt install -y \
  alsa-utils \
  pulseaudio \
  pulseaudio-utils \
  jackd2 \
  libjack-jackd2-dev \
  python3-pip \
  python3-numpy \
  git
```

### Install Low-Latency Kernel (Optional but Recommended)

```bash
# Check available kernels
apt-cache search linux-image | grep rt

# Install RT kernel if available
sudo apt install -y linux-image-rt-arm64

# Reboot to use new kernel
sudo reboot
```

---

## Step 4: Configure USB Audio Interface

### Connect Interface

1. Connect USB audio interface to Pi 5 USB 3.0 port
2. Connect power to interface if required

### Verify Detection

```bash
# List audio devices
arecord -l

# Expected output:
# card 1: UMC1820 [UMC1820], device 0: USB Audio [USB Audio]
#   Subdevices: 1/1
#   Subdevice #0: subdevice #0
```

### Check Supported Formats

```bash
# Show supported formats
cat /proc/asound/card1/stream0

# Look for:
# Format: S24_3LE or S16_LE
# Rates: 44100, 48000
# Channels: 8 or more
```

---

## Step 5: Test Audio Capture

### Record Test

```bash
# Record 8 channels, 48kHz, 16-bit, 5 seconds
arecord -D hw:1,0 -f S16_LE -r 48000 -c 8 -d 5 test.wav

# Play back through headphones (plug into interface)
aplay -D hw:1,0 test.wav
```

### Monitor Levels

```bash
# Install level meter
sudo apt install -y alsa-tools alsamixergui

# Run CLI mixer
alsamixer -c 1

# Use arrow keys to adjust levels
# M to mute/unmute
```

---

## Step 6: Configure for Low Latency

### ALSA Configuration

Create `/etc/asound.conf`:

```bash
sudo nano /etc/asound.conf
```

Add:

```
# Default to USB interface
defaults.pcm.card 1
defaults.ctl.card 1

# Low-latency settings
pcm.lowlatency {
    type hw
    card 1
    device 0
    rate 48000
    format S16_LE
    channels 8
}
```

### Disable PulseAudio (Optional, for Lower Latency)

```bash
# Stop PulseAudio
systemctl --user stop pulseaudio.socket
systemctl --user stop pulseaudio.service

# Disable auto-start
systemctl --user disable pulseaudio.socket
systemctl --user disable pulseaudio.service
```

### Real-Time Priority

```bash
# Add user to audio group
sudo usermod -a -G audio pi

# Edit limits
sudo nano /etc/security/limits.conf

# Add:
@audio   -  rtprio     95
@audio   -  memlock    unlimited
```

---

## Step 7: Create Systemd Service

Create auto-start service for audio routing (will be completed in D-3.3):

```bash
sudo nano /etc/systemd/system/pelican-audio.service
```

```ini
[Unit]
Description=Pelican Cinema Audio Router
After=sound.target

[Service]
Type=simple
User=pi
WorkingDirectory=/home/pi/pelican-cinema
ExecStart=/usr/bin/python3 /home/pi/pelican-cinema/audio_router.py
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

---

## Verification Checklist

- [ ] Pi 5 boots successfully
- [ ] SSH access works
- [ ] Audio interface detected (`arecord -l`)
- [ ] 8 channels available
- [ ] Test recording works
- [ ] Low-latency config applied
- [ ] Service file created (not enabled yet)

---

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| Interface not detected | USB power | Use powered hub |
| Only 2 channels | Driver limitation | Check interface compatibility |
| High latency | PulseAudio | Disable PulseAudio |
| Crackling | Buffer underrun | Increase buffer size |
| No audio | Wrong device | Check `arecord -l` output |

---

## Completion Checklist

- [ ] Pi 5 fully configured
- [ ] Audio interface working
- [ ] 8-channel capture tested
- [ ] Low-latency settings applied

**→ Proceed to D-3.3 (Channel Routing)**
