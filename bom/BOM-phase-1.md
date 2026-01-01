# Phase 1: Wireless Proof of Concept — Bill of Materials

**Phase Budget:** $45  
**Status:** 🟡 Active  

## Summary

| Category | Cost |
|----------|------|
| Development Boards | $20 |
| Audio Components | $10 |
| Cables & Misc | $15 |
| **Total** | **$45** |

---

## Development Boards

| Item | Qty | Unit | Total | Source | Status |
|------|-----|------|-------|--------|--------|
| ESP32-S3-DevKitC-1 | 2 | $10.00 | $20.00 | [Amazon](https://amazon.com) / [Mouser](https://mouser.com) | ⬜ |

**Notes:**
- Need 2 boards: 1 TX (transmitter), 1 RX (receiver)
- ESP32-S3 recommended for better I2S support
- S3-DevKitC-1 includes USB-C and onboard antenna

---

## Audio Components

| Item | Qty | Unit | Total | Source | Status |
|------|-----|------|-------|--------|--------|
| PCM5102A DAC Breakout | 1 | $5.00 | $5.00 | [Amazon](https://amazon.com) / [AliExpress](https://aliexpress.com) | ⬜ |
| 3.5mm Audio Cable | 1 | $3.00 | $3.00 | Amazon | ⬜ |
| Breadboard (400 tie) | 1 | $2.00 | $2.00 | Amazon | ⬜ |

**Notes:**
- PCM5102A is a high-quality I2S DAC, perfect for testing
- Get the breakout board version (not bare chip)

---

## Cables & Misc

| Item | Qty | Unit | Total | Source | Status |
|------|-----|------|-------|--------|--------|
| USB-C Cables | 2 | $3.00 | $6.00 | Amazon | ⬜ |
| Dupont Jumper Wires (M-M) | 1 pack | $4.00 | $4.00 | Amazon | ⬜ |
| Dupont Jumper Wires (M-F) | 1 pack | $4.00 | $4.00 | Amazon | ⬜ |
| Header Pins (optional) | 1 | $1.00 | $1.00 | Amazon | ⬜ |

---

## Optional / Nice to Have

| Item | Qty | Unit | Total | Source | Notes |
|------|-----|------|-------|--------|-------|
| Logic Analyzer | 1 | $10.00 | $10.00 | Amazon | Debug I2S signals |
| Oscilloscope (DSO138) | 1 | $25.00 | $25.00 | Amazon | Visual debugging |
| Powered Speaker | 1 | $20.00 | $20.00 | Any | For audio testing |

---

## Order Checklist

- [ ] ESP32-S3-DevKitC-1 × 2
- [ ] PCM5102A DAC Breakout × 1
- [ ] 3.5mm Audio Cable × 1
- [ ] Breadboard × 1
- [ ] USB-C Cables × 2
- [ ] Jumper Wires (M-M and M-F)

---

## Supplier Notes

| Supplier | Shipping | Notes |
|----------|----------|-------|
| Amazon | 1-2 days (Prime) | Most convenient |
| Mouser | 2-3 days | Better quality ESP32 boards |
| AliExpress | 2-4 weeks | Cheapest, slow shipping |
| Adafruit | 2-3 days | Premium quality, higher price |
