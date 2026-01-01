# Implementation Guide: D-1.2 I2S Audio Output Test

## Overview
**Time Estimate:** 2-3 hours  
**Skill Level:** Beginner-Intermediate  
**Cost:** $10  
**Depends On:** D-1.1 complete

---

## Step 1: Order Parts

| Item | Qty | Link | Notes |
|------|-----|------|-------|
| PCM5102A DAC Breakout | 1 | [AliExpress](https://aliexpress.com/item/32826765601.html) | ~$3-5, get the purple/blue board |
| 3.5mm Audio Cable | 1 | Amazon | Male-to-male |
| Jumper Wires | 10 | Amazon | Male-to-male recommended |

**Note:** PCM5102A takes 1-3 weeks from AliExpress. Amazon has faster options for ~$8.

---

## Step 2: Understand the PCM5102A

### Pinout Reference

```
PCM5102A Breakout Board:
┌─────────────────────────────────────┐
│  ┌───┐                              │
│  │ ● │ VCC  - 3.3V Power            │
│  │ ● │ GND  - Ground                │
│  │ ● │ BCK  - Bit Clock (I2S)       │
│  │ ● │ DIN  - Data In (I2S)         │
│  │ ● │ LCK  - L/R Clock (I2S)       │
│  │ ● │ SCK  - System Clock          │
│  │ ● │ FMT  - Format Select         │
│  │ ● │ XMT  - Soft Mute             │
│  └───┘                              │
│                          ┌────────┐ │
│                          │ 3.5mm  │ │
│                          │  OUT   │ │
│                          └────────┘ │
└─────────────────────────────────────┘
```

### Pin Functions
| Pin | Function | Connect To |
|-----|----------|------------|
| VCC | Power | ESP32 3V3 |
| GND | Ground | ESP32 GND |
| BCK | Bit Clock | ESP32 GPIO 4 |
| DIN | Audio Data | ESP32 GPIO 6 |
| LCK | Word Select | ESP32 GPIO 5 |
| SCK | System Clock | **GND** (uses internal) |
| FMT | Format | **GND** (I2S standard) |
| XMT | Mute | **3V3** (unmuted) |

---

## Step 3: Wiring

### Schematic

```
ESP32-S3-DevKitC                    PCM5102A
┌──────────────┐                   ┌──────────────┐
│              │                   │              │
│         3V3 ─┼───────────────────┼─ VCC         │
│              │         ┌─────────┼─ XMT         │
│              │         │         │              │
│         GND ─┼─────────┼─────────┼─ GND         │
│              │         │    ┌────┼─ SCK         │
│              │         │    │ ┌──┼─ FMT         │
│              │         │    │ │  │              │
│       GPIO4 ─┼─────────┼────┼─┼──┼─ BCK         │
│       GPIO5 ─┼─────────┼────┼─┼──┼─ LCK         │
│       GPIO6 ─┼─────────┼────┼─┼──┼─ DIN         │
│              │         │    │ │  │              │
└──────────────┘         │    └─┴──┘     ┌──────┐ │
                         │               │3.5mm │ │
                         └───────────────│ OUT  │─┼──► 🎧
                                         └──────┘ │
                                        └──────────────┘
```

### Breadboard Layout

```
Breadboard:
    1   5   10  15  20  25  30
    │   │   │   │   │   │   │
  ┌─┴───┴───┴───┴───┴───┴───┴─┐
  │ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ │ ← Power rail (+)
  │ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ │ ← Power rail (-)
  ├───────────────────────────┤
a │ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ │
b │ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ │
c │ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ │
d │ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ │
e │ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ │
  ├───────────────────────────┤
f │ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ │
g │ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ │
h │ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ │
i │ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ │
j │ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ │
  └───────────────────────────┘

Place PCM5102A board spanning rows d-g around column 15.
```

### Wire List
| From | To | Wire Color (suggested) |
|------|----|------------------------|
| ESP32 3V3 | + rail | Red |
| ESP32 GND | - rail | Black |
| + rail | PCM5102A VCC | Red |
| + rail | PCM5102A XMT | Red |
| - rail | PCM5102A GND | Black |
| - rail | PCM5102A SCK | Black |
| - rail | PCM5102A FMT | Black |
| ESP32 GPIO4 | PCM5102A BCK | Yellow |
| ESP32 GPIO5 | PCM5102A LCK | Green |
| ESP32 GPIO6 | PCM5102A DIN | Blue |

---

## Step 4: Test Code

### 4.1 Simple Sine Wave Test

```cpp
// File: i2s_sine_test.ino

#include "driver/i2s.h"
#include <math.h>

#define I2S_BCK_PIN  4
#define I2S_WS_PIN   5
#define I2S_DATA_PIN 6

#define SAMPLE_RATE  44100
#define TONE_FREQ    1000  // 1kHz test tone
#define AMPLITUDE    10000

void setup() {
  Serial.begin(115200);
  Serial.println("I2S Audio Test Starting...");
  
  // I2S configuration
  i2s_config_t i2s_config = {
    .mode = (i2s_mode_t)(I2S_MODE_MASTER | I2S_MODE_TX),
    .sample_rate = SAMPLE_RATE,
    .bits_per_sample = I2S_BITS_PER_SAMPLE_16BIT,
    .channel_format = I2S_CHANNEL_FMT_RIGHT_LEFT,
    .communication_format = I2S_COMM_FORMAT_STAND_I2S,
    .intr_alloc_flags = ESP_INTR_FLAG_LEVEL1,
    .dma_buf_count = 8,
    .dma_buf_len = 64,
    .use_apll = false,
    .tx_desc_auto_clear = true
  };
  
  i2s_pin_config_t pin_config = {
    .bck_io_num = I2S_BCK_PIN,
    .ws_io_num = I2S_WS_PIN,
    .data_out_num = I2S_DATA_PIN,
    .data_in_num = I2S_PIN_NO_CHANGE
  };
  
  // Install and start I2S driver
  esp_err_t err = i2s_driver_install(I2S_NUM_0, &i2s_config, 0, NULL);
  if (err != ESP_OK) {
    Serial.printf("I2S driver install failed: %d
", err);
    return;
  }
  
  err = i2s_set_pin(I2S_NUM_0, &pin_config);
  if (err != ESP_OK) {
    Serial.printf("I2S set pin failed: %d
", err);
    return;
  }
  
  Serial.println("I2S initialized successfully!");
  Serial.printf("Playing %d Hz tone at %d Hz sample rate
", TONE_FREQ, SAMPLE_RATE);
}

void loop() {
  static float phase = 0;
  const float phaseIncrement = 2.0 * PI * TONE_FREQ / SAMPLE_RATE;
  
  int16_t samples[64];
  
  for (int i = 0; i < 32; i++) {
    int16_t sample = (int16_t)(AMPLITUDE * sin(phase));
    samples[i * 2] = sample;      // Left channel
    samples[i * 2 + 1] = sample;  // Right channel
    
    phase += phaseIncrement;
    if (phase >= 2.0 * PI) phase -= 2.0 * PI;
  }
  
  size_t bytesWritten;
  i2s_write(I2S_NUM_0, samples, sizeof(samples), &bytesWritten, portMAX_DELAY);
}
```

### 4.2 Multi-Frequency Test

```cpp
// File: i2s_sweep_test.ino
// Plays multiple frequencies to verify full range

#include "driver/i2s.h"
#include <math.h>

#define I2S_BCK_PIN  4
#define I2S_WS_PIN   5
#define I2S_DATA_PIN 6
#define SAMPLE_RATE  44100
#define AMPLITUDE    10000

int frequencies[] = {440, 880, 1000, 2000, 4000, 8000};
int freqIndex = 0;

void setupI2S() {
  i2s_config_t i2s_config = {
    .mode = (i2s_mode_t)(I2S_MODE_MASTER | I2S_MODE_TX),
    .sample_rate = SAMPLE_RATE,
    .bits_per_sample = I2S_BITS_PER_SAMPLE_16BIT,
    .channel_format = I2S_CHANNEL_FMT_RIGHT_LEFT,
    .communication_format = I2S_COMM_FORMAT_STAND_I2S,
    .intr_alloc_flags = ESP_INTR_FLAG_LEVEL1,
    .dma_buf_count = 8,
    .dma_buf_len = 64,
    .use_apll = false,
    .tx_desc_auto_clear = true
  };
  
  i2s_pin_config_t pin_config = {
    .bck_io_num = I2S_BCK_PIN,
    .ws_io_num = I2S_WS_PIN,
    .data_out_num = I2S_DATA_PIN,
    .data_in_num = I2S_PIN_NO_CHANGE
  };
  
  i2s_driver_install(I2S_NUM_0, &i2s_config, 0, NULL);
  i2s_set_pin(I2S_NUM_0, &pin_config);
}

void setup() {
  Serial.begin(115200);
  setupI2S();
  Serial.println("Frequency sweep test - changes every 3 seconds");
}

void loop() {
  static uint32_t lastChange = 0;
  static float phase = 0;
  
  int freq = frequencies[freqIndex];
  float phaseIncrement = 2.0 * PI * freq / SAMPLE_RATE;
  
  // Change frequency every 3 seconds
  if (millis() - lastChange > 3000) {
    freqIndex = (freqIndex + 1) % 6;
    Serial.printf("Now playing: %d Hz
", frequencies[freqIndex]);
    lastChange = millis();
  }
  
  int16_t samples[64];
  for (int i = 0; i < 32; i++) {
    int16_t sample = (int16_t)(AMPLITUDE * sin(phase));
    samples[i * 2] = sample;
    samples[i * 2 + 1] = sample;
    phase += phaseIncrement;
    if (phase >= 2.0 * PI) phase -= 2.0 * PI;
  }
  
  size_t bytesWritten;
  i2s_write(I2S_NUM_0, samples, sizeof(samples), &bytesWritten, portMAX_DELAY);
}
```

---

## Step 5: Testing Procedure

### 5.1 Pre-Test Checklist
- [ ] Wiring double-checked against schematic
- [ ] Headphones or powered speaker ready
- [ ] Volume turned LOW initially

### 5.2 Test Sequence

1. **Upload** `i2s_sine_test.ino`
2. **Open** Serial Monitor (115200 baud)
3. **Verify** "I2S initialized successfully!" message
4. **Connect** headphones to PCM5102A 3.5mm jack
5. **Listen** for 1kHz tone (should be clear, steady beep)
6. **Upload** `i2s_sweep_test.ino`
7. **Verify** frequencies change every 3 seconds
8. **Listen** for all frequencies (440Hz to 8kHz)

### 5.3 What to Listen For

| Frequency | Should Sound Like |
|-----------|-------------------|
| 440 Hz | Musical A note, low pitch |
| 880 Hz | A note, one octave higher |
| 1000 Hz | Standard test tone |
| 2000 Hz | Higher pitch |
| 4000 Hz | High pitch |
| 8000 Hz | Very high pitch (may be hard to hear) |

---

## Step 6: Document Results

### Test Results

| Test | Expected | Actual | Pass? |
|------|----------|--------|-------|
| I2S initializes | "successfully!" | | |
| 1kHz tone plays | Clear tone | | |
| No crackling/pops | Silent background | | |
| Left channel works | Audio in left ear | | |
| Right channel works | Audio in right ear | | |
| All frequencies audible | 440-8000 Hz | | |

---

## Troubleshooting

| Symptom | Cause | Solution |
|---------|-------|----------|
| No audio at all | Wiring wrong | Check all connections |
| | XMT not connected to 3V3 | DAC is muted |
| | Wrong pins in code | Verify GPIO numbers |
| Crackling/popping | Buffer underrun | Increase dma_buf_count |
| | Ground loop | Use common ground |
| Very quiet | Amplitude too low | Increase AMPLITUDE value |
| Only one channel | Channel format wrong | Check I2S_CHANNEL_FMT |
| Distorted | Amplitude too high | Decrease AMPLITUDE |
| High-pitched whine | Clock issues | Try use_apll = true |

---

## Completion Checklist

- [ ] PCM5102A wired correctly
- [ ] I2S initializes without errors
- [ ] 1kHz test tone plays clearly
- [ ] No noise or distortion
- [ ] Both L/R channels work
- [ ] Multiple frequencies tested

**→ When complete, close Issue #2 and proceed to D-1.3**
