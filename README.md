# 🚑 SMART-AMBULANCE-COMMUNICATION-SYSTEM
> An IoT-enabled embedded system for automated ambulance traffic preemption, real-time patient vitals monitoring, and advance hospital notification using ESP32, ESP8266, LoRa SX1278, GSM SIM800L, MAX30102, and MPU6050.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [System Architecture](#system-architecture)
- [Hardware Components](#hardware-components)
- [Software Tools](#software-tools)
- [Circuit & Pin Connections](#circuit--pin-connections)
- [How It Works](#how-it-works)
- [Communication Protocols](#communication-protocols)
- [Results](#results)
- [Future Scope](#future-scope)

---

## Overview

Urban emergency medical services are critically compromised by traffic congestion, which delays ambulance response times and reduces the effectiveness of pre-hospital care. Existing traffic management systems cannot respond dynamically to approaching emergency vehicles, and hospitals receive little or no advance notice before ambulance arrival, leaving medical teams unprepared.

This project presents a **fully integrated, automated emergency response platform** operating across three interconnected nodes — ambulance, traffic junction, and hospital — without any human intervention or internet dependency at the radio communication level.

The ambulance unit broadcasts emergency beacon signals at 433 MHz via LoRa SX1278, enabling ESP8266-based junction receivers to detect the approaching ambulance up to 500 meters in advance and autonomously switch traffic signals to green, clearing a path. Simultaneously, a GSM SIM800L module transmits structured patient vitals — heart rate and SpO2 measured by the MAX30102 sensor — to the hospital via SMS, enabling medical staff to prepare before the ambulance arrives. The MPU6050 accelerometer detects ambulance accidents and triggers automatic emergency alerts to the hospital.

---

## Features

- ✅ Autonomous LoRa-based traffic signal preemption — no internet required
- ✅ Real-time patient vitals monitoring via MAX30102 (SpO2 + Heart Rate)
- ✅ GSM SIM800L hospital pre-arrival notification with patient data via SMS
- ✅ Google Voice Assistant integration for hands-free hospital search and selection
- ✅ MPU6050-based ambulance accident detection with automatic GPS alert
- ✅ 7-segment countdown display showing signal hold period at each junction
- ✅ LM2596 buck converter for stable 5V power from 12V vehicle supply
- ✅ Rolling-code packet authentication to prevent replay attacks on LoRa channel
- ✅ FreeRTOS dual-core task allocation on ESP32 for zero-conflict parallel operation
- ✅ OTA firmware update support for field-deployed ambulance units

---

## System Architecture

The system is divided into three hardware nodes, each with its own communication role:

```
┌──────────────────────────────────────────────────────────────────────┐
│  AMBULANCE NODE      ESP8266 (central hub)                           │
│                      MPU6050 (accident detection + direction)        │
│                      MAX30102 (SpO2 + Heart Rate)                    │
│                      LoRa SX1278 TX (433 MHz beacon @ 500ms)         │
│                      GSM SIM800L (patient SMS to hospital)           │
│                      Google Voice Assistant (hospital selection)     │
├──────────────────────────────────────────────────────────────────────┤
│  TRAFFIC JUNCTION    ESP32 (junction controller)                     │
│  NODE                LoRa SX1278 RX (packet validation + preemption) │
│                      4× Traffic Signal Lights (via relay)            │
│                      7-Segment Counter (hold countdown)              │
├──────────────────────────────────────────────────────────────────────┤
│  HOSPITAL NODE       ESP8266 NodeMCU (Wi-Fi polling)                 │
│                      GSM SMS Receiver (patient data display)         │
│                      Flask Backend (SQLite + REST API)               │
└──────────────────────────────────────────────────────────────────────┘
```

### Block Diagram

```
MPU6050 IMU Sensor ──┐
MAX30102 SPO2 Sensor ─┤
                      ▼
               ESP8266 NodeMCU ──► LoRa SX1278 TX ──────► LoRa SX1278 RX
               [AMBULANCE UNIT]                              │
                      │                               ESP32 MCU [SIGNAL UNIT]
                      │                                      │
               Google Voice ──► GSM SIM800L ──► HOSPITAL ◄──┴─► 7 Segment Display
               Assistant        (SMS Alert)     UNIT             4× Traffic Lights
```

---

## Hardware Components

| Component | Model | Interface | Purpose |
|---|---|---|---|
| Microcontroller (Ambulance) | ESP8266 NodeMCU CH340 | SPI / UART / I2C | Central processing hub for ambulance node |
| Microcontroller (Junction) | ESP32 | SPI / GPIO | Junction receiver and traffic light controller |
| LoRa Transceiver | SX1278 | SPI | 433 MHz emergency beacon TX/RX |
| GSM Module | SIM800L | UART | Hospital SMS notification and GPRS data |
| IMU Sensor | MPU6050 | I2C (0x68) | Accident detection + ambulance direction |
| SpO2 Sensor | MAX30102 | I2C (0x57) | Heart rate and blood oxygen saturation |
| Traffic Signals | 4× LED Signal Lights | Relay / GPIO | Emergency preemption (red/green switching) |
| Display | 7-Segment Counter | BCD / GPIO | Signal hold period countdown |
| Power Regulator | LM2596 Buck Converter | — | 12V → 5V regulation for all modules |

---

## Software Tools

| Tool | Purpose |
|---|---|
| **Arduino IDE 2.3.x** | Primary firmware development and upload for ESP32 and ESP8266 |
| **PlatformIO (VS Code)** | Final optimization, collision detection calibration, JTAG debugging |
| **Python Flask 3.0** | Hospital backend REST API server with SQLite patient database |
| **Arduino LoRa Library** | SX1278 SPI abstraction — packet TX/RX, SF/BW/power config |
| **TinyGSM Library** | SIM800L GPRS/SMS management via AT command abstraction |
| **TinyGPSPlus Library** | NMEA sentence parsing for GPS position and speed |
| **MPU6050 Library** | Accelerometer/gyroscope I2C access and DMP calibration |
| **SparkFun MAX3010x Library** | PPG data acquisition, SpO2 and heart rate computation |
| **ArduinoJson 7.x** | JSON serialization for GSM HTTP payloads |

---

## Circuit & Pin Connections

### LoRa SX1278 → ESP8266 (Ambulance TX)

| SX1278 Pin | ESP8266 Pin | Description |
|---|---|---|
| MISO | GPIO12 (D6) | SPI Data to ESP8266 |
| MOSI | GPIO13 (D7) | SPI Data from ESP8266 |
| SCK | GPIO14 (D5) | SPI Clock |
| NSS (CS) | GPIO15 (D8) | Chip Select |
| RST | GPIO16 (D0) | Module Reset |
| DIO0 | GPIO5 (D1) | Packet Ready Interrupt |

### LoRa SX1278 → ESP32 (Junction RX)

| SX1278 Pin | ESP32 Pin | Description |
|---|---|---|
| MISO | GPIO19 | SPI Data to ESP32 |
| MOSI | GPIO23 | SPI Data from ESP32 |
| SCK | GPIO18 | SPI Clock |
| NSS (CS) | GPIO5 | Chip Select |
| RST | GPIO14 | Module Reset |
| DIO0 | GPIO2 | Packet Ready Interrupt |

### GSM SIM800L → ESP8266 (Ambulance)

| SIM800L Pin | ESP8266 Pin | Description |
|---|---|---|
| TXD | GPIO3 (D3 / RX) | GSM data to ESP8266 |
| RXD | GPIO4 (D4 / TX) | Commands to GSM |
| VCC | 4.0–4.2V (external) | Dedicated LDO supply |
| GND | GND | Common ground |

### MPU6050 → ESP8266 (I2C)

| MPU6050 Pin | ESP8266 Pin | Description |
|---|---|---|
| VCC | 3.3V | Power |
| GND | GND | Ground |
| SCL | GPIO14 (D5) | I2C Clock (shared bus) |
| SDA | GPIO4 (D2) | I2C Data (shared bus) |
| AD0 | GND | I2C Address = 0x68 |

### MAX30102 → ESP8266 (I2C, shared bus)

| MAX30102 Pin | ESP8266 Pin | Description |
|---|---|---|
| VCC | 3.3V | Power |
| GND | GND | Ground |
| SCL | GPIO14 (D5) | I2C Clock (shared with MPU6050) |
| SDA | GPIO4 (D2) | I2C Data (shared with MPU6050) |

### Traffic Signals + 7-Segment → ESP32 (Junction)

| Component | ESP32 Pin | Description |
|---|---|---|
| North RED relay IN | GPIO25 | North direction red lamp |
| North GREEN relay IN | GPIO26 | North direction green lamp |
| East RED relay IN | GPIO27 | East direction red lamp |
| East GREEN relay IN | GPIO32 | East direction green lamp |
| South RED relay IN | GPIO33 | South direction red lamp |
| South GREEN relay IN | GPIO34 | South direction green lamp |
| West RED relay IN | GPIO35 | West direction red lamp |
| West GREEN relay IN | GPIO36 | West direction green lamp |
| CD4511 BCD A–D (tens) | GPIO13–16 | Tens digit BCD input |
| CD4511 BCD A–D (units) | GPIO17–20 | Units digit BCD input |
| Buzzer | GPIO21 | Preemption audio alert |

### LM2596 Buck Converter → System Power

| LM2596 Pin | Connection | Description |
|---|---|---|
| IN+ | 12V vehicle supply (fused 3A) | Input from vehicle electrical system |
| IN− | GND | Input ground |
| OUT+ | 5V rail → ESP32, ESP8266, relays, GPS, 7-segment | Regulated 5V output |
| OUT− | GND | Output ground |

---

## How It Works

1. **Activation** — Ambulance driver presses the dashboard button; single press cycles hospital options, long press (>2s) confirms selection and activates emergency mode.
2. **LoRa Beacon** — ESP8266 broadcasts 18-byte emergency packets at 433 MHz every 500ms containing ambulance ID, sequence number, direction flags, and vitals snapshot.
3. **Junction Preemption** — When the ambulance enters ~300–500m of a junction, the ESP32 receiver validates the packet (sync word + rolling sequence check), suspends the normal signal cycle, switches the ambulance-approach lane to GREEN, holds all crossing lanes at RED.
4. **Countdown Display** — 7-segment counter counts down from the hold period (30s); timer resets on each received packet; reverts to normal cycle after ambulance clears.
5. **Vitals Acquisition** — MAX30102 collects 100-sample PPG buffers; Beer-Lambert law derivation computes SpO2%; inter-beat interval computes heart rate in BPM.
6. **Hospital Notification** — GSM SIM800L sends structured SMS to hospital: `PATIENT: <name> | HR: <bpm> | SPO2: <pct>% | CONDITION: <status>` — transmitted on emergency activation and on critical threshold breach.
7. **Google Assistant** — Driver activates voice assistant to query nearest hospital; Places API returns ranked list; driver selects verbally; hospital coordinates sent to ESP8266 for ETA computation.
8. **Accident Detection** — MPU6050 streams accelerometer at 100Hz; RMS g-force sliding window triggers crash alert when >3.5g sustained over 3 samples; GSM transmits alert with last GPS coordinates.

### Detection Logic (simplified)

```c
// Crash detection — imu_monitor
float gForce = calculateGForce(ax, ay, az);   // sqrt(ax²+ay²+az²) / 16384.0
if (gForce > CRASH_THRESHOLD_G) {
    crashCount++;
    if (crashCount >= CRASH_CONFIRM_SAMPLES) {
        g_crashDetected = true;
        buildLoRaCrashPacket(buf, len);
        sendLoRaPacket(buf, len);
        gsmSendSMS(HOSPITAL_SMS_NUMBER, buildEmergencyPayload());
    }
} else {
    crashCount = 0;
}

// Junction preemption — junction firmware
void onLoRaReceive(int packetSize) {
    LoRa.readBytes(pktBuf, packetSize);
    if (validateSyncWord(pktBuf) && validateSequence(pktBuf)) {
        ambulanceApproach = true;
        setGreen(detectedDirection);
        holdCountdown = PREEMPTION_HOLD_SEC;
    }
}
```

### LoRa Packet Structure

| Byte | Field | Description |
|---|---|---|
| 0 | Packet Type | 0x01=Vitals, 0x02=Crash, 0x03=SOS, 0x04=Heartbeat |
| 1–2 | Sequence Number | uint16, incremented each TX, persisted to EEPROM |
| 3–4 | Heart Rate | int16 (BPM × 1) |
| 5–6 | SpO2 | int16 (% × 1) |
| 7–8 | Accel X | int16 raw |
| 9–10 | Accel Y | int16 raw |
| 11–12 | Accel Z | int16 raw |
| 13 | Flags | Bit 0=Emergency active, Bit 1=Crash, Bit 2=GSM OK |
| 14 | Checksum | XOR of bytes 0–13 |

### GSM SMS Payload (Hospital Side)

```
PATIENT - VISHNU
HEART RATE - 80 BPM
SPO2 - 98%
CONDITION - NORMAL
```

---

## Communication Protocols

### LoRa (SX1278 — Ambulance ↔ Junction)
- **Frequency:** 433 MHz ISM band
- **Spreading Factor:** SF9 | **Bandwidth:** 125 kHz | **Coding Rate:** 4/5
- **TX Power:** 17 dBm | **Sync Word:** 0xA5 (custom, filters foreign traffic)
- **CRC:** Enabled | **Packet interval:** 500ms (emergency), 5s (standby heartbeat)
- **Range:** 300–500m urban (confirmed); up to 2–5 km line-of-sight
- **Security:** Rolling sequence number rejects replay attacks

**LoRa API used:**
```c
LoRa.begin(433E6);
LoRa.setSpreadingFactor(9);
LoRa.setSignalBandwidth(125E3);
LoRa.setCodingRate4(5);
LoRa.setTxPower(17);
LoRa.setSyncWord(0xA5);
LoRa.enableCrc();
// TX
LoRa.beginPacket(); LoRa.write(buf, len); LoRa.endPacket();
// RX
LoRa.onReceive(onLoRaReceive); LoRa.receive();
```

### GSM / GPRS (SIM800L — Ambulance → Hospital)
- **Interface:** UART @ 9600 baud, 8N1
- **Services used:** SMS (hospital alert), GPRS HTTP POST (backend server)
- **Library:** TinyGSM + ArduinoHttpClient
- **Retry logic:** 5 retries with 15s backoff on 5xx; auto hardware-reset after persistent failure

**Example AT commands (SMS path):**
```
ATE0                          // Disable echo
AT+CPIN?                      // Verify SIM ready
AT+CMGF=1                     // Set SMS text mode
AT+CMGS="+91XXXXXXXXXX"       // Set recipient
> PATIENT - VISHNU | HR: 80 BPM | SPO2: 98% | CONDITION: NORMAL
```

### I2C (MPU6050 + MAX30102 — Shared Bus)
- **Pins:** SDA = GPIO4 (D2), SCL = GPIO14 (D5) on ESP8266
- **Speed:** 400 kHz Fast Mode
- **MPU6050 address:** 0x68 | **MAX30102 address:** 0x57
- **Pull-ups:** 4.7 kΩ to 3.3V
- **DLPF:** 42 Hz low-pass on MPU6050 to suppress vehicle vibration noise

### SPI (LoRa SX1278)
- **Interface:** HSPI @ 8 MHz | Mode 0 (CPOL=0, CPHA=0) | MSB first
- **ESP8266 pins:** MISO=D6, MOSI=D7, SCK=D5, CS=D8, RST=D0, DIO0=D1

### HAL Functions Used

```c
// I2C
Wire.begin(I2C_SDA, I2C_SCL);
imu.initialize(); imu.getAcceleration(&ax, &ay, &az);
spo2Sensor.begin(Wire, I2C_SPEED_FAST);

// UART (GSM)
gsmSerial.begin(GSM_BAUD);
modem.gprsConnect(apn, user, pass);
modem.sendSMS(number, message);

// GPIO (traffic lights)
pinMode(N_GRN, OUTPUT); digitalWrite(N_GRN, HIGH);

// Watchdog
ESP.wdtFeed();  // ESP8266 software watchdog feed
```

---

## Results

The system was successfully validated on a physical hardware prototype simulating a four-way intersection. Key outcomes:

| Parameter | Result |
|---|---|
| LoRa detection range (urban) | 300–500 m confirmed |
| Packet delivery rate (urban) | > 95% at 433 MHz / SF9 |
| Junction preemption latency | < 500 ms from packet reception |
| Traffic clearance trigger time | Immediate on first valid packet |
| GSM SMS delivery to hospital | Confirmed with patient vitals (HR 80 BPM, SpO2 98%) |
| Google Assistant hospital search | Operational — nearby hospitals listed and selected by voice |
| Crash detection (MPU6050) | Triggered correctly on simulated 3.5g+ impact events |
| False positive rate | Zero — DLPF + sliding window suppressed road vibrations |
| SpO2 measurement accuracy | Valid readings across 3-round averaging (80–100% range) |
| System power draw | Stable at 5V regulated from 12V input via LM2596 |

- The ESP32 dual-core FreeRTOS allocation kept LoRa beacon timing unaffected by GSM transmission delays.
- The 7-segment countdown display correctly showed hold period and blanked after timer expiry.
- The hospital-side SMS received complete patient data within 2–4 seconds of ambulance activation.
- The rolling sequence number authentication rejected all replayed test packets at the junction.

---

## Future Scope

- **LoRaWAN green corridor** — Connect junction ESP32 units to a LoRaWAN gateway for city-wide predictive signal preemption along the full planned route simultaneously
- **Real-time route optimization** — Integrate Google Maps Directions API via GPRS for dynamic re-routing based on live traffic conditions with I2S turn-by-turn voice guidance
- **12-lead ECG module** — Add pre-hospital ECG transmission to activate cardiac catheterization team before patient arrival, targeting 20–30 minute reduction in door-to-balloon time
- **ML-based patient triage** — Deploy TensorFlow Lite vital-sign classification model on ESP32 to auto-assign triage categories (immediate / urgent / delayed) for hospital resource pre-allocation
- **Supercapacitor power buffer** — Add bulk capacitance on the 5V rail to handle SIM800L peak current transients and protect against engine-cranking voltage dips
- **Solar + Li-ion backup for junctions** — Maintain preemption capability during mains power outages at traffic cabinets
- **Multi-vehicle support** — Extend LoRa packet structure and junction arbitration logic to handle simultaneous preemption requests from fire engines and police vehicles
- **Smart city integration** — Expose junction LoRa backbone as a shared LoRaWAN layer for concurrent smart city applications (parking, air quality, street lighting)

---

> *Built to reduce ambulance transit times and improve emergency patient outcomes through intelligent embedded systems and autonomous IoT infrastructure.*
