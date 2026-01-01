# Implementation Guide: D-3.1 HDMI Switch + Audio Extractor

## Overview
**Time Estimate:** 2-3 hours | **Cost:** $160 | **Depends On:** Phase 2 complete

---

## Objective
Set up HDMI source switching and 7.1 audio extraction.

## Shopping List
| Item | Model Recommendation | Cost |
|------|---------------------|------|
| HDMI Switch 4x1 | SGEYR 4K@60Hz | $40 |
| Audio Extractor | Monoprice Blackbird 7.1 | $80 |
| HDMI Cables (5) | 6ft High Speed | $40 |

## System Diagram
```
[PS5]──────┐
[Xbox]─────┤
[Apple TV]─┼──►[4x1 Switch]──►[Audio Extractor]──┬──►[Projector]
[Blu-ray]──┘                        │            └── HDMI Out
                                    ▼
                              [7.1 Audio Out]
                              (8-ch analog or HDMI ARC)
```

## Audio Extractor Selection Criteria
- Must support 7.1 PCM extraction (not just stereo)
- HDMI 2.0+ for 4K passthrough
- Low latency (<10ms)
- Recommended: Monoprice Blackbird, OREI HDA-912

## Wiring Procedure
1. Connect all source HDMI cables to switch inputs
2. Connect switch output to audio extractor input
3. Connect extractor HDMI out to projector
4. Connect extractor audio out to Pi audio interface

## Configuration
1. Set extractor to PCM output (not bitstream)
2. Set to 7.1 channel mode
3. Verify video passthrough works
4. Test each source individually

## Testing Each Source
| Source | Video OK | Audio OK | Notes |
|--------|----------|----------|-------|
| PS5 | [ ] | [ ] | Set to PCM in PS5 settings |
| Xbox | [ ] | [ ] | Set to Uncompressed 7.1 |
| Apple TV | [ ] | [ ] | May need audio format change |
| Blu-ray | [ ] | [ ] | Check disc audio format |

## Troubleshooting
| Issue | Solution |
|-------|----------|
| No video | Check HDMI cables, try different port |
| No audio | Verify PCM mode, check source settings |
| Only stereo | Source may be sending compressed audio |
| Lip sync issues | Some extractors have delay adjustment |

## Success Criteria
- [ ] All 4 sources switch correctly
- [ ] Video passes to projector at 4K
- [ ] 7.1 audio extracted (8 channels)
- [ ] No visible/audible sync issues

**→ Proceed to D-3.2**
