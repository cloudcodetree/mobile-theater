# Implementation Guide: D-4.3 Speaker Pod Enclosure

## Overview
**Time Estimate:** 6-10 hours  
**Skill Level:** Intermediate  
**Cost:** $40  
**Depends On:** D-4.1, D-4.2 complete

---

## Design Requirements

| Requirement | Target | Notes |
|-------------|--------|-------|
| Internal volume | 2-3 liters | For 4" driver |
| Driver cutout | 100mm | Dayton ND105 |
| Component space | Fits all electronics | See layout |
| Weight | <2 kg empty | Portable |
| Weatherproof | IPX4 splash resistant | Outdoor use |

---

## Enclosure Options

### Option A: 3D Printed (Recommended for Prototype)

**Pros:** Custom fit, easy iteration, no tools needed
**Cons:** Print time, layer adhesion concerns

**Material:** PETG (weather resistant)
**Settings:**
- Layer height: 0.2mm
- Infill: 30% minimum
- Walls: 4 perimeters (2mm thick)
- Print time: ~20-30 hours per half

### Option B: Wood/MDF

**Pros:** Traditional speaker material, easy to work
**Cons:** Requires tools, heavier

**Material:** 1/2" MDF
**Finish:** Paint + clear coat for weather protection

### Option C: Commercial Modification

**Pros:** Ready-made, durable
**Cons:** May not fit perfectly

**Options:**
- Small ammo can
- Food storage container
- Commercial speaker cabinet

---

## Enclosure Dimensions

```
┌─────────────────────────────────────────────────────────────────┐
│                    SPEAKER POD DIMENSIONS                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  FRONT VIEW:              SIDE VIEW:              TOP VIEW:     │
│  ┌───────────┐            ┌───────┐              ┌───────────┐ │
│  │           │            │       │              │           │ │
│  │   ┌───┐   │            │       │              │           │ │
│  │   │ ● │   │ 200mm      │       │ 150mm        │           │ │
│  │   │   │   │            │       │              │   150mm   │ │
│  │   └───┘   │            │       │              │           │ │
│  │   100mm   │            │       │              └───────────┘ │
│  │           │            └───────┘                            │
│  └───────────┘                                                 │
│      150mm                                                      │
│                                                                 │
│  Volume = 150 × 150 × 200 = 4.5L (minus components ≈ 2.5L)    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Internal Layout

```
┌─────────────────────────────────────────────────────────────────┐
│                    INTERNAL COMPONENT LAYOUT                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  FRONT (driver side):                                          │
│  ┌─────────────────────────────────────┐                       │
│  │                                     │                       │
│  │            ┌───────────┐            │                       │
│  │            │           │            │                       │
│  │            │  DRIVER   │            │                       │
│  │            │   4"      │            │                       │
│  │            │           │            │                       │
│  │            └───────────┘            │                       │
│  │                                     │                       │
│  └─────────────────────────────────────┘                       │
│                                                                 │
│  REAR (electronics side):                                       │
│  ┌─────────────────────────────────────┐                       │
│  │  ┌─────────┐     ┌─────────┐       │                       │
│  │  │ Battery │     │ TPA3116 │       │                       │
│  │  │  Pack   │     │   Amp   │       │                       │
│  │  │ 3S LFP  │     └─────────┘       │                       │
│  │  │         │                        │                       │
│  │  └─────────┘     ┌─────────┐       │                       │
│  │                  │ ESP32 + │       │                       │
│  │  ┌─────────┐     │   DAC   │       │                       │
│  │  │  Buck   │     └─────────┘       │                       │
│  │  │  Conv   │                        │                       │
│  │  └─────────┘     [PWR] [CHRG]      │ ← External ports      │
│  └─────────────────────────────────────┘                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Step 1: 3D Print Design (If Using Option A)

### STL Files to Create

1. **Front half** - Driver mounting face
2. **Rear half** - Electronics compartment
3. **Gasket** - TPU seal between halves

### OpenSCAD Basic Design

```openscad
// speaker_pod.scad

// Dimensions
width = 150;
height = 200;
depth = 150;
wall = 3;
driver_dia = 100;
driver_depth = 40;

// Front half with driver cutout
module front_half() {
    difference() {
        // Outer shell
        cube([width, height, depth/2]);
        
        // Inner cavity
        translate([wall, wall, wall])
            cube([width-2*wall, height-2*wall, depth/2]);
        
        // Driver cutout
        translate([width/2, height/2, 0])
            cylinder(d=driver_dia, h=wall+1, $fn=64);
        
        // Driver mounting holes
        for (a = [0:90:270]) {
            rotate([0, 0, a])
                translate([driver_dia/2+10, 0, 0])
                    cylinder(d=4, h=wall+1, $fn=32);
        }
    }
}

front_half();
```

---

## Step 2: Driver Mounting

### Mounting Options

1. **Flush mount** - Driver face sits in routed recess
2. **Surface mount** - Driver sits on front face
3. **Rear mount** - Driver inside, visible from front

### Seal the Driver

```
Driver Sealing:
┌─────────────────────────────────────────┐
│                                         │
│  1. Apply foam gasket tape around       │
│     driver flange                       │
│                                         │
│  2. Secure with 4 screws (M4)          │
│                                         │
│  3. Verify airtight seal               │
│     (push on cone, should resist)       │
│                                         │
└─────────────────────────────────────────┘
```

---

## Step 3: Electronics Mounting

### Mounting Points

| Component | Method | Notes |
|-----------|--------|-------|
| ESP32 | M2.5 standoffs | 4mm height |
| PCM5102A | Double-sided tape | Or standoffs |
| TPA3116 | M3 standoffs | Include heatsink clearance |
| Battery | Foam padding | Or custom holder |
| Buck converter | Velcro | Easy replacement |

---

## Step 4: External Ports

### Required Ports

| Port | Type | Location |
|------|------|----------|
| Power switch | Rocker, 6A | Side or rear |
| Charge port | 5.5×2.1mm barrel | Rear |
| Status LED | 5mm | Front or top |

### Port Panel

```
Rear Panel Layout:
┌─────────────────────────────────────────┐
│                                         │
│     [PWR]        [CHARGE]        [LED]  │
│      ○              ○              ◉    │
│    Switch       DC Jack          Status │
│                                         │
└─────────────────────────────────────────┘
```

---

## Step 5: Assembly

### Assembly Order

1. Install driver in front half
2. Mount electronics in rear half
3. Wire all connections
4. Test before closing
5. Apply gasket between halves
6. Secure with screws
7. Final test

### Wire Routing

- Keep power wires away from audio wires
- Use zip ties for cable management
- Leave slack for servicing

---

## Completion Checklist

- [ ] Enclosure fabricated/printed
- [ ] Driver mounted and sealed
- [ ] Electronics mounted
- [ ] Ports installed
- [ ] All wiring complete
- [ ] Enclosure sealed
- [ ] External finish applied

**→ Proceed to D-4.4 (Assembly & Test)**
