# Phase 1: Wireless Proof of Concept

**Budget:** $45  
**Timeline:** 1-2 weeks  
**Status:** 🟡 Active  

---

## Objective

Validate that ESP-NOW can stream audio with acceptable latency and quality for a wireless speaker system.

---

## Entry Criteria

- [ ] Decision made to proceed with ESP-NOW approach
- [ ] Basic electronics tools available (soldering iron, multimeter)
- [ ] Computer with USB ports for development

---

## Deliverables

| ID | Title | Cost | Status | Link |
|----|-------|------|--------|------|
| D-1.1 | ESP32 Dev Environment Setup | $20 | ⬜ | [Doc](../deliverables/D-1.1-dev-environment.md) |
| D-1.2 | I2S Audio Output Test | $10 | ⬜ | [Doc](../deliverables/D-1.2-i2s-audio-output.md) |
| D-1.3 | ESP-NOW Audio Streaming | $15 | ⬜ | [Doc](../deliverables/D-1.3-espnow-audio-stream.md) |

---

## Architecture

```
┌─────────────────┐         ESP-NOW         ┌─────────────────┐
│    TX ESP32     │      ═══════════►       │    RX ESP32     │
│                 │        2.4 GHz          │                 │
│  Test Tone or   │                         │     PCM5102A    │
│  Audio Input    │                         │       DAC       │
│                 │                         │        │        │
└─────────────────┘                         │        ▼        │
                                            │    Headphones   │
                                            └─────────────────┘
```

---

## Success Criteria

| Metric | Target | Measured |
|--------|--------|----------|
| End-to-end latency | < 30ms | — |
| Packet loss (10m) | < 1% | — |
| Audio quality | Acceptable | — |
| Indoor range | > 15m | — |

---

## Go/No-Go Gate

**Question:** Can ESP-NOW reliably stream audio with < 30ms latency?

| Answer | Action |
|--------|--------|
| ✅ Yes | Proceed to Phase 2 (Multi-Channel Sync) |
| ❌ No | Investigate alternatives: WiFi multicast, custom RF, wired fallback |

---

## Risks

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| ESP-NOW latency too high | Low | High | Pre-research shows <20ms achievable |
| Packet loss causes dropouts | Medium | Medium | Implement jitter buffer |
| I2S configuration issues | Medium | Low | Well-documented, many examples |

---

## Exit Criteria

- [ ] All three deliverables complete
- [ ] Success criteria met
- [ ] Go/No-Go decision documented
- [ ] Lessons learned captured

---

## Notes

- This phase is intentionally cheap ($45) to validate the core concept before investing more
- If ESP-NOW fails, alternatives exist but will change the architecture significantly
- The jitter buffer size is the main latency vs. reliability tradeoff

---

## Timeline

```
Week 1:
├── D-1.1: Dev environment (1-2 days)
├── D-1.2: I2S audio test (1-2 days)
└── D-1.3: ESP-NOW streaming (3-4 days)

Week 2:
├── Testing and optimization
├── Documentation
└── Go/No-Go decision
```
