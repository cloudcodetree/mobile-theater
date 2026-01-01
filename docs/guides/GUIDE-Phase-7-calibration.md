# Implementation Guide: Phase 7 - Calibration & Testing

## Overview
**Phase Budget:** $50  
**Time Estimate:** 1 week  
**Objective:** Calibrate system for optimal audio, conduct field tests

---

## D-7.1: Measurement Setup

### Equipment

| Item | Model | Cost |
|------|-------|------|
| Measurement Mic | miniDSP UMIK-1 | $75 (or borrow) |
| Mic Stand | Any boom stand | $15 |
| Software | REW (free) | $0 |
| Laptop | Any with USB | — |

### Software Setup

1. **Download REW**
   - https://www.roomeqwizard.com/
   - Free, cross-platform

2. **Install UMIK-1 Drivers**
   - Download calibration file from miniDSP
   - Load into REW

3. **Configure REW**
   ```
   Preferences → Soundcard:
   - Output: Your laptop speakers (for sweep)
   - Input: UMIK-1
   
   Preferences → Mic/Meter:
   - Load UMIK-1 calibration file
   ```

---

## D-7.2: System Calibration

### 7.1 Speaker Placement

```
Optimal 7.1 Layout (Top View):

                    SCREEN
          ┌─────────────────────────┐
          │                         │
       FL │           C             │ FR
       ◉  │           ◉             │  ◉
          │                         │
          │                         │
    SL ◉  │                         │  ◉ SR
          │           👤            │
          │        LISTENER         │
          │                         │
    RL ◉  │          SUB            │  ◉ RR
          │           ◉             │
          └─────────────────────────┘

Angles from listener:
- FL/FR: ±30°
- SL/SR: ±90-110°
- RL/RR: ±135-150°
- C: 0° (center)
- SUB: Anywhere front half
```

### 7.2 Level Matching

1. Place UMIK-1 at listening position
2. Play pink noise on each channel
3. Measure SPL with REW
4. Adjust amplifier gain to match 75dB reference

| Channel | Target SPL | Measured | Adjustment |
|---------|------------|----------|------------|
| FL | 75 dB | | |
| FR | 75 dB | | |
| C | 75 dB | | |
| SL | 75 dB | | |
| SR | 75 dB | | |
| RL | 75 dB | | |
| RR | 75 dB | | |
| SUB | 75 dB | | |

### 7.3 Distance Compensation

Measure distance from each speaker to listening position:

| Channel | Distance | Delay (ms) |
|---------|----------|------------|
| FL | _____ m | |
| FR | _____ m | |
| C | _____ m | |
| SL | _____ m | |
| SR | _____ m | |
| RL | _____ m | |
| RR | _____ m | |
| SUB | _____ m | |

```
Delay calculation:
delay_ms = (max_distance - speaker_distance) / 0.343

Example: If farthest speaker is 4m and this speaker is 3m:
delay = (4 - 3) / 0.343 = 2.9 ms
```

Implement delay in firmware:

```cpp
// In satellite_rx.ino
#define DELAY_SAMPLES 128  // Calculated per speaker

int16_t delayBuffer[DELAY_SAMPLES];
int delayIndex = 0;

int16_t applyDelay(int16_t sample) {
  int16_t delayed = delayBuffer[delayIndex];
  delayBuffer[delayIndex] = sample;
  delayIndex = (delayIndex + 1) % DELAY_SAMPLES;
  return delayed;
}
```

### 7.4 Frequency Response

Use REW to measure each speaker:

1. Run frequency sweep (20Hz - 20kHz)
2. View frequency response graph
3. Note any major peaks/dips
4. Apply EQ if needed (optional)

### 7.5 Subwoofer Integration

1. Set sub crossover to 80Hz
2. Match sub level to mains
3. Check phase (0° vs 180°)
4. Verify smooth transition at crossover

---

## D-7.3: Field Deployment Test

### Test Scenarios

#### Scenario 1: Backyard Movie Night
- [ ] Setup on grass/patio
- [ ] Test with 2-hour movie
- [ ] Verify battery lasts
- [ ] Check audio at distance

#### Scenario 2: Indoor Setup
- [ ] Living room deployment
- [ ] Test surround effect
- [ ] Check reflection handling

#### Scenario 3: Outdoor Event
- [ ] Park or campsite
- [ ] Test range from command module
- [ ] Verify weatherproofing

### Test Checklist

- [ ] Unpack and setup in <15 min
- [ ] All speakers power on
- [ ] All speakers receive audio
- [ ] Sync sounds correct
- [ ] Surround effect works
- [ ] Gaming latency acceptable
- [ ] 3+ hour runtime achieved
- [ ] Teardown in <10 min

---

## Final Documentation

### System Specifications

```
PELICAN CINEMA - FINAL SPECS
─────────────────────────────────────────
Configuration:     7.1 Wireless
Speakers:          8 (7 satellite + 1 sub)
Total Budget:      $________
Total Weight:      ________ lbs
Setup Time:        ________ min
Runtime:           ________ hours
Latency:           ________ ms
Range:             ________ m
─────────────────────────────────────────
```

### Lessons Learned
1. ________________________________
2. ________________________________
3. ________________________________

### Future Improvements
1. ________________________________
2. ________________________________
3. ________________________________

---

## 🎉 Project Complete!

### Share Your Build
- [ ] Take photos of completed system
- [ ] Record demo video
- [ ] Update GitHub repo with final docs
- [ ] Consider writing build guide for others

### Maintenance Schedule
- Monthly: Check battery health
- Quarterly: Update firmware if needed
- Annually: Replace any worn components

---

**Congratulations on completing the Pelican Cinema!** 🎬🔊
