# D-7.1: Measurement Setup (UMIK-1 + REW)

## Overview
| Field | Value |
|-------|-------|
| **Phase** | 7 - Calibration & Testing |
| **Cost** | $40-90 |
| **Time** | 2-3 hours |
| **Depends On** | All pods built and functional |

---

## Objective
Set up professional audio measurement system for calibrating the 7.1 speaker array.

---

## Bill of Materials

| Item | Model | Cost | Notes |
|------|-------|------|-------|
| Measurement Mic | miniDSP UMIK-1 | $75 | USB, calibrated |
| Mic Stand | Any boom stand | $15 | Reaches ear height |
| USB Extension | 10ft USB 2.0 | $8 | Optional |
| **Total** | | **$98** | |

### Alternatives
- **Dayton Audio EMM-6** - $20 (needs interface + calibration)
- **Borrow/Rent UMIK-1** - Check local audio groups

---

## Step 1: Download & Install REW

### Get Room EQ Wizard (FREE)
```
https://www.roomeqwizard.com/
```

Works on: Windows, Mac, Linux

### Installation
1. Download installer for your OS
2. Run installer
3. Launch REW

---

## Step 2: Set Up UMIK-1

### 2.1 Download Calibration File
1. Go to: https://www.minidsp.com/products/acoustic-measurement/umik-1
2. Enter your mic's serial number (on the mic body)
3. Download the `.txt` calibration file
4. Save to known location

### 2.2 Connect Mic
```
[UMIK-1] ──USB──► [Laptop/PC]
```

### 2.3 Verify in OS
- **Windows:** Device Manager → Sound → UMIK-1
- **Mac:** System Preferences → Sound → Input → UMIK-1
- **Linux:** `arecord -l` shows UMIK-1

---

## Step 3: Configure REW

### 3.1 Audio Settings
```
Preferences → Soundcard

Output Device: [Your laptop speakers or audio interface]
Input Device:  UMIK-1

Sample Rate: 48000
```

### 3.2 Load Calibration
```
Preferences → Mic/Meter

Calibration: [Browse to your UMIK-1 .txt file]
C Weighted: Unchecked (use flat)
```

### 3.3 Verify Levels
```
Tools → SPL Meter

- Clap or snap near mic
- Meter should respond
- Typical room: 30-50 dB
```

---

## Step 4: Test Measurement

### 4.1 Simple Sweep Test
1. Position mic at listening position (ear height)
2. Play test tone from ONE speaker
3. In REW: `Measure` button
4. Check graph shows frequency response

### 4.2 Expected Response

```
Good 4" full-range driver:
                              
   dB │          ╭──────────────╮
  90 ─┤         ╱                ╲
  80 ─┤        ╱                  ╲
  70 ─┤       ╱                    ╲
  60 ─┤──────╱                      ╲───
      └──────┴───────────────────────────
           100    1k     10k    20k  Hz
           
Note: Roll-off below 80Hz is normal for 4" driver
```

---

## Step 5: Room Setup for Measurement

### Mic Placement

```
                SCREEN
    ┌─────────────────────────────┐
    │                             │
    │                             │
    │                             │
    │                             │
    │             🎤              │  ← Mic at ear height
    │         (UMIK-1)            │    when seated
    │                             │
    │                             │
    └─────────────────────────────┘

Height: 36-42 inches (seated ear level)
Position: Primary listening position
Orientation: Pointing UP (90° mode)
```

### UMIK-1 Orientation
- **0°** - Point directly at speaker (single speaker measurement)
- **90°** - Point UP (room/multi-speaker measurement)

For 7.1 calibration, use **90° mode** (pointing up).

---

## Verification Checklist

- [ ] REW installed and running
- [ ] UMIK-1 recognized as input device
- [ ] Calibration file loaded
- [ ] SPL meter responds to sound
- [ ] Test sweep produces valid graph
- [ ] Mic stand at correct height

---

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| No input signal | Wrong device selected | Check Preferences → Soundcard |
| Flat line at -∞ | Mic muted | Check OS input settings |
| Very low level | Need preamp | UMIK-1 has built-in preamp, check USB |
| Noisy measurement | Interference | Move laptop power supply away |
| Distorted peaks | Clipping | Reduce output volume |

---

## Reference: SPL Levels

| Description | Level |
|-------------|-------|
| Quiet room | 30-40 dB |
| Normal conversation | 60-70 dB |
| Reference level (THX) | 85 dB |
| Loud movie peaks | 105 dB |
| Threshold of pain | 120 dB |

---

## Completion Checklist

- [ ] UMIK-1 acquired
- [ ] REW installed
- [ ] Calibration file loaded
- [ ] Test measurement successful
- [ ] Ready for full system calibration

**→ Proceed to D-7.2: System Calibration**
