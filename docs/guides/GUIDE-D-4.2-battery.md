# Implementation Guide: D-4.2 Battery System Design

## Overview
**Time Estimate:** 3-4 hours  
**Skill Level:** Intermediate  
**Cost:** $70  
**Depends On:** D-4.1 complete

⚠️ **SAFETY WARNING:** LiFePO4 batteries are safer than Li-ion but still require careful handling. Use proper BMS and never short-circuit!

---

## Shopping List

| Item | Specs | Qty | Cost |
|------|-------|-----|------|
| LiFePO4 Cells | 32650, 3.2V, 6000mAh | 3 | $15 ea = $45 |
| 3S BMS | 10A continuous, balance | 1 | $8 |
| Buck Converter | 12V→5V, 3A | 1 | $5 |
| Cell Holder | 32650 3S | 1 | $5 |
| XT60 Connector | Pair | 1 | $3 |
| 18AWG Wire | Red/Black, 3ft each | 1 | $4 |

**Total:** $70

---

## Why LiFePO4?

| Property | LiFePO4 | Li-ion (18650) |
|----------|---------|----------------|
| Safety | Excellent | Good |
| Cycle life | 2000+ cycles | 500 cycles |
| Thermal stability | Very stable | Less stable |
| Voltage/cell | 3.2V nom | 3.7V nom |
| Energy density | Lower | Higher |
| Cost | Higher | Lower |

**Conclusion:** LiFePO4 is ideal for this application due to safety and longevity.

---

## Battery Pack Design

### Configuration

```
3S1P = 3 cells in Series, 1 Parallel

Cell 1 ──+──┬── Cell 2 ──+──┬── Cell 3 ──+──
         │  │            │  │            │
        3.2V│           3.2V│           3.2V
         │  │            │  │            │
         ─  ┴────────────┴  ┴────────────┴──
         
Pack Voltage: 3.2V × 3 = 9.6V nominal
              3.65V × 3 = 10.95V fully charged
              2.5V × 3 = 7.5V empty (cutoff)
              
Capacity: 6Ah (single parallel)
Energy: 9.6V × 6Ah = 57.6Wh
```

---

## BMS (Battery Management System)

### What BMS Does

1. **Over-discharge protection** - Cuts off below 2.5V/cell
2. **Over-charge protection** - Cuts off above 3.65V/cell
3. **Balance charging** - Equalizes cell voltages
4. **Over-current protection** - Limits discharge current
5. **Short-circuit protection** - Instant cutoff

### BMS Wiring

```
3S BMS Wiring Diagram:

       Cell 1          Cell 2          Cell 3
    ┌─────────┐     ┌─────────┐     ┌─────────┐
    │    +    │     │    +    │     │    +    │
    │  3.2V   │     │  3.2V   │     │  3.2V   │
    │    -    │     │    -    │     │    -    │
    └────┬────┘     └────┬────┘     └────┬────┘
         │               │               │
         │    ┌──────────┴───────────────┤
         │    │          ┌───────────────┤
         │    │          │               │
    ┌────┴────┴──────────┴───────────────┴────┐
    │    B-   B1        B2              B+    │
    │                                         │
    │              3S BMS                     │
    │                                         │
    │    P-                             P+    │
    └────┬──────────────────────────────┬────┘
         │                              │
         │      OUTPUT TO LOAD          │
         │     (9.6V nominal)           │
         └──────────────────────────────┘

BMS Pin Connections:
─────────────────────────────────────────────
B-  = Cell 1 negative (most negative point)
B1  = Cell 1 positive / Cell 2 negative (3.2V)
B2  = Cell 2 positive / Cell 3 negative (6.4V)  
B+  = Cell 3 positive (9.6V)
P-  = Output negative (to load)
P+  = Output positive (to load)
```

---

## Step 1: Assemble Battery Pack

### Safety First!

- [ ] Work on non-conductive surface
- [ ] Remove jewelry
- [ ] Have fire extinguisher nearby
- [ ] Never short positive to negative!

### Assembly Steps

1. **Test individual cells**
   ```
   Measure each cell voltage with multimeter
   Should be 3.2-3.3V (shipped ~50% charge)
   Reject any cell below 3.0V
   ```

2. **Insert cells into holder**
   ```
   Observe polarity markings!
   All cells same direction for series
   ```

3. **Solder balance wires**
   ```
   Use thin wire (22-24 AWG) for balance leads
   B- to Cell 1 negative
   B1 to Cell 1 positive
   B2 to Cell 2 positive
   B+ to Cell 3 positive
   ```

4. **Solder output wires**
   ```
   Use thick wire (18 AWG) for output
   P- to load negative
   P+ to load positive
   ```

---

## Step 2: Add Buck Converter

The TPA3116 runs on 9.6V directly, but ESP32 and DAC need 5V:

```
Battery Pack (9.6V)
        │
        ├────────────────────► TPA3116 VCC+ (direct)
        │
        ▼
  ┌──────────────┐
  │ Buck Convert │
  │  Input: 9.6V │
  │ Output: 5.0V │
  └──────┬───────┘
         │
         ├───────► ESP32 5V pin
         └───────► PCM5102A VCC
```

### Buck Converter Setup

1. Connect input to battery P+ and P-
2. Adjust output to exactly 5.0V using trim pot
3. Verify stable output under load

---

## Step 3: Charging System

### Option A: External Charger (Recommended)

Use a dedicated LiFePO4 3S charger:

| Charger | Cost | Notes |
|---------|------|-------|
| iMAX B6 | $30 | Versatile, adjustable |
| LiFePO4 3S Charger | $15 | Simple, automatic |

Charge parameters:
- Voltage: 10.95V (3.65V × 3)
- Current: 2A (0.3C for 6Ah pack)
- Termination: Current drops to 0.1A

### Option B: Onboard Charging

Add TP5000 charging IC:
- Input: 5V USB or 12V adapter
- Output: Charges single cell
- Need one per cell for balance, or use BMS balance feature

---

## Step 4: Runtime Calculation

### Power Budget

| Component | Voltage | Current | Power |
|-----------|---------|---------|-------|
| ESP32-S3 | 5V | 200mA | 1.0W |
| PCM5102A | 5V | 20mA | 0.1W |
| TPA3116 (idle) | 9.6V | 50mA | 0.5W |
| TPA3116 (medium) | 9.6V | 1A | 9.6W |
| **Total (medium vol)** | — | — | **~11W** |

### Runtime Estimate

```
Battery energy: 57.6Wh
Efficiency losses: ~85%
Usable energy: 57.6 × 0.85 = 49Wh

Runtime at medium volume:
49Wh / 11W = 4.4 hours

Runtime at low volume (~5W):
49Wh / 5W = 9.8 hours

Runtime at high volume (~20W):
49Wh / 20W = 2.4 hours
```

---

## Step 5: Safety Features

### Add Inline Fuse

```
Battery P+ ──► [10A Fuse] ──► Load
```

### Add Power Switch

```
Battery P+ ──► [Fuse] ──► [Switch] ──► Load
```

### Status LED (Optional)

```
Battery P+ ──► [1kΩ] ──► [LED] ──► P-
```

---

## Completion Checklist

- [ ] Cells tested and matched
- [ ] Pack assembled in holder
- [ ] BMS wired correctly
- [ ] Balance leads connected
- [ ] Output voltage verified (9.6V)
- [ ] Buck converter outputs 5.0V
- [ ] Fuse installed
- [ ] Charging tested

**→ Proceed to D-4.3 (Enclosure)**
