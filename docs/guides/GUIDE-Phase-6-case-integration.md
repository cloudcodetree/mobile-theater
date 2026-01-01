# Implementation Guide: Phase 6 - Case Integration

## Overview
**Phase Budget:** $350  
**Time Estimate:** 1-2 weeks  
**Objective:** Package everything into rugged rolling cases

---

## Case Plan

```
┌─────────────────────────────────────────────────────────────────────┐
│                      CASE LOADOUT                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  CASE 1: Command Module (Pelican 1510)                              │
│  ┌─────────────────────────────────────┐                           │
│  │ ┌─────────┐ ┌─────────┐ ┌────────┐ │                           │
│  │ │Projector│ │ HDMI    │ │ Pi 5   │ │                           │
│  │ │         │ │ Switch  │ │ + ESP32│ │                           │
│  │ └─────────┘ └─────────┘ └────────┘ │                           │
│  │ ┌─────────────────────────────────┐ │                           │
│  │ │    Cables, Remote, Accessories  │ │                           │
│  │ └─────────────────────────────────┘ │                           │
│  └─────────────────────────────────────┘                           │
│                                                                     │
│  CASE 2: Speakers A (Pelican 1620)                                  │
│  ┌─────────────────────────────────────┐                           │
│  │ ┌────┐ ┌────┐ ┌────┐ ┌────┐       │                           │
│  │ │ FL │ │ FR │ │ C  │ │ SL │       │                           │
│  │ └────┘ └────┘ └────┘ └────┘       │                           │
│  └─────────────────────────────────────┘                           │
│                                                                     │
│  CASE 3: Speakers B (Pelican 1620)                                  │
│  ┌─────────────────────────────────────┐                           │
│  │ ┌────┐ ┌────┐ ┌────────┐ ┌──────┐ │                           │
│  │ │ SR │ │ RL │ │   RR   │ │Stands│ │                           │
│  │ └────┘ └────┘ └────────┘ └──────┘ │                           │
│  └─────────────────────────────────────┘                           │
│                                                                     │
│  CASE 4: Subwoofer (Pelican 1650 or Soft Case)                     │
│  ┌─────────────────────────────────────┐                           │
│  │ ┌──────────────────────────────┐   │                           │
│  │ │         SUBWOOFER            │   │                           │
│  │ └──────────────────────────────┘   │                           │
│  │ ┌────────────┐ ┌────────────────┐  │                           │
│  │ │  Chargers  │ │ Speaker Stands │  │                           │
│  │ └────────────┘ └────────────────┘  │                           │
│  └─────────────────────────────────────┘                           │
│                                                                     │
│  SCREEN BAG: 120" Collapsible Frame                                │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## D-6.1: Case Selection

### Recommended Cases

| Case | Model | Size | Weight | Cost |
|------|-------|------|--------|------|
| Command | Pelican 1510 | 22×14×9" | 13 lbs | $180 |
| Speakers A | Pelican 1620 | 25×20×12" | 22 lbs | $280 |
| Speakers B | Pelican 1620 | 25×20×12" | 22 lbs | $280 |
| Subwoofer | Soft padded | Custom | — | $50 |

### Alternatives
- **SKB Cases** - Similar quality, sometimes cheaper
- **Nanuk** - Good value
- **Apache** (Harbor Freight) - Budget option

---

## D-6.2: Custom Foam Inserts

### Foam Options

1. **Pick-N-Pluck** (included with most cases)
   - Easy but imprecise
   - Good for prototyping

2. **Kaizen Foam**
   - Layered foam sheets
   - Cut with knife/laser
   - Professional look

3. **Custom CNC Cut**
   - Send dimensions to foam company
   - Most professional, most expensive

### Foam Layout Template

```
Pelican 1620 - Speakers A
┌─────────────────────────────────────────┐
│  ┌───────────┐  ┌───────────┐          │
│  │           │  │           │          │
│  │    FL     │  │    FR     │          │
│  │           │  │           │          │
│  └───────────┘  └───────────┘          │
│                                         │
│  ┌───────────┐  ┌───────────┐          │
│  │           │  │           │          │
│  │     C     │  │    SL     │          │
│  │           │  │           │          │
│  └───────────┘  └───────────┘          │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │     Chargers / Accessories      │   │
│  └─────────────────────────────────┘   │
└─────────────────────────────────────────┘
```

### Charger Access

Design foam so pods can charge IN the case:
- Cut channels for charging cables
- Add panel-mount charging ports
- Or use wireless charging pads

---

## D-6.3: Speaker Stands

### Requirements
- Support 3kg pods
- Adjustable height (3-6 feet)
- Quick-release mount
- Collapsible for transport

### Options

| Type | Model | Height | Cost |
|------|-------|--------|------|
| Light stands | Amazon Basics | 7ft | $25/pair |
| Speaker stands | On-Stage SS7761B | 6ft | $60/pair |
| Tripod stands | Pyle PSTND2 | 6ft | $40/pair |

### Mounting Bracket

Design a universal mount that attaches pod to any 35mm pole:

```
┌─────────────────┐
│   Speaker Pod   │
└────────┬────────┘
         │
    ┌────┴────┐
    │ Bracket │ ◄── 3D printed or aluminum
    │  35mm   │
    │  hole   │
    └────┬────┘
         │
    ┌────┴────┐
    │  Stand  │
    │  Pole   │
```

---

## Deployment Procedure

### Setup (Target: 15 minutes)

```
1. Unload cases from vehicle (2 min)
2. Set up screen (5 min)
3. Deploy speaker stands (2 min)
4. Place speakers on stands (2 min)
5. Set up command module (2 min)
6. Power on and test (2 min)
─────────────────────────────────
Total: ~15 minutes
```

### Teardown Checklist

- [ ] Power off all pods
- [ ] Collapse stands
- [ ] Return pods to foam cutouts
- [ ] Coil and store cables
- [ ] Pack screen
- [ ] Secure all case latches
- [ ] Load into vehicle

---

## Phase 6 Completion

- [ ] All cases procured
- [ ] Foam inserts cut
- [ ] All pods fit securely
- [ ] Stands functional
- [ ] Setup time <15 min
- [ ] Teardown time <10 min

**→ Proceed to Phase 7 (Calibration)**
