# Implementation Guide: D-1.1 ESP32 Dev Environment Setup

## Overview
**Time Estimate:** 2-4 hours  
**Skill Level:** Beginner  
**Cost:** $20

---

## Step 1: Order Parts (Day 1)

### Shopping List
| Item | Link | Notes |
|------|------|-------|
| ESP32-S3-DevKitC-1 (×2) | [Amazon](https://amazon.com/dp/B0B6HT7V4M) | Must be S3 variant |
| USB-C cables (×2) | Any | Data cables, not charge-only |

### Verification
- [ ] Ordered 2× ESP32-S3-DevKitC-1
- [ ] Ordered USB-C data cables

---

## Step 2: Install Development Environment (While Waiting)

### Option A: Arduino IDE (Recommended for Beginners)

1. **Download Arduino IDE 2.x**
   - https://www.arduino.cc/en/software
   - Install for your OS

2. **Add ESP32 Board Support**
   ```
   File → Preferences → Additional Board Manager URLs
   ```
   Add: `https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json`

3. **Install ESP32 Boards**
   ```
   Tools → Board → Boards Manager
   Search: "esp32"
   Install: "esp32 by Espressif Systems" (v2.0.11+)
   ```

4. **Select Board**
   ```
   Tools → Board → esp32 → ESP32S3 Dev Module
   ```

### Option B: ESP-IDF (Advanced)

```bash
# Linux/Mac
mkdir -p ~/esp
cd ~/esp
git clone -b v5.1.2 --recursive https://github.com/espressif/esp-idf.git
cd esp-idf
./install.sh esp32s3
source export.sh
```

### Verification
- [ ] IDE installed
- [ ] ESP32 board package installed
- [ ] Can see ESP32S3 in board list

---

## Step 3: First Board Test (When Parts Arrive)

### 3.1 Connect First ESP32

1. Plug USB-C into ESP32 (use the **USB** port, not UART)
2. Connect to computer
3. Check device detected:
   - **Windows:** Device Manager → Ports → "USB Serial"
   - **Mac:** `ls /dev/cu.usb*`
   - **Linux:** `ls /dev/ttyUSB*` or `ls /dev/ttyACM*`

### 3.2 Flash Blink Test

```cpp
// File: blink_test.ino

#define LED_PIN 2  // Onboard LED (may vary)

void setup() {
  Serial.begin(115200);
  pinMode(LED_PIN, OUTPUT);
  Serial.println("ESP32-S3 Blink Test");
}

void loop() {
  digitalWrite(LED_PIN, HIGH);
  Serial.println("LED ON");
  delay(500);
  
  digitalWrite(LED_PIN, LOW);
  Serial.println("LED OFF");
  delay(500);
}
```

### 3.3 Upload
```
Tools → Port → (select your port)
Tools → Upload Speed → 921600
Click Upload button
```

### 3.4 Verify
- [ ] Code compiles without errors
- [ ] Upload completes
- [ ] LED blinks
- [ ] Serial monitor shows "LED ON/OFF"

---

## Step 4: Get MAC Addresses

### Flash to BOTH boards:

```cpp
// File: get_mac.ino

#include <WiFi.h>

void setup() {
  Serial.begin(115200);
  delay(1000);
  
  WiFi.mode(WIFI_STA);
  
  Serial.println("

=== ESP32-S3 MAC Address ===");
  Serial.print("MAC: ");
  Serial.println(WiFi.macAddress());
  Serial.println("============================
");
}

void loop() {
  delay(10000);
  Serial.println(WiFi.macAddress());
}
```

### Record Your MACs
```
Board 1 (TX): ___:___:___:___:___:___
Board 2 (RX): ___:___:___:___:___:___
```

---

## Step 5: ESP-NOW Communication Test

### 5.1 Flash TX Board (Board 1)

```cpp
// File: espnow_tx.ino

#include <esp_now.h>
#include <WiFi.h>

// REPLACE with your RX board MAC address
uint8_t rxMAC[] = {0xXX, 0xXX, 0xXX, 0xXX, 0xXX, 0xXX};

typedef struct {
  uint32_t counter;
  uint32_t timestamp;
} Message;

Message msg;
uint32_t successCount = 0;
uint32_t failCount = 0;

void onSent(const uint8_t *mac, esp_now_send_status_t status) {
  if (status == ESP_NOW_SEND_SUCCESS) {
    successCount++;
  } else {
    failCount++;
  }
}

void setup() {
  Serial.begin(115200);
  WiFi.mode(WIFI_STA);
  
  if (esp_now_init() != ESP_OK) {
    Serial.println("ESP-NOW init failed!");
    return;
  }
  
  esp_now_register_send_cb(onSent);
  
  esp_now_peer_info_t peerInfo = {};
  memcpy(peerInfo.peer_addr, rxMAC, 6);
  peerInfo.channel = 0;
  peerInfo.encrypt = false;
  
  if (esp_now_add_peer(&peerInfo) != ESP_OK) {
    Serial.println("Failed to add peer");
    return;
  }
  
  Serial.println("TX Ready - Sending packets...");
}

void loop() {
  msg.counter++;
  msg.timestamp = micros();
  
  esp_now_send(rxMAC, (uint8_t *)&msg, sizeof(msg));
  
  Serial.printf("Sent #%lu | Success: %lu | Fail: %lu
", 
                msg.counter, successCount, failCount);
  
  delay(100);  // 10 packets/second
}
```

### 5.2 Flash RX Board (Board 2)

```cpp
// File: espnow_rx.ino

#include <esp_now.h>
#include <WiFi.h>

typedef struct {
  uint32_t counter;
  uint32_t timestamp;
} Message;

uint32_t lastCounter = 0;
uint32_t receivedCount = 0;
uint32_t lostCount = 0;

void onReceive(const uint8_t *mac, const uint8_t *data, int len) {
  Message *msg = (Message *)data;
  
  uint32_t latency = micros() - msg->timestamp;
  
  // Check for lost packets
  if (lastCounter > 0 && msg->counter != lastCounter + 1) {
    lostCount += (msg->counter - lastCounter - 1);
  }
  lastCounter = msg->counter;
  receivedCount++;
  
  Serial.printf("RX #%lu | Latency: %lu us | Lost: %lu
",
                msg->counter, latency, lostCount);
}

void setup() {
  Serial.begin(115200);
  WiFi.mode(WIFI_STA);
  
  Serial.print("RX MAC: ");
  Serial.println(WiFi.macAddress());
  
  if (esp_now_init() != ESP_OK) {
    Serial.println("ESP-NOW init failed!");
    return;
  }
  
  esp_now_register_recv_cb(onReceive);
  Serial.println("RX Ready - Waiting for packets...");
}

void loop() {
  // Print stats every 5 seconds
  static uint32_t lastPrint = 0;
  if (millis() - lastPrint > 5000) {
    float lossRate = (float)lostCount / (receivedCount + lostCount) * 100;
    Serial.printf("
=== STATS: Received: %lu | Lost: %lu (%.2f%%) ===

",
                  receivedCount, lostCount, lossRate);
    lastPrint = millis();
  }
}
```

---

## Step 6: Verify & Document Results

### Expected Output

**TX Serial Monitor:**
```
TX Ready - Sending packets...
Sent #1 | Success: 1 | Fail: 0
Sent #2 | Success: 2 | Fail: 0
...
```

**RX Serial Monitor:**
```
RX MAC: AA:BB:CC:DD:EE:FF
RX Ready - Waiting for packets...
RX #1 | Latency: 1823 us | Lost: 0
RX #2 | Latency: 1756 us | Lost: 0
...
=== STATS: Received: 50 | Lost: 0 (0.00%) ===
```

### Success Criteria
| Metric | Target | Your Result |
|--------|--------|-------------|
| TX sends packets | Yes | |
| RX receives packets | Yes | |
| Latency | <5ms (5000 us) | |
| Packet loss | <1% | |

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| "No port available" | Try different USB cable (data, not charge-only) |
| Upload fails | Hold BOOT button while uploading |
| ESP-NOW init failed | Ensure WiFi.mode(WIFI_STA) called first |
| No packets received | Double-check MAC address in TX code |
| High packet loss | Move boards closer, check for interference |

---

## Completion Checklist

- [ ] Both ESP32-S3 boards flash successfully
- [ ] MAC addresses recorded
- [ ] ESP-NOW TX sends packets
- [ ] ESP-NOW RX receives packets
- [ ] Latency <5ms confirmed
- [ ] Packet loss <1% confirmed
- [ ] Results documented

**→ When complete, close Issue #1 and proceed to D-1.2**
