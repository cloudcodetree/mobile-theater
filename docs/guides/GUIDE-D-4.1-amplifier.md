# Implementation Guide: D-4.1 Class-D Amplifier Integration

## Overview
**Time Estimate:** 2-3 hours  
**Skill Level:** Beginner  
**Cost:** $45  
**Depends On:** Phase 3 complete

---

## Shopping List

| Item | Model | Cost | Link |
|------|-------|------|------|
| TPA3116D2 Amp Board | 2×50W | $12 | [Amazon](https://amazon.com/dp/B07Q2X8J5G) |
| 4" Full-Range Driver | Dayton Audio ND105-4 | $25 | [Parts Express](https://parts-express.com/dayton-audio-nd105-4) |
| Binding Posts | Pair | $5 | Amazon |
| Hookup Wire | 18 AWG | $3 | Amazon |

**Total:** $45

---

## TPA3116D2 Overview

### Specifications

| Spec | Value |
|------|-------|
| Power | 2×50W @ 4Ω, 2×100W @ 2Ω |
| Voltage | 12-24V DC |
| THD | <0.1% at 1W |
| SNR | >95dB |
| Efficiency | >90% |

### Board Layout

```
TPA3116D2 Amp Board (Top View):
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   [VCC+] [VCC-]                           [R+] [R-]            │
│     ●      ●                                ●    ●              │
│     │      │                                │    │              │
│   Power Input                          Right Speaker            │
│   12-24V DC                                                     │
│                                                                 │
│                    ┌─────────┐                                  │
│                    │         │                                  │
│                    │  TPA3116│                                  │
│                    │   Chip  │                                  │
│                    │         │                                  │
│                    └─────────┘                                  │
│                                                                 │
│   [L IN]  [R IN]  [GND]                   [L+] [L-]            │
│     ●       ●       ●                       ●    ●              │
│     │       │       │                       │    │              │
│   Audio Input                          Left Speaker             │
│   (from DAC)                                                    │
│                                                                 │
│   Volume Pot                [MUTE]  [STBY]                     │
│   ┌───────────┐               ●       ●                        │
│   │     ◯     │                                                 │
│   └───────────┘                                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Step 1: Understand the Signal Chain

```
┌─────────────────────────────────────────────────────────────────┐
│                        SIGNAL FLOW                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  [ESP32] ──I2S──► [PCM5102A] ──Analog──► [TPA3116] ──► [Speaker]│
│                      DAC                    Amp                  │
│                       │                      │                   │
│                   Line Level            Amplified               │
│                   ~2V p-p               ~20V p-p                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Step 2: Wiring

### DAC to Amp Connection

```
PCM5102A                    TPA3116D2
┌────────────┐              ┌────────────┐
│            │              │            │
│  L OUT  ●──┼──────────────┼──● L IN    │
│            │              │            │
│  R OUT  ●──┼──────────────┼──● R IN    │
│            │              │            │
│  GND    ●──┼──────────────┼──● GND     │
│            │              │            │
└────────────┘              └────────────┘
```

### Amp to Speaker Connection

```
TPA3116D2                   Speaker
┌────────────┐              ┌────────────┐
│            │              │            │
│  L+     ●──┼──────Red─────┼──● +       │
│            │              │            │
│  L-     ●──┼──────Black───┼──● -       │
│            │              │            │
└────────────┘              └────────────┘

Note: For mono speaker pod, use only L channel
      Or bridge both channels for more power (see below)
```

### Power Connection

```
Battery (9.6V LiFePO4)      TPA3116D2
┌────────────┐              ┌────────────┐
│            │              │            │
│  +      ●──┼──────Red─────┼──● VCC+    │
│            │              │            │
│  -      ●──┼──────Black───┼──● VCC-    │
│            │              │            │
└────────────┘              └────────────┘
```

---

## Step 3: Complete Wiring Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    SPEAKER POD WIRING                                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  BATTERY PACK                                                           │
│  ┌─────────────┐                                                       │
│  │ 3S LiFePO4  │                                                       │
│  │   9.6V      │                                                       │
│  │  + ─────────┼───┬────────────────────────────► TPA3116 VCC+         │
│  │             │   │                                                    │
│  │  - ─────────┼───┼─┬──────────────────────────► TPA3116 VCC-         │
│  └─────────────┘   │ │                                                  │
│                    │ │                                                  │
│               ┌────┴─┴────┐                                            │
│               │ Buck Conv │                                             │
│               │ 5V Output │                                             │
│               │  + ───────┼───────► ESP32 5V + PCM5102A VCC            │
│               │  - ───────┼───────► ESP32 GND + PCM5102A GND           │
│               └───────────┘                                             │
│                                                                         │
│  ESP32-S3                                                               │
│  ┌─────────────┐                                                       │
│  │ GPIO4 ──────┼───────► PCM5102A BCK                                  │
│  │ GPIO5 ──────┼───────► PCM5102A LCK                                  │
│  │ GPIO6 ──────┼───────► PCM5102A DIN                                  │
│  └─────────────┘                                                       │
│                                                                         │
│  PCM5102A                                                               │
│  ┌─────────────┐                                                       │
│  │ L OUT ──────┼───────► TPA3116 L IN                                  │
│  │ GND ────────┼───────► TPA3116 GND                                   │
│  └─────────────┘                                                       │
│                                                                         │
│  TPA3116                                                                │
│  ┌─────────────┐                                                       │
│  │ L+ ─────────┼───────► Speaker +                                     │
│  │ L- ─────────┼───────► Speaker -                                     │
│  │ MUTE ───────┼───────► 3.3V (unmuted)                                │
│  └─────────────┘                                                       │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Step 4: Bench Test

### Before Installing in Enclosure

1. **Power test (no signal)**
   - Apply 12V to amp (bench supply with current limit)
   - Verify no smoke, normal current draw (~50mA idle)
   
2. **Signal test**
   - Connect DAC to amp
   - Connect speaker to amp
   - Play test tone from ESP32
   - Verify clean audio

3. **Volume test**
   - Adjust volume pot
   - Verify range from quiet to loud
   - Check for distortion at high volume

### Expected Current Draw

| State | Current @ 12V |
|-------|---------------|
| Idle | ~50mA |
| Low volume | ~200mA |
| Medium volume | ~500mA |
| High volume | ~1-2A |

---

## Step 5: Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| No audio | Mute pin floating | Connect MUTE to 3.3V |
| Distortion | Clipping | Reduce volume or input level |
| Oscillation | No load | Always connect speaker |
| Hum/buzz | Ground loop | Use single-point grounding |
| Overheating | Too much power | Add heatsink or reduce volume |

---

## Completion Checklist

- [ ] Amp board received and inspected
- [ ] Speaker driver received
- [ ] DAC to amp wired correctly
- [ ] Amp to speaker wired correctly
- [ ] Power connected (with fuse!)
- [ ] Bench test passed
- [ ] Audio quality acceptable

**→ Proceed to D-4.2 (Battery System)**
