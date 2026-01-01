# Implementation Guide: D-6.3 Speaker Stands & Mounting

## Overview
**Time Estimate:** 2-3 hours | **Cost:** $110 | **Depends On:** D-6.2

---

## Stand Options
| Type | Model | Height | Cost/pair |
|------|-------|--------|-----------|
| Light Stand | Amazon Basics | 7ft | $25 |
| Speaker Stand | On-Stage SS7761B | 6ft | $60 |
| Tripod | Pyle PSTND2 | 6ft | $40 |
| Telescoping | Ultimate Support | 6ft | $100 |

## Recommended Setup
- 4× Light stands for surrounds ($50)
- 2× Speaker stands for fronts ($60)
- No stand needed for center (on table/shelf)

## Mounting Bracket
Most stands have 35mm (1-3/8") pole top.
Design adapter for speaker pods:
```
┌─────────────────────┐
│    SPEAKER POD      │
└─────────┬───────────┘
          │
    ┌─────┴─────┐
    │  Bracket  │  ← 3D printed or aluminum
    │           │
    │   ○ 35mm  │  ← Fits standard pole
    │    hole   │
    └─────┬─────┘
          │
    ┌─────┴─────┐
    │   STAND   │
    │   POLE    │
    └───────────┘
```

## 3D Printed Bracket
```
// OpenSCAD or similar

difference() {
    // Outer shape
    hull() {
        cylinder(h=20, d=60);
        translate([0,0,20])
            cube([100,100,10], center=true);
    }
    // Pole hole
    cylinder(h=25, d=35.5);
    // Mounting holes for speaker
    // ...
}
```

## Quick-Release Option
Add toggle clamps or wing nuts for fast setup:
```
┌─────────────────────┐
│    [Wing Nut]       │
│       ◈             │
│    SPEAKER          │
│       ◈             │
│    [Wing Nut]       │
└─────────────────────┘
```

## Height Guidelines
| Speaker | Height | Notes |
|---------|--------|-------|
| FL/FR | Ear level seated | ~3-4 ft |
| C | Below screen | Table/shelf |
| SL/SR | Ear level +1ft | ~4-5 ft |
| RL/RR | Ear level +1ft | ~4-5 ft |
| SUB | Floor | No stand needed |

## Collapsing for Transport
1. Remove speakers from stands
2. Collapse stand legs
3. Bundle with velcro straps
4. Store in case or bag

## Stand Bag
Consider a bag for stands:
- Drum hardware bag (~$30)
- Golf travel bag
- DIY with fabric

## Success Criteria
- [ ] Stands support pod weight (3kg+)
- [ ] Quick attach/detach (<30 sec each)
- [ ] Stable on uneven ground
- [ ] Collapse to fit in case
- [ ] All heights adjustable

**→ Complete Phase 6 checkpoint**
