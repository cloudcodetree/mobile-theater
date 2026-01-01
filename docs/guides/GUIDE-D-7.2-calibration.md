# Implementation Guide: D-7.2 System Calibration

## Overview
**Time Estimate:** 4-6 hours | **Cost:** $0 | **Depends On:** D-7.1

---

## Calibration Steps

### Step 1: Level Matching
Set all speakers to same SPL at listening position.

1. Play pink noise on FL
2. Measure SPL with UMIK-1 (target: 75dB)
3. Adjust amp gain
4. Repeat for each speaker:

| Channel | Target | Measured | Adjustment |
|---------|--------|----------|------------|
| FL | 75dB | | |
| FR | 75dB | | |
| C | 75dB | | |
| SL | 75dB | | |
| SR | 75dB | | |
| RL | 75dB | | |
| RR | 75dB | | |
| SUB | 75dB | | |

### Step 2: Distance/Delay
Measure distance from each speaker to MLP:

| Channel | Distance | Delay (ms) |
|---------|----------|------------|
| FL | ___m | |
| FR | ___m | |
| C | ___m | |
| SL | ___m | |
| SR | ___m | |
| RL | ___m | |
| RR | ___m | |
| SUB | ___m | |

```
Delay formula:
delay_ms = (max_distance - this_distance) / 0.343
```

### Step 3: Frequency Response
For each speaker:
1. Run REW frequency sweep
2. Observe response curve
3. Note any major problems

```
Ideal response: Flat ±3dB from 80Hz-16kHz
Satellites likely: Roll-off below 100Hz (OK)
Subwoofer: Response 30-80Hz
```

### Step 4: Subwoofer Integration
1. Set sub crossover to 80Hz
2. Play sweep, measure response
3. Check for gap or overlap at crossover
4. Try sub phase 0° vs 180°
5. Choose setting with smoothest response

### Step 5: Apply EQ (Optional)
If firmware supports it, apply corrections:
```cpp
// Simple 3-band EQ
#define BASS_GAIN 0.0    // dB
#define MID_GAIN  0.0    // dB
#define TREBLE_GAIN 0.0  // dB
```

## Save Configuration
Document all settings for reproducibility:
```
PELICAN CINEMA CALIBRATION
Date: ________

Levels (dB):
FL: ___ FR: ___ C: ___ SL: ___
SR: ___ RL: ___ RR: ___ SUB: ___

Delays (ms):
FL: ___ FR: ___ C: ___ SL: ___
SR: ___ RL: ___ RR: ___ SUB: ___

Sub Phase: 0° / 180°
Sub Crossover: ___Hz
```

## Success Criteria
- [ ] All channels within ±2dB
- [ ] Delays compensated
- [ ] Smooth crossover with sub
- [ ] No major frequency anomalies

**→ Proceed to D-7.3**
