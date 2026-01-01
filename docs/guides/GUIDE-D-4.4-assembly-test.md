# Implementation Guide: D-4.4 Prototype Pod Assembly & Test

## Overview
**Time Estimate:** 4-6 hours  
**Skill Level:** Intermediate  
**Cost:** $20 (misc supplies)  
**Depends On:** D-4.1, D-4.2, D-4.3 complete

---

## Final Assembly Checklist

### Components Ready

- [ ] Enclosure complete (D-4.3)
- [ ] Battery pack assembled (D-4.2)
- [ ] Amp + Driver tested (D-4.1)
- [ ] ESP32 + DAC from Phase 1
- [ ] All wiring prepared

---

## Step 1: Pre-Assembly Test

**Before sealing in enclosure, verify everything works on the bench:**

```
Bench Test Setup:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  [Battery] ──► [BMS] ──┬──► [TPA3116] ──► [Speaker]            │
│                        │                                        │
│                        └──► [Buck 5V] ──► [ESP32+DAC]          │
│                                                                 │
│  Expected:                                                      │
│  1. ESP32 boots, shows MAC on serial                           │
│  2. Receives audio from TX                                     │
│  3. DAC outputs to amp                                         │
│  4. Speaker plays audio                                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Bench Test Procedure

1. Connect battery to BMS output
2. Verify 9.6V at amp input
3. Verify 5.0V at ESP32
4. Power on TX (from Phase 2/3)
5. Verify ESP32 receives packets
6. Verify audio plays through speaker

---

## Step 2: Final Assembly

### Assembly Order

```
1. Mount battery pack (secure with foam/velcro)
         │
         ▼
2. Mount ESP32 + DAC (standoffs)
         │
         ▼
3. Mount amplifier (heatsink clearance!)
         │
         ▼
4. Mount buck converter
         │
         ▼
5. Connect all wiring (follow diagram)
         │
         ▼
6. Install power switch + charge port
         │
         ▼
7. Route antenna (if external)
         │
         ▼
8. Final test BEFORE closing
         │
         ▼
9. Apply gasket and seal enclosure
         │
         ▼
10. Final test AFTER closing
```

---

## Step 3: Comprehensive Testing

### Test 1: Power System

| Test | Expected | Actual | Pass |
|------|----------|--------|------|
| Battery voltage | 9.6V ±0.5V | | |
| Buck output | 5.0V ±0.1V | | |
| Idle current | <100mA | | |
| BMS cutoff works | Yes | | |

### Test 2: Wireless Reception

| Test | Expected | Actual | Pass |
|------|----------|--------|------|
| ESP32 boots | Serial output | | |
| Receives from TX | <1% loss | | |
| Range 1m | 100% | | |
| Range 10m | >98% | | |
| Range 20m | >95% | | |

### Test 3: Audio Quality

| Test | Expected | Actual | Pass |
|------|----------|--------|------|
| 1kHz tone | Clean | | |
| No hum/buzz | Silent background | | |
| Full volume | No distortion | | |
| Low frequencies | Audible | | |
| High frequencies | Clear | | |

### Test 4: Runtime

| Test | Expected | Actual | Pass |
|------|----------|--------|------|
| Low volume | >6 hours | | |
| Medium volume | >4 hours | | |
| High volume | >2 hours | | |

### Test 5: Durability

| Test | Expected | Actual | Pass |
|------|----------|--------|------|
| Drop 0.5m onto grass | Survives | | |
| Shake test | Nothing loose | | |
| 1 hour continuous | No overheating | | |

---

## Step 4: Document Results

### Prototype Specifications

```
PROTOTYPE POD #1 - FINAL SPECS
─────────────────────────────────────────────────────
Channel Assignment:    _____________
Weight:                _____________ g
Dimensions:            ___ × ___ × ___ mm
Battery Voltage:       _____________ V
Runtime (med vol):     _____________ hours
Max SPL:               _____________ dB @ 1m
─────────────────────────────────────────────────────
```

### Issues Found

| Issue | Severity | Solution |
|-------|----------|----------|
| | | |
| | | |
| | | |

---

## Step 5: GO/NO-GO Decision

### Success Criteria

| Criterion | Target | Actual | Pass? |
|-----------|--------|--------|-------|
| Audio quality | Good | | |
| Runtime | >3 hours | | |
| Range | >10m | | |
| Sync accuracy | <2ms | | |
| Durability | Survives drop | | |

### Decision

- [ ] ✅ **GO** - Proceed to Phase 5 (build 7 more)
- [ ] 🔄 **REVISE** - Minor changes needed, rebuild prototype
- [ ] ❌ **REDESIGN** - Major issues, back to design phase

---

## Phase 4 Complete!

If prototype passes all tests:

- [x] D-4.1: Amplifier integration ✓
- [x] D-4.2: Battery system ✓
- [x] D-4.3: Enclosure ✓
- [x] D-4.4: Assembly & test ✓

**→ Proceed to Phase 5 (Production Build)**
