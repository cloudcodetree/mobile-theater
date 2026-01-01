# Implementation Guide: D-4.1 Class-D Amplifier Integration

## Overview
**Time Estimate:** 3-4 hours | **Cost:** $45 | **Depends On:** Phase 3

---

## Parts
| Item | Model | Cost |
|------|-------|------|
| TPA3116D2 Amp Board | 2x50W | $12 |
| 4" Full-Range Driver | Dayton Audio ND105 | $25 |
| Binding Posts | Banana plug compatible | $5 |
| Wire | 16 AWG speaker wire | $3 |

## TPA3116 Board Pinout
```
┌──────────────────────────────────────┐
│  [VCC+] [VCC-]      [R+] [R-]       │
│   Power Input        Right Output    │
│                                      │
│  [L-IN] [R-IN] [GND]  [L+] [L-]     │
│   Audio Input         Left Output    │
│                                      │
│  [MUTE] [STBY]    [VOL POT]         │
│   Control          Volume            │
└──────────────────────────────────────┘
```

## Wiring Diagram
```
PCM5102A DAC          TPA3116D2
┌──────────┐          ┌──────────┐
│ L OUT ───┼──────────┼─► L-IN   │
│ R OUT ───┼──────────┼─► R-IN   │
│ GND ─────┼──────────┼─► GND    │
└──────────┘          │          │
                      │ L+ ──────┼───► Speaker +
Battery 9.6V ─────────┼─► VCC+   │
Battery GND ──────────┼─► VCC-   │
                      │ L- ──────┼───► Speaker -
3.3V ─────────────────┼─► MUTE   │  (unmuted)
                      └──────────┘
```

## Bench Test Procedure
1. Connect amp to bench power supply (12V, 2A limit)
2. Connect audio source (phone) to input
3. Connect to test speaker (NOT the final driver yet)
4. Slowly increase volume
5. Check for clean audio, no oscillation

## Integration Test
1. Replace bench supply with 9.6V battery
2. Replace audio source with DAC output
3. Connect final 4" driver
4. Play test tones from ESP32 RX
5. Measure SPL at 1 meter

## Gain Adjustment
Most TPA3116 boards have a gain pot. Start at minimum and increase until:
- Full volume is loud enough (90+ dB SPL)
- No clipping at max volume
- No audible noise floor

## Success Criteria
- [ ] Amp powers from battery voltage
- [ ] Clean audio from DAC input
- [ ] No oscillation or noise
- [ ] Adequate volume (>85 dB SPL at 1m)

**→ Proceed to D-4.2**
