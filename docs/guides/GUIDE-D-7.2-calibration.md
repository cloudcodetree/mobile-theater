# D-7.2 System Calibration

## Overview
**Time:** 3-4 hours | **Cost:** $0 | **Skill:** Intermediate

## Calibration Steps

### 1. Level Matching
Match all speakers to same SPL reference.

```
Procedure:
1. Set all amp gains to minimum
2. Play pink noise on FL
3. Adjust FL to 75dB at mic
4. Repeat for each speaker
5. Document gain settings

| Channel | Gain Setting | SPL |
|---------|--------------|-----|
| FL | | 75dB |
| FR | | 75dB |
| C | | 75dB |
| SL | | 75dB |
| SR | | 75dB |
| RL | | 75dB |
| RR | | 75dB |
| SUB | | 75dB |
```

### 2. Distance Compensation
Add delay to closer speakers.

```
Measure distances:
| Channel | Distance | Delay (ms) |
|---------|----------|------------|
| FL | ___m | |
| FR | ___m | |
| C | ___m | |
| ... | | |

Formula:
delay_ms = (max_distance - this_distance) / 0.343

Example: Max=4m, this=3m
delay = (4-3)/0.343 = 2.9ms
```

### 3. Subwoofer Integration
```
1. Set sub crossover to 80Hz
2. Play sweep from 60-100Hz
3. Check for dip or peak at crossover
4. Adjust sub phase (0° or 180°)
5. Fine-tune sub level (+/- 3dB)
```

### 4. Frequency Response
```
Per speaker:
1. Run REW sweep (20Hz-20kHz)
2. Export measurement
3. Note peaks/dips
4. Optional: Apply EQ correction
```

## Saving Calibration
Document all settings:
```
PELICAN CINEMA CALIBRATION
Date: _______________
Location: _______________

LEVELS (amp gain position):
FL: ___ FR: ___ C: ___
SL: ___ SR: ___
RL: ___ RR: ___
SUB: ___

DELAYS (firmware values):
FL: ___ms FR: ___ms C: ___ms
SL: ___ms SR: ___ms
RL: ___ms RR: ___ms
SUB: ___ms

SUB SETTINGS:
Crossover: 80Hz
Phase: 0° / 180°
Level trim: ___dB
```

→ Proceed to D-7.3 (Field Test)
