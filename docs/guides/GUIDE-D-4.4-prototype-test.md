# Implementation Guide: D-4.4 Prototype Pod Assembly & Test

## Overview
**Time Estimate:** 4-6 hours | **Cost:** $20 (misc) | **Depends On:** D-4.1, D-4.2, D-4.3

---

## Objective
Assemble complete prototype pod and validate all systems.

## Pre-Assembly Checklist
- [ ] Enclosure complete and dry
- [ ] All electronics tested separately
- [ ] Battery fully charged
- [ ] Firmware flashed with correct channel

## Assembly Order
1. Install battery pack (secure firmly!)
2. Mount BMS and buck converter
3. Wire power distribution
4. Mount ESP32 and DAC
5. Mount amplifier (check airflow)
6. Wire audio path
7. Mount driver (seal carefully)
8. Add status LED + power switch
9. Close enclosure
10. Final test

## Wiring Checklist
```
□ Battery → BMS (balance wires B-, B1, B2, B+)
□ BMS P+/P- → Power distribution
□ Power → Buck converter input
□ Buck 5V → ESP32 VIN
□ Buck 5V → DAC VCC
□ Power → Amp VCC
□ ESP32 I2S → DAC (BCK, LCK, DIN)
□ DAC audio out → Amp input
□ Amp output → Driver
□ Power switch in line with P+
□ LED to ESP32 GPIO + resistor
```

## Functional Tests

### Test 1: Power On
- [ ] Power LED lights
- [ ] ESP32 boots (check serial)
- [ ] No smoke or hot components

### Test 2: Wireless
- [ ] ESP32 receives broadcast from TX
- [ ] Packet loss <1%
- [ ] Sync beacons received

### Test 3: Audio
- [ ] Test tone plays through driver
- [ ] No distortion at moderate volume
- [ ] No noise when silent

### Test 4: Runtime
- [ ] Start with full charge
- [ ] Play audio continuously
- [ ] Record time until shutoff
- [ ] Target: >3 hours

### Test 5: Thermal
- [ ] Run for 1 hour at 50% volume
- [ ] Check component temperatures
- [ ] Amp: <60°C
- [ ] ESP32: <50°C
- [ ] Battery: <40°C

### Test 6: Drop Test (Optional)
- [ ] Drop from 0.5m onto grass
- [ ] Verify still functional
- [ ] Check for loose components

## Test Results
| Test | Target | Actual | Pass? |
|------|--------|--------|-------|
| Power on | Yes | | |
| Wireless RX | <1% loss | | |
| Audio quality | Clean | | |
| Runtime | >3 hrs | | |
| Max temp | <60°C | | |

## Issues Found
| Issue | Severity | Solution |
|-------|----------|----------|
| | | |
| | | |

## Design Revisions
Based on prototype testing, note any changes for production:
- [ ] No changes needed
- [ ] Minor tweaks: _______________
- [ ] Major redesign needed: _______________

## Success Criteria
- [ ] All tests pass
- [ ] Runtime >3 hours
- [ ] No overheating
- [ ] Audio quality acceptable
- [ ] Build process documented

## GO/NO-GO Decision
- [ ] ✅ GO to Phase 5 (Production)
- [ ] 🔄 ITERATE on prototype
- [ ] ❌ MAJOR REDESIGN needed

**→ If GO, proceed to Phase 5**
