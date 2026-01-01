# Implementation Guide: D-3.5 Command Module Enclosure

## Overview
**Time Estimate:** 4-8 hours  
**Skill Level:** Beginner-Intermediate  
**Cost:** $75  
**Depends On:** D-3.1 through D-3.4 complete

---

## Components to House

| Component | Size (approx) | Heat | Notes |
|-----------|---------------|------|-------|
| HDMI Switch | 4"×3"×1" | Low | Front panel access |
| Audio Extractor | 5"×3"×1" | Low | Rear panel RCA |
| Raspberry Pi 5 | 3.4"×2.2"×0.8" | Medium | Needs cooling |
| Pi Active Cooler | — | — | Built into Pi |
| USB Audio Interface | 8"×5"×2" | Low | Large unit |
| ESP32 Master TX | 2"×1"×0.5" | Low | Antenna access |
| Power Strip | 12"×2"×2" | Low | Internal |

---

## Enclosure Options

### Option A: Project Box (Recommended)

| Model | Size | Cost | Pros/Cons |
|-------|------|------|-----------|
| BUD Industries | 12"×8"×4" | $30 | ABS plastic, easy to cut |
| Hammond 1591XXL | 10"×8"×4" | $25 | Good ventilation options |
| Generic Project Box | Various | $15-30 | Amazon/AliExpress |

### Option B: Rack Mount

| Model | Size | Cost | Notes |
|-------|------|------|-------|
| 2U Rack Case | 19"×3.5"×12" | $50 | Pro look, needs rack |
| 3U Rack Case | 19"×5.25"×12" | $60 | More room |

### Option C: Pelican Case Lid-Mount

Mount components inside the Pelican 1510 lid using dense foam as a base.

---

## Layout Design

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         COMMAND MODULE LAYOUT                           │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  FRONT PANEL:                                                           │
│  ┌───────────────────────────────────────────────────────────────────┐ │
│  │ [HDMI 1] [HDMI 2] [HDMI 3] [HDMI 4]   [PWR]  [STATUS]            │ │
│  │    PS5     Xbox     ATV      BD        LED     LED                │ │
│  └───────────────────────────────────────────────────────────────────┘ │
│                                                                         │
│  TOP VIEW (inside):                                                     │
│  ┌───────────────────────────────────────────────────────────────────┐ │
│  │                                                                   │ │
│  │   ┌─────────────┐    ┌─────────────┐    ┌─────────────────────┐  │ │
│  │   │   HDMI      │    │   Audio     │    │                     │  │ │
│  │   │   Switch    │    │   Extract   │    │   USB Audio         │  │ │
│  │   │             │    │             │    │   Interface         │  │ │
│  │   └─────────────┘    └─────────────┘    │                     │  │ │
│  │                                          └─────────────────────┘  │ │
│  │   ┌─────────────┐    ┌─────────────┐    ┌─────────────────────┐  │ │
│  │   │   Pi 5      │    │   ESP32     │    │   Power Strip       │  │ │
│  │   │   + Cooler  │    │   TX        │    │                     │  │ │
│  │   └─────────────┘    └─────────────┘    └─────────────────────┘  │ │
│  │                                                                   │ │
│  │   [FAN]                                              [FAN]       │ │
│  └───────────────────────────────────────────────────────────────────┘ │
│                                                                         │
│  REAR PANEL:                                                           │
│  ┌───────────────────────────────────────────────────────────────────┐ │
│  │ [HDMI OUT] [8× RCA IN] [USB] [ETH] [POWER] [ANT] [VENT]          │ │
│  │  Projector   Audio      Pi    Pi    AC     ESP32                  │ │
│  └───────────────────────────────────────────────────────────────────┘ │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Step 1: Prepare Enclosure

### Cut Panel Openings

1. **Mark locations** with masking tape
2. **Drill pilot holes** at corners
3. **Cut with rotary tool** (Dremel) or nibbler
4. **File edges smooth**
5. **Test fit components**

### Front Panel Cutouts

| Opening | Size | For |
|---------|------|-----|
| HDMI (×4) | 15mm × 8mm | HDMI switch inputs |
| LED (×2) | 5mm hole | Status LEDs |
| Power switch | 20mm hole | Rocker switch |

### Rear Panel Cutouts

| Opening | Size | For |
|---------|------|-----|
| HDMI out | 15mm × 8mm | To projector |
| RCA (×8) | 8mm holes | Audio from extractor |
| USB | 12mm × 5mm | Pi USB |
| Ethernet | 15mm × 13mm | Pi network |
| IEC power | 50mm × 28mm | AC inlet |
| SMA antenna | 6mm hole | ESP32 external antenna |
| Vent holes | Various | Cooling |

---

## Step 2: Ventilation

### Heat Sources
- Raspberry Pi 5: ~5-10W
- USB Audio Interface: ~2W
- Total: ~15W in enclosed space

### Cooling Options

1. **Passive Vents** (minimum)
   - Front bottom: intake vents
   - Rear top: exhaust vents
   - Convection cooling

2. **Active Fans** (recommended)
   - 40mm or 60mm fans
   - Intake front, exhaust rear
   - Run from 5V or 12V

```
Air Flow:
                    ┌─────────────────────────────────┐
                    │                                 │
   Cool air ═══════►│    Components                  │═══════► Warm air
   (front intake)   │                                 │    (rear exhaust)
                    └─────────────────────────────────┘
```

---

## Step 3: Internal Wiring

### Power Distribution

```
AC Inlet ──► Fuse ──► Power Strip
                         │
                         ├──► Pi 5 USB-C (27W)
                         ├──► Audio Interface
                         ├──► HDMI Switch
                         └──► Fan (via 5V adapter)
```

### Signal Wiring

1. HDMI from front panel to switch
2. HDMI from switch to extractor
3. HDMI from extractor to rear panel
4. RCA from extractor to rear panel (or direct to audio interface)
5. USB from audio interface to Pi
6. USB from Pi to ESP32
7. Antenna cable from ESP32 to rear panel

---

## Step 4: Component Mounting

### Mounting Methods

| Component | Method | Notes |
|-----------|--------|-------|
| Pi 5 | Standoffs | M2.5 screws |
| HDMI Switch | Velcro | Easy removal |
| Audio Extract | Velcro | Easy removal |
| Audio Interface | Shelf/brackets | Larger unit |
| ESP32 | Standoffs | Near antenna |
| Power Strip | Screws | Secure mount |

### Standoff Heights

- Pi 5: 6mm standoffs
- ESP32: 4mm standoffs

---

## Step 5: External Antenna (Optional but Recommended)

For better ESP-NOW range, add external antenna:

### Parts

| Item | Cost |
|------|------|
| SMA to U.FL pigtail | $3 |
| SMA panel mount | $2 |
| 2.4GHz antenna | $5 |

### Installation

1. Remove ESP32 onboard antenna (carefully!)
2. Connect U.FL connector to ESP32 antenna pad
3. Route pigtail to rear panel SMA mount
4. Attach external antenna

---

## Step 6: Final Assembly

### Assembly Checklist

- [ ] All components mounted securely
- [ ] Power wiring complete
- [ ] Signal cables connected
- [ ] Ventilation adequate
- [ ] Lid closes properly
- [ ] All external ports accessible
- [ ] Labels applied

### Test After Assembly

- [ ] Power on, all components boot
- [ ] No overheating (run 1 hour)
- [ ] All HDMI inputs work
- [ ] Audio outputs to Pi
- [ ] ESP32 broadcasts
- [ ] Fans running (if installed)

---

## Completion Checklist

- [ ] Enclosure selected/purchased
- [ ] Panels cut
- [ ] Components mounted
- [ ] Wiring complete
- [ ] Ventilation working
- [ ] Fully tested

---

## Phase 3 Complete!

All Command Module deliverables done:

- [x] D-3.1: HDMI Switch + Audio Extractor
- [x] D-3.2: Raspberry Pi 5 Setup
- [x] D-3.3: Channel Routing Software
- [x] D-3.4: ESP32 Master TX
- [x] D-3.5: Enclosure

**→ Proceed to Phase 4 (Prototype Speaker Pod)**
