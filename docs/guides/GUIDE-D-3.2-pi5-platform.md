# Implementation Guide: D-3.2 Raspberry Pi 5 Audio Platform

## Overview
**Time Estimate:** 4-6 hours | **Cost:** $220 | **Depends On:** D-3.1

---

## Shopping List
| Item | Cost |
|------|------|
| Raspberry Pi 5 (8GB) | $80 |
| Active Cooler | $10 |
| 27W USB-C PSU | $15 |
| 64GB MicroSD | $15 |
| USB Audio Interface (8ch) | $100 |

## Recommended Audio Interfaces
| Model | Channels | Cost | Notes |
|-------|----------|------|-------|
| Behringer UMC1820 | 18in/20out | $250 | Best value |
| Focusrite Scarlett 18i8 | 18in/8out | $350 | Better quality |
| MOTU UltraLite mk5 | 18in/22out | $650 | Pro quality |

## OS Installation
1. Download Raspberry Pi OS Lite (64-bit)
2. Flash to SD card with Raspberry Pi Imager
3. Enable SSH in Imager settings
4. Set hostname: `pelican-cinema`
5. Boot and connect via SSH

## Initial Setup
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install audio packages
sudo apt install -y alsa-utils pulseaudio jackd2 python3-pip

# Install Python audio libraries
pip3 install sounddevice numpy

# Check audio devices
arecord -l
aplay -l
```

## Low-Latency Tuning
```bash
# Add user to audio group
sudo usermod -a -G audio $USER

# Edit limits.conf
sudo nano /etc/security/limits.conf
# Add:
@audio - rtprio 99
@audio - memlock unlimited

# Optional: Install RT kernel
sudo apt install -y linux-image-rt-arm64
```

## Audio Interface Setup
```bash
# Find your interface
aplay -l
# Note the card number (e.g., card 1)

# Set as default
echo "defaults.pcm.card 1" >> ~/.asoundrc
echo "defaults.ctl.card 1" >> ~/.asoundrc

# Test playback
speaker-test -c 8 -t wav
```

## Test Recording
```bash
# Record 8 channels
arecord -D hw:1,0 -f S16_LE -r 48000 -c 8 -d 10 test.wav

# Play back
aplay test.wav
```

## Success Criteria
- [ ] Pi boots headless
- [ ] SSH access working
- [ ] Audio interface detected
- [ ] 8-channel recording works
- [ ] Low-latency settings applied

**→ Proceed to D-3.3**
