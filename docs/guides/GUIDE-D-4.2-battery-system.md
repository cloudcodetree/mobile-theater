# Implementation Guide: D-4.2 Battery System Design

## Overview
**Time Estimate:** 4-6 hours | **Cost:** $70 | **Depends On:** D-4.1

---

## Parts
| Item | Specs | Qty | Cost |
|------|-------|-----|------|
| LiFePO4 Cell | 32650, 3.2V, 6Ah | 3 | $45 |
| 3S BMS | 10A continuous, 20A peak | 1 | $8 |
| Buck Converter | 5V 3A output | 1 | $5 |
| XT60 Connectors | Male + Female | 2 | $3 |
| Battery Holder | 3S 32650 | 1 | $5 |
| Charging Port | 5.5x2.1mm barrel | 1 | $2 |
| Wire | 16 AWG silicone | 1m | $2 |

## Why LiFePO4?
| Feature | LiFePO4 | Li-Ion |
|---------|---------|--------|
| Safety | Very safe | Can be dangerous |
| Cycle life | 2000+ | 500 |
| Voltage | 3.2V nominal | 3.7V nominal |
| Fire risk | Very low | Higher |

## Cell Configuration
```
3S1P = 3 cells in Series, 1 Parallel
Voltage: 3.2V × 3 = 9.6V nominal
         3.6V × 3 = 10.8V fully charged
         2.8V × 3 = 8.4V cutoff
Capacity: 6Ah (single cell)
Energy: 9.6V × 6Ah = 57.6Wh
```

## BMS Wiring
```
         ┌─────────────────────────────────────┐
         │              BMS                    │
         │  B- B1 B2 B3 B+    P- P+ C- C+    │
         └───┬──┬──┬──┬───────┬──┬──┬──┬─────┘
             │  │  │  │       │  │  │  │
    ┌────────┘  │  │  │       │  │  │  └── Charger +
    │     ┌─────┘  │  │       │  │  └───── Charger -
    │     │   ┌────┘  │       │  └──────── Output +
    │     │   │  ┌────┘       └─────────── Output -
    │     │   │  │
  ┌─┴─┐ ┌─┴─┐ ┌─┴─┐ ┌─┴─┐
  │ - │ │ + │ │ + │ │ + │
  │   │ │ - │ │ - │ │ - │
  │ 1 │ │ 2 │ │ 3 │ │   │
  └───┘ └───┘ └───┘ └───┘
  Cell1  Cell2  Cell3
```

## Power Distribution
```
Battery Pack (9.6V)
       │
       ├─── [XT60] ─── To TPA3116 Amp (direct)
       │
       └─── [Buck Converter]
                 │
                 └─── 5V to ESP32 + PCM5102A
```

## Runtime Calculation
```
Component Power:
- ESP32: 0.2A × 5V = 1W
- PCM5102A: 0.05A × 5V = 0.25W
- TPA3116 (avg): 2A × 9.6V = 19W
- Buck overhead: ~10% loss

Total: ~22W average

Runtime: 57.6Wh / 22W = 2.6 hours
At lower volume: 3-4 hours
```

## Charging
- Use 12.6V charger (4.2V × 3 for LiFePO4 balance)
- Actually for LiFePO4: 10.8-11.4V charger
- BMS handles balancing

## Assembly Steps
1. Test each cell voltage (should be ~3.2-3.3V)
2. Place cells in holder
3. Connect cells in series
4. Connect BMS balance wires (B-, B1, B2, B+)
5. Connect BMS output (P-, P+)
6. Add XT60 connector to output
7. Wire buck converter to output
8. Test voltages at each stage

## Safety Testing
- [ ] No shorts before connecting BMS
- [ ] BMS cuts off at low voltage
- [ ] No excessive heat during charging
- [ ] Pack voltage stable under load

**→ Proceed to D-4.3**
