# Implementation Guide: D-6.2 Custom Foam Inserts

## Overview
**Time Estimate:** 4-6 hours | **Cost:** $120 | **Depends On:** D-6.1

---

## Foam Options

### Pick-N-Pluck (Included)
- Pre-scored cubes you pull out
- Quick but imprecise
- Good for prototyping

### Kaizen Foam
- Layered PE foam sheets
- Cut with knife/hot wire
- Professional look
- ~$30 per case

### CNC Cut Foam
- Send CAD file to company
- Perfect precision
- Most expensive (~$80+)

## Layout Planning
```
Pelican 1620 Layout (Speakers):

┌─────────────────────────────────────┐
│  ┌─────────┐      ┌─────────┐      │
│  │   FL    │      │   FR    │      │
│  │         │      │         │      │
│  └─────────┘      └─────────┘      │
│                                     │
│  ┌─────────┐      ┌─────────┐      │
│  │    C    │      │   SL    │      │
│  │         │      │         │      │
│  └─────────┘      └─────────┘      │
│                                     │
│  ┌─────────────────────────────┐   │
│  │      Accessories Layer      │   │
│  │   (Chargers, Cables, etc)   │   │
│  └─────────────────────────────┘   │
└─────────────────────────────────────┘
```

## Cutting Procedure (Kaizen)
1. Trace component outlines on foam
2. Add 1/4" margin for snug fit
3. Stack layers and mark depth
4. Cut with sharp knife or hot wire
5. Test fit each component
6. Trim as needed

## Charging Access
Design foam so pods can charge IN case:
```
Option A: Cable channels
┌─────────┐
│ Speaker │───[Cable channel to edge]
└─────────┘

Option B: Panel-mount ports
[Drill case, add panel-mount USB-C]

Option C: Remove lid to charge
[Simplest, but less convenient]
```

## Layer Strategy
```
Layer 1 (Bottom): Base padding, cable routing
Layer 2 (Middle): Main components
Layer 3 (Top):    Lid padding, small accessories
```

## Foam Materials
| Material | Use | Cost |
|----------|-----|------|
| Kaizen PE | Main inserts | $30/sheet |
| Closed-cell foam | Padding | $10 |
| Egg crate | Lid liner | $8 |

## Success Criteria
- [ ] All components fit snugly
- [ ] No rattling when shaken
- [ ] Easy to remove/replace items
- [ ] Charging accessible
- [ ] Looks professional

**→ Proceed to D-6.3**
