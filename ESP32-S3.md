# ESP32-S3 options

From Core Electronics: [ESP32-S3-Nano Development Board](https://core-electronics.com.au/esp32-s3-nano-development-board-based-on-esp32-s3r8-compatible-with-arduino-nano-esp32.html)

# ESP32-S3 Voice Satellite & Timer Project Blueprint

This document outlines the hardware specifications, power management states, and software architecture for a low-power, "push-to-listen" smart home voice satellite featuring a countdown timer UI.

## 🛠️ Hardware Specification

### 1. Central Processing Unit
* **Development Board:** Core Electronics ESP32-S3-Nano
* **Processor:** Xtensa® 32-bit LX7 dual-core (configured up to 240 MHz)
* **Memory Config:** 16MB Flash (W25Q128JVSIQ) / 8MB On-Chip PSRAM (R8 variant)
* **Antenna:** Integrated 2.4G Ceramic Antenna

### 2. Audio & Display Peripherals
* **Microphone:** INMP441 or Adafruit (ICS-43434) Digital I2S MEMS Microphone Array
  * *Note: Analog microphones must be avoided due to electrical noise and ADC constraints.*
* **Display:** 128x128 I2C/SPI OLED Screen (SSD1327 or SSD1306 driver)

### 3. Power Infrastructure (V2 Strategy)
* **Battery:** 3.7V Lithium-Polymer (LiPo) Battery (~1200 mAh)
* **Power Regulation:** MP2322GQH 3.3V Regulator
* **Charging:** On-board USB Type-C charging circuit

---

## 🔋 Power Management & State Machine

To compromise between battery life constraints and hands-free voice operations, the firmware implements a **Push-to-Listen** multi-tier power state routine:

[ DEEP SLEEP ] ──(Physical Button Press)──► [ LIGHT SLEEP ] ──(Wake Word Matched)──► [ ACTIVE STREAMING ]
▲                                            │                                        │
└───────────(Timer Hits 0)───────────────────┴───────────(x Seconds Idle)─────────────┘

### State Breakdown

| State | ESP32-S3 Configuration | OLED Screen Status | Est. Power Draw | Expected Longevity (1200mAh) |
| :--- | :--- | :--- | :--- | :--- |
| **1. Deep Sleep** | CPU Powered Off (`esp_deep_sleep_start()`). Listening via external GPIO Interrupt. | Powered Down via software command (`0xAE`). | `~10–100 µA` | **1+ Years** (Standby) |
| **2. Light Sleep** | Dynamic Frequency Scaling active (`~10-20MHz`). Running `microWakeWord` / WakeNet. | **Active.** Displaying running countdown timer. | `~8–15 mA` | **70–100 Hours** (Cumulative screen-on time) |
| **3. Active Streaming** | Cores locked at **240 MHz**. Wi-Fi active. Streaming raw I2S audio frames over network. | **Active.** Showing listening/processing UI. | `~80–120 mA` | Spikes briefly during command processing |

---

## 🧠 Software & Network Architecture

### 1. Satellite Firmware (ESP32-S3)
* **Wake Word Processing:** Local wake-word validation using Espressif's **WakeNet / microWakeWord** models, taking advantage of the S3's AI vector instructions.
* **Audio Pipeline:** When the wake word is matched, raw microphone data is immediately packaged and pushed across the local network via network sockets (UDP/WebSockets).
* **Display Driver:** UI rendered using a lightweight framework (e.g., `U8g2` or ESPHome Display component). The countdown loop forces a millisecond-level CPU wake to refresh pixels before slipping back into low-power cycles.

### 2. Central Processing Hub (Raspberry Pi)
* **Network Receiver:** A dedicated listener instance (such as the Home Assistant **Wyoming Protocol** stack) continuously listens for incoming network audio bursts from the satellite room.
* **Speech to Text (STT):** Local, 100% offline decoding of the **Fixed Vocabulary** using resource-friendly models (e.g., *Faster-Whisper Tiny* or *Rhasspy* engine). 

---

## ⚠️ Core Engineering Caveats

1. **OLED Hard Shutoff:** You must explicitly send a hardware power-down instruction to the OLED driver board prior to initializing `esp_deep_sleep_start()`. Failing to do so will freeze the last drawn frame on-screen and fully discharge the battery within a few days.
2. **Ceramic Antenna Constraints:** The surface-mounted ceramic antenna on the Nano footprint has reduced penetration power compared to external dipole antennas. Keep the device within close proximity (ideally one room over with nominal obstructions) to the primary Wi-Fi access point to prevent dropped audio packets during streaming.
