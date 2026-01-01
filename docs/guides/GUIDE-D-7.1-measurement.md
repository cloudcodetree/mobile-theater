# D-7.1 Measurement Setup (UMIK-1 + REW)

## Overview
**Time:** 2-3 hours | **Cost:** $90 | **Skill:** Intermediate

## Equipment Needed
| Item | Model | Cost |
|------|-------|------|
| Measurement Mic | miniDSP UMIK-1 | $75 |
| Mic Stand | Any boom stand | $15 |
| Software | REW (free) | $0 |
| Laptop | USB-equipped | — |

## Software Setup

### Install REW
1. Download: https://www.roomeqwizard.com/
2. Install for your OS
3. Launch REW

### Configure UMIK-1
```
1. Connect UMIK-1 to laptop USB
2. Download calibration file from miniDSP
   (serial number specific)
3. In REW: Preferences → Mic/Meter
4. Load calibration file
5. Set C-weighted
```

### REW Settings
```
Preferences → Soundcard:
├── Output: Laptop speakers (for sweep)
├── Input: UMIK-1
└── Sample rate: 48000 Hz

Preferences → Mic/Meter:
├── Cal file: [Your UMIK-1 file]
└── Sensitivity: From cal file
```

## Test Procedure

### SPL Meter Mode
```
1. REW → Tools → SPL Meter
2. Position mic at ear height
3. Play pink noise from system
4. Read SPL level
5. Adjust speaker gain to match 75dB
```

### Frequency Sweep
```
1. REW → Measure
2. Set sweep: 20Hz - 20kHz
3. Level: -12dB
4. Click "Start"
5. View response graph
```

## Mic Positioning
```
Optimal Position:
- Seated ear height
- Centered between speakers  
- Away from walls (>1m)
- Pointed up (omnidirectional)

    [FL]         [C]         [FR]
      \          |          //
       \         |         //
        \        |        //
         \       |       //
          [======MIC======]
               👤
            Listener
```

## Calibration File
Create measurement log:
```
# UMIK-1 Calibration
Serial: _______________
Cal file: _______________
Date: _______________

# System Under Test
Project: Pelican Cinema
Speakers: 7.1 wireless
Location: _______________
```

→ Proceed to D-7.2 (Calibration)
