# Implementation Guide: D-5.1 Production Jig & Process

## Overview
**Time:** 4-6 hours | **Cost:** $50

---

## Assembly Jig Design

```
WIRING JIG (Plywood Base):
┌──────────────────────────────────────────────────────┐
│  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐    │
│  │ ESP32  │  │  DAC   │  │  Amp   │  │  Buck  │    │
│  │ Holder │  │ Holder │  │ Holder │  │ Holder │    │
│  └───┬────┘  └───┬────┘  └───┬────┘  └───┬────┘    │
│      │           │           │           │          │
│  ────┴───────────┴───────────┴───────────┴────      │
│              Wire Routing Channels                   │
│                                                      │
│  [Solder Station]              [Test Points]        │
└──────────────────────────────────────────────────────┘
```

## QC Test Firmware

```cpp
// qc_test.ino
void setup() {
  Serial.begin(115200);
  WiFi.mode(WIFI_STA);
  Serial.println("=== QC TEST ===");
  Serial.printf("MAC: %s
", WiFi.macAddress().c_str());
  // Test I2S, ESP-NOW...
  Serial.println("PASS");
}
void loop() {}
```

## Checklist per Pod
- [ ] Flash firmware with channel ID
- [ ] Verify ESP-NOW reception
- [ ] Audio output test
- [ ] Battery voltage check
- [ ] 1-hour burn-in

**→ Proceed to D-5.2**
