# D-7.2: System Calibration

## Overview
| Field | Value |
|-------|-------|
| **Phase** | 7 - Calibration & Testing |
| **Cost** | $0 (software only) |
| **Time** | 3-4 hours |
| **Depends On** | D-7.1 complete |

---

## Objective
Calibrate all 8 speakers for matched levels, proper delays, and optimized frequency response.

---

## Calibration Order

```
1. Level Matching     → All speakers same volume
2. Distance/Delay     → All speakers arrive at same time
3. Frequency Response → Identify any problems
4. Subwoofer Integration → Smooth crossover
5. Final Verification → Full system test
```

---

## Step 1: Level Matching

### 1.1 Setup
- Position UMIK-1 at listening position (90° up)
- Open REW SPL Meter: `Tools → SPL Meter`

### 1.2 Generate Pink Noise

In REW: `Tools → Generator`
- Signal: Pink Noise
- Level: -20 dB (adjust as needed)
- Output: Route to each speaker individually

Or use test file: https://www.audiocheck.net/testtones_pinknoise.php

### 1.3 Measure Each Speaker

Play pink noise on ONE speaker at a time. Record SPL:

| Channel | Target | Measured | Adjustment |
|---------|--------|----------|------------|
| FL | 75 dB | _____ dB | |
| FR | 75 dB | _____ dB | |
| C | 75 dB | _____ dB | |
| SL | 75 dB | _____ dB | |
| SR | 75 dB | _____ dB | |
| RL | 75 dB | _____ dB | |
| RR | 75 dB | _____ dB | |
| SUB | 75 dB | _____ dB | |

### 1.4 Adjust Levels

**Option A: Amplifier Trim**
Adjust gain pot on each TPA3116 amp board

**Option B: Firmware Volume**
```cpp
// In satellite firmware
#define VOLUME_SCALE 0.85  // Reduce to 85%

int16_t adjustVolume(int16_t sample) {
  return (int16_t)(sample * VOLUME_SCALE);
}
```

**Option C: Command Module DSP**
Adjust per-channel volume in Pi 5 audio routing

---

## Step 2: Distance/Delay Compensation

### 2.1 Measure Distances

Measure from each speaker to UMIK-1 position:

```
                    SCREEN
         FL ◉─────────────────◉ FR
              ╲    C ◉    ╱
               ╲   │   ╱
          SL ◉──╲──│──╱──◉ SR
                 ╲ │ ╱
                  ╲│╱
                   🎤 ← Measure to here
                  ╱│╲
          RL ◉──╱──│──╲──◉ RR
               ╱       ╲
            SUB ◉
```

Record distances:

| Channel | Distance (m) | Distance (ft) |
|---------|--------------|---------------|
| FL | | |
| FR | | |
| C | | |
| SL | | |
| SR | | |
| RL | | |
| RR | | |
| SUB | | |

### 2.2 Calculate Delays

Find the FARTHEST speaker. All other speakers get delay.

```
Speed of sound: 343 m/s = 1125 ft/s

Delay (ms) = (Max_Distance - This_Distance) / 0.343

Example:
- Farthest speaker (RL): 4.0 m
- This speaker (C): 2.5 m
- Delay = (4.0 - 2.5) / 0.343 = 4.37 ms
```

Calculate for all:

| Channel | Distance | Delay (ms) | Delay (samples @ 48kHz) |
|---------|----------|------------|-------------------------|
| FL | | | |
| FR | | | |
| C | | | |
| SL | | | |
| SR | | | |
| RL | | | |
| RR | | | |
| SUB | | | |

### 2.3 Implement Delay

**In Satellite Firmware:**

```cpp
// Per-speaker delay configuration
#define DELAY_MS 4.37  // Set per speaker
#define DELAY_SAMPLES (int)(DELAY_MS * 48)  // @ 48kHz

int16_t delayBuffer[DELAY_SAMPLES];
int delayIdx = 0;

int16_t applyDelay(int16_t sample) {
  int16_t delayed = delayBuffer[delayIdx];
  delayBuffer[delayIdx] = sample;
  delayIdx = (delayIdx + 1) % DELAY_SAMPLES;
  return delayed;
}

// In audio playback loop:
for (int i = 0; i < SAMPLES_PER_PACKET; i++) {
  outputSample = applyDelay(inputSample);
  // ... send to I2S
}
```

---

## Step 3: Frequency Response

### 3.1 Measure Each Speaker

In REW:
1. Select speaker output
2. Click `Measure`
3. REW plays sweep, records response
4. Save measurement with speaker name

### 3.2 Analyze Response

Look for:
- **Major dips** (>6dB) - Room nulls, can't fix with EQ
- **Major peaks** (>6dB) - Room modes, can reduce with EQ
- **Roll-off points** - Expected for small drivers

### 3.3 Typical 4" Driver Response

```
Expected:
- Usable range: 80 Hz - 18 kHz
- -3dB point: ~100 Hz
- Rising response toward treble: normal
- Slight dip at 3-4 kHz: crossover region

Red Flags:
- Sharp notch at specific frequency: phase issue
- Excessive peaks: room problem
- Early roll-off: driver/enclosure issue
```

---

## Step 4: Subwoofer Integration

### 4.1 Set Crossover

Subwoofer should handle below 80 Hz:

```cpp
// In subwoofer firmware - simple low-pass filter
#define CROSSOVER_FREQ 80  // Hz
#define SAMPLE_RATE 48000

float alpha = (2.0 * PI * CROSSOVER_FREQ) / SAMPLE_RATE;
float filtered = 0;

int16_t lowPassFilter(int16_t sample) {
  filtered = filtered + alpha * (sample - filtered);
  return (int16_t)filtered;
}
```

### 4.2 Phase Alignment

Test with phase at 0° and 180°. Choose setting with MORE bass at crossover.

```cpp
// Phase invert option
#define PHASE_INVERT false  // Set true for 180°

int16_t output = PHASE_INVERT ? -sample : sample;
```

### 4.3 Level Match

Subwoofer often needs +3 to +6 dB boost for movie content:

| Setting | Use Case |
|---------|----------|
| +0 dB | Reference/flat |
| +3 dB | Music |
| +6 dB | Movies |
| +10 dB | Action movies/bass lovers |

---

## Step 5: Final Verification

### 5.1 Play Test Content

| Test | What to Listen For |
|------|-------------------|
| Pink noise all channels | Even coverage, no holes |
| Channel ID test | Each speaker plays correct channel |
| Bass sweep | Smooth, no rattles |
| Dialogue test | Clear, centered |
| Surround demo | Effects move smoothly |

### 5.2 Test Files

Free test files:
- https://www.dolby.com/siteassets/technologies/dolby-atmos/dolby-atmos-trailer_amaze_702-2ch.mp4
- https://thedigitaltheater.com/dolby-trailers/

### 5.3 Document Settings

Record final calibration:

| Channel | Level (dB) | Delay (ms) | Notes |
|---------|------------|------------|-------|
| FL | | | |
| FR | | | |
| C | | | |
| SL | | | |
| SR | | | |
| RL | | | |
| RR | | | |
| SUB | | | |

---

## Completion Checklist

- [ ] All channels level-matched to ±1 dB
- [ ] Distance delays calculated and applied
- [ ] Frequency response measured
- [ ] Subwoofer crossover set
- [ ] Subwoofer phase optimized
- [ ] Final verification with test content
- [ ] Settings documented

**→ Proceed to D-7.3: Field Deployment Test**
