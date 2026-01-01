# Phase 1: Wireless Proof of Concept

| Field | Value |
|-------|-------|
| **Status** | 🟡 In Progress |
| **Budget** | $45 |
| **Duration** | 1-2 weeks |
| **Risk Level** | Medium |

## Objective

Prove that ESP-NOW can transmit audio with acceptable latency (<30ms) between two ESP32-S3 boards.

## Entry Criteria
- [ ] Development environment decision made (ESP-IDF vs Arduino)
- [ ] ESP32-S3 boards ordered/received
- [ ] Basic understanding of I2S protocol

## Deliverables

| ID | Title | Cost | Status |
|----|-------|------|--------|
| D-1.1 | [ESP32 Dev Environment Setup](../deliverables/D-1.1-esp32-dev-setup.md) | $20 | ⬜ |
| D-1.2 | [I2S Audio Output Test](../deliverables/D-1.2-i2s-audio-test.md) | $10 | ⬜ |
| D-1.3 | [ESP-NOW Audio Streaming](../deliverables/D-1.3-espnow-audio.md) | $15 | ⬜ |

## Architecture

```
PHASE 1 TEST SETUP:

    [PC/Phone]                    [Powered Speaker]
        │                               ▲
        │ USB/BT Audio                  │ 3.5mm
        ▼                               │
    ┌─────────┐    ESP-NOW      ┌─────────┐
    │ ESP32   │ ─────────────►  │ ESP32   │
    │ TX      │    2.4GHz       │ RX      │
    │         │                 │    │    │
    └─────────┘                 └────┼────┘
                                     │ I2S
                                     ▼
                                ┌─────────┐
                                │ PCM5102 │
                                │ DAC     │
                                └─────────┘
```

## Bill of Materials

| Item | Qty | Unit | Total | Source | Status |
|------|-----|------|-------|--------|--------|
| ESP32-S3-DevKitC-1 | 2 | $10 | $20 | Amazon/Mouser | ⬜ |
| PCM5102A DAC Breakout | 1 | $5 | $5 | AliExpress | ⬜ |
| 3.5mm Audio Cable | 1 | $3 | $3 | Amazon | ⬜ |
| Breadboard | 2 | $2 | $4 | Amazon | ⬜ |
| Jumper Wires | 1 | $5 | $5 | Amazon | ⬜ |
| USB-C Cables | 2 | $4 | $8 | Amazon | ⬜ |
| **Total** | | | **$45** | | |

## Go/No-Go Criteria

### ✅ GO if:
- Audio streams with <30ms latency
- Packet loss < 1%
- Audio quality acceptable (no constant crackling)
- Range adequate (10+ meters line of sight)

### ❌ NO-GO if:
- Latency consistently >50ms
- Frequent dropouts (>5% packet loss)
- Sync between multiple receivers impossible

### 🔄 PIVOT if NO-GO:
- Investigate WiFi multicast
- Consider custom RF solution (nRF24L01)
- Evaluate commercial wireless audio modules

## Exit Criteria
- [ ] D-1.1 complete: Both ESP32s programmed and communicating
- [ ] D-1.2 complete: I2S audio playing through DAC
- [ ] D-1.3 complete: Audio streaming wirelessly TX→RX
- [ ] Latency measured and documented
- [ ] Go/No-Go decision made

## Risks

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| ESP-NOW bandwidth insufficient | Medium | High | Test early, pivot to WiFi |
| I2S configuration complex | Low | Medium | Use known-good examples |
| DAC quality issues | Low | Low | Use proven PCM5102A |

## References

- [ESP-NOW Documentation](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/network/esp_now.html)
- [ESP32 I2S API](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/peripherals/i2s.html)
- [PCM5102A Datasheet](https://www.ti.com/lit/ds/symlink/pcm5102a.pdf)

## Log

### YYYY-MM-DD
- Phase started
