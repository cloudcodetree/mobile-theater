# Implementation Guide: D-7.1 Measurement Setup (UMIK-1 + REW)

## Overview
**Time Estimate:** 2-3 hours | **Cost:** $90 | **Depends On:** Phase 6

---

## Equipment
| Item | Model | Cost |
|------|-------|------|
| Measurement Mic | miniDSP UMIK-1 | $75 |
| Mic Stand | Any boom stand | $15 |
| USB Extension | 10ft active | $10 |

## Software (Free)
- REW (Room EQ Wizard): https://www.roomeqwizard.com/

## UMIK-1 Setup

### 1. Download Calibration File
1. Go to miniDSP website
2. Enter UMIK-1 serial number
3. Download .cal file
4. Save to known location

### 2. Install REW
```
Windows: Download installer
Mac: Download DMG
Linux: Download JAR, run with Java
```

### 3. Configure REW
```
Preferences → Soundcard:
├── Output Device: Laptop speakers (for sweep)
├── Input Device: UMIK-1
├── Sample Rate: 48000
└── Buffer Size: 4096

Preferences → Mic/Meter:
└── Load calibration file (.cal)
```

### 4. Test Setup
1. Play REW test signal
2. Check UMIK-1 level meter
3. Adjust laptop volume for ~75dB

## Measurement Position
```
           SCREEN
    ┌─────────────────────┐
    │                     │
  FL│         C           │FR
    │                     │
    │                     │
    │         🎤          │  ← UMIK-1 at ear height
    │      (MLP)          │     at Main Listening Position
    │                     │
    │                     │
  SL│                     │SR
    │                     │
    └─────────────────────┘
```

## Quick Test Procedure
1. Place mic at listening position (ear height)
2. Point mic straight up (omnidirectional)
3. Play test sweep in REW
4. View frequency response graph
5. Save measurement

## Success Criteria
- [ ] UMIK-1 recognized in REW
- [ ] Calibration file loaded
- [ ] Test sweep plays and measures
- [ ] SPL reads correctly (~75dB)

**→ Proceed to D-7.2**
