# Implementation Guide: D-4.3 Speaker Pod Enclosure

## Overview
**Time Estimate:** 8-12 hours | **Cost:** $40 | **Depends On:** D-4.2

---

## Design Requirements
| Requirement | Target |
|-------------|--------|
| Internal volume | 2-3 liters |
| Driver cutout | 4" (102mm) |
| Weight | <2kg empty |
| Weather resistance | IPX4 (splash proof) |
| Mounting | 35mm pole adapter |

## Enclosure Dimensions
```
┌─────────────────────────────────┐
│                                 │
│   Height: 200mm (8")            │
│   Width:  150mm (6")            │
│   Depth:  150mm (6")            │
│                                 │
│   Internal Volume: ~2.5L        │
│   Wall Thickness: 3-5mm         │
│                                 │
└─────────────────────────────────┘
```

## Layout (Cross Section)
```
┌─────────────────────────┐
│ ╔═══════════════════╗   │ ← Driver
│ ║                   ║   │   (front baffle)
│ ║     ACOUSTIC      ║   │
│ ║      VOLUME       ║   │
│ ║                   ║   │
│ ╚═══════════════════╝   │
│ ┌─────┬───────┬─────┐   │
│ │ESP32│  Amp  │Batt │   │ ← Electronics
│ └─────┴───────┴─────┘   │   (rear compartment)
│ [PWR][LED]  [CHARGE]    │ ← Controls
└─────────────────────────┘
```

## Build Options

### Option A: 3D Printed (Recommended)
- Material: PETG (strong, weather resistant)
- Infill: 30%+
- Wall: 3 perimeters
- Print time: ~20 hours
- Files: See /cad folder

### Option B: Wood/MDF
- 12mm MDF
- Wood glue + screws
- Seal with exterior paint
- More work but better acoustics

### Option C: Modify Existing
- Ammo can (surplus)
- Food storage container
- Commercial speaker cabinet

## Acoustic Considerations
- Sealed enclosure (simpler, smaller)
- Add polyfill for acoustic damping
- Seal all joints with silicone
- Gasket around driver

## Driver Mounting
```
Front Baffle Detail:

        ┌───────────────────┐
        │   ○   ○   ○   ○   │  ← Screw holes
        │ ┌─────────────┐   │
        │ │             │   │
        │ │   DRIVER    │   │  ← 102mm cutout
        │ │             │   │
        │ └─────────────┘   │
        │   ○   ○   ○   ○   │
        └───────────────────┘
        
- Use T-nuts or threaded inserts
- Add foam gasket
- Screws: M4 × 20mm
```

## Electronics Compartment
- Separate from acoustic volume
- Accessible panel for maintenance
- Ventilation holes (covered with mesh)
- Cable glands for waterproofing

## 3D Print Files
```
/cad/speaker-pod/
├── body-main.stl
├── front-baffle.stl
├── rear-cover.stl
├── pole-mount.stl
└── speaker-pod.f3d  (Fusion 360 source)
```

## Assembly Order
1. Print/build enclosure parts
2. Install threaded inserts
3. Test fit all electronics
4. Mount driver (with gasket)
5. Wire all electronics
6. Add polyfill to acoustic volume
7. Seal electronics compartment
8. Final assembly

## Success Criteria
- [ ] All components fit
- [ ] Driver sealed properly
- [ ] Electronics accessible
- [ ] Pole mount secure
- [ ] Weather resistant

**→ Proceed to D-4.4**
