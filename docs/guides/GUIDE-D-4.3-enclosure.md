# D-4.3 Speaker Pod Enclosure

## Overview
**Time:** 6-10 hours | **Cost:** $40 | **Skill:** Intermediate

## Design Specs
- Internal volume: 2-3L for 4" driver
- Dimensions: 150×150×200mm (6"×6"×8")
- Material: PETG (3D print) or MDF

## Internal Layout
```
┌─────────────────────┐
│   [4" DRIVER]       │ ← Front, sealed
│                     │
│  [ESP32] [AMP]      │ ← Middle
│                     │
│  [BATTERY PACK]     │ ← Bottom
│  [PWR][CHG][LED]    │ ← Rear panel
└─────────────────────┘
```

## Build Steps
1. Print/build enclosure (PETG, 0.2mm layers, 30% infill)
2. Install driver with foam gasket
3. Mount ESP32+DAC and amp
4. Install battery with foam padding
5. Wire all components
6. Seal and test for air leaks

## Stand Mount
- 1/4"-20 threaded insert (camera mount standard)
- Or 35mm pole socket for speaker stands

## Checklist
- [ ] Enclosure complete
- [ ] Driver sealed
- [ ] Electronics mounted
- [ ] Battery installed
- [ ] Stand mount ready

→ Proceed to D-4.4
