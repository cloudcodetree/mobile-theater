# Implementation Guide: D-3.1 HDMI Switch + Audio Extractor

## Overview
**Time Estimate:** 3-4 hours  
**Skill Level:** Beginner  
**Cost:** $160  
**Depends On:** Phase 2 complete

---

## Objective

Set up HDMI input switching for 4 sources and extract 7.1 channel audio for processing.

---

## Shopping List

| Item | Model/Specs | Cost | Link |
|------|-------------|------|------|
| HDMI Switch | 4×1, 4K@60Hz, HDMI 2.0 | $40 | [Amazon](https://amazon.com/dp/B07RK6BNFT) |
| Audio Extractor | 7.1ch, eARC/ARC support | $80 | [Amazon](https://amazon.com/dp/B08HDLN8LM) |
| HDMI Cables | 6ft High Speed (×5) | $30 | Amazon |
| RCA Cables | 8-channel (4 stereo pairs) | $10 | Amazon |

**Total:** $160

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         HDMI ROUTING                                    │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  SOURCES                  SWITCH              EXTRACTOR      OUTPUT     │
│  ───────                  ──────              ─────────      ──────     │
│                                                                         │
│  ┌─────┐                                                                │
│  │ PS5 │────HDMI────┐                                                  │
│  └─────┘            │     ┌──────────┐     ┌──────────┐                │
│                     ├────►│          │     │          │    ┌─────────┐ │
│  ┌─────┐            │     │  4×1     │────►│  Audio   │───►│Projector│ │
│  │Xbox │────HDMI────┤     │  Switch  │     │  Extract │    └─────────┘ │
│  └─────┘            │     │          │     │          │                │
│                     ├────►│          │     │   ┌──────┤                │
│  ┌─────┐            │     └──────────┘     │   │ 7.1  │                │
│  │ ATV │────HDMI────┤                      │   │ PCM  │    ┌────────┐  │
│  └─────┘            │                      │   │ Out  │───►│  Pi 5  │  │
│                     │                      │   └──────┘    └────────┘  │
│  ┌─────┐            │                      │                           │
│  │ BD  │────HDMI────┘                      └───────────┘               │
│  └─────┘                                                                │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Step 1: HDMI Switch Selection

### Requirements

| Feature | Required | Why |
|---------|----------|-----|
| 4K@60Hz | ✓ | Future-proof, PS5/Xbox support |
| HDMI 2.0+ | ✓ | HDR support |
| Audio passthrough | ✓ | Don't strip audio |
| IR Remote | Nice | Convenient switching |
| Auto-switch | Nice | Detects active input |

### Recommended Models

1. **SGEYR 4×1 HDMI Switch** (~$40)
   - 4K@60Hz, HDR
   - IR remote included
   - Auto-switching

2. **Kinivo 550BN** (~$50)
   - 5 inputs
   - Highly rated

3. **Zettaguard 4×1** (~$35)
   - Budget option
   - Basic but works

---

## Step 2: Audio Extractor Selection

### Critical: 7.1 Channel Support

Most cheap extractors only do **stereo**! You need one that outputs **7.1 PCM**.

### Requirements

| Feature | Required | Why |
|---------|----------|-----|
| 7.1 channel output | ✓ | Full surround |
| HDMI passthrough | ✓ | Video to projector |
| PCM output | ✓ | Uncompressed audio |
| 4K passthrough | ✓ | Don't downgrade video |

### Recommended Models

1. **Monoprice Blackbird 4K** (~$80)
   - 7.1 analog output
   - 4K passthrough
   - Decodes Dolby/DTS

2. **J-Tech Digital JTECH-18G** (~$120)
   - 18Gbps HDMI 2.0
   - 7.1 + optical out
   - Better build quality

3. **OREI HDA-912** (~$90)
   - 7.1 analog + digital
   - 4K@60Hz HDR

### Output Connections

```
Audio Extractor Rear Panel:
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  [HDMI IN]  [HDMI OUT]  [OPTICAL]  [COAX]                  │
│                                                             │
│  ANALOG 7.1 OUTPUT:                                         │
│  ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐  │
│  │ FL │ │ FR │ │ C  │ │LFE │ │ SL │ │ SR │ │ RL │ │ RR │  │
│  └────┘ └────┘ └────┘ └────┘ └────┘ └────┘ └────┘ └────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Step 3: Physical Setup

### Wiring Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   [PS5]───HDMI───┐                                             │
│                  │                                              │
│   [Xbox]──HDMI───┼───►[HDMI Switch]───HDMI───►[Audio Extract]  │
│                  │         │                        │           │
│   [ATV]───HDMI───┤         │                        ├──HDMI──►[Projector]
│                  │      [IR Rcvr]                   │           │
│   [BD]────HDMI───┘                                  │           │
│                                                     │           │
│                              ┌──────────────────────┘           │
│                              │                                  │
│                              ▼                                  │
│                    [8× RCA to Pi Audio Interface]              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Cable Management Tips

1. Label each HDMI cable (PS5, Xbox, etc.)
2. Use cable ties to bundle
3. Leave slack for reconfiguration
4. Keep HDMI cables under 6ft for reliability

---

## Step 4: Test Each Source

### Test Procedure

For each source (PS5, Xbox, Apple TV, Blu-ray):

1. [ ] Power on source
2. [ ] Switch selects correct input (auto or manual)
3. [ ] Video appears on projector/TV
4. [ ] Audio settings show 7.1/5.1 available
5. [ ] Audio plays from all 8 analog outputs

### Test Content

- **PS5/Xbox:** Game with surround (any action game)
- **Apple TV:** Movie with 5.1/7.1 (Netflix, Disney+)
- **Blu-ray:** Known 7.1 disc (Atmos demo disc)

### Verification Checklist

| Source | Video OK | Audio Detected | 7.1 Output |
|--------|----------|----------------|------------|
| PS5 | [ ] | [ ] | [ ] |
| Xbox | [ ] | [ ] | [ ] |
| Apple TV | [ ] | [ ] | [ ] |
| Blu-ray | [ ] | [ ] | [ ] |

---

## Step 5: Configure Audio Settings

### PS5 Audio Settings

```
Settings → Sound → Audio Output
- HDMI Device Type: AV Amplifier
- Number of Channels: 7.1
- Audio Format: Linear PCM
```

### Xbox Audio Settings

```
Settings → General → Volume & Audio Output
- HDMI Audio: 7.1 Uncompressed
- Bitstream Format: Off (use PCM)
```

### Apple TV Settings

```
Settings → Video and Audio
- Audio Format: Change Format → Off
- (Forces PCM output)
```

---

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| No video | Bad HDMI cable | Try different cable |
| No audio | Extractor not decoding | Check audio format settings |
| Only stereo | Source set to stereo | Change to 7.1 in source settings |
| Audio delay | Processing lag | Usually <50ms, acceptable |
| Switching slow | HDMI handshake | Normal, wait 2-3 seconds |

---

## Completion Checklist

- [ ] HDMI switch installed
- [ ] Audio extractor installed  
- [ ] All 4 sources connected
- [ ] Video passes to display
- [ ] 7.1 audio extracted
- [ ] All sources tested

**→ Proceed to D-3.2 (Raspberry Pi 5 Setup)**
