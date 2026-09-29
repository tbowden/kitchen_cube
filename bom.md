# Bill of Materials (BOM) - Voice-Controlled Kitchen Timer Cube

This document lists the required electronic components to build the rechargeable, voice-controlled kitchen bench timer using a classic Wi-Fi ESP32, a 128x128 monochrome OLED display, and a hardware-debounced mechanical interface.

## 🧠 Core Processing & Communication
*   **1x ESP32-WROOM-32D Development Board**
    *   *Specification:* Dual-core 32-bit Xtensa processor, integrated 2.4 GHz Wi-Fi and Bluetooth. 
    *   *Purpose:* Runs the main firmware, handles multiple local timer instances, drives the UI, and streams audio to Home Assistant.
    *   *Form Factor:* DIP-30 pinout development board preferred for easy prototyping.

## 📺 User Interface (Display & Input)
*   **1x 1.5-inch 128x128 Monochrome OLED Display Module (SSD1327 Driver)**
    *   *Specification:* Grayscale capable (4-bit), high contrast, square form factor.
    *   *Interface:* **4-wire SPI** interface (Highly recommended over I2C to handle the higher data refresh rate without screen lag).
*   **1x 3.3V CMOS 555 Timer IC**
    *   *Part Options:* **TLC555**, **LMC555**, or **ICM7555** (Do *not* buy standard NE555 chips, as they require 5V and will damage the ESP32).
    *   *Package:* DIP-8 for breadboards or SOP-8 for surface mount.
    *   *Purpose:* Implements the hardware monostable debouncer circuit for the primary mechanical switch.
*   **1x Mechanical Pushbutton / Tactile Switch**
    *   *Purpose:* Acts as the primary interaction button (e.g., Push-To-Talk for voice commands or waking the cube from deep sleep).

## 🔊 Audio & Voice Control
*   **1x INMP441 Digital I2S Microphone Module**
    *   *Specification:* Omnidirectional microphone with high-performance I2S digital output.
    *   *Purpose:* Captures crisp voice commands locally to stream to the Home Assistant Voice Pipeline over Wi-Fi.
*   **1x 5V/3.3V Active Buzzer Module or Magnetic Siren**
    *   *Specification:* Low current draw, audible tone emitter.
    *   *Purpose:* Triggers unique audio frequencies and alerts for when cooking timers expire.

## 🔋 Power Management & USB Recharging
*   **1x 3.7V Lithium-Polymer (LiPo) Battery Pouch or 18650 Cell**
    *   *Specification:* Recommended capacity of **1200mAh to 2000mAh** to safely sustain active Wi-Fi data transmissions.
*   **1x TP4056 Lithium Battery Charger Board with Protection**
    *   *Specification:* USB Type-C input interface. Must include the integrated battery protection circuit (`DW01` / `8205A` IC chips) with separate `B+/B-` and `OUT+/OUT-` solder terminals.
*   **1x AP2112K-3.3 Linear Voltage Regulator (LDO) or 3.3V Buck-Boost Module**
    *   *Specification:* Rated for at least **500mA to 1A continuous output**. 
    *   *Purpose:* Steps down fluctuating battery voltage (3.0V–4.2V) to a stable 3.3V while surviving sudden 250mA+ Wi-Fi radio power spikes.

## 🔌 Discrete Components (Resistors, Capacitors, Boards)
*   **Timing Components (For the 555 Debouncer ~50ms Pulse Window):**
    *   1x 43 kΩ Resistor (1/4 Watt)
    *   1x 1 µF Electrolytic or Tantalum Capacitor (Rated 10V or higher)
*   **Circuit Pull-ups & Control:**
    *   2x 10 kΩ Resistors (For 555 Trigger pin pull-up and general lines)
    *   1x 1 kΩ Resistor (In-line protection between 555 output and ESP32 GPIO)
    *   1x 0.01 µF (10nF) Ceramic Capacitor (For 555 Control Voltage Pin 5 stability)
*   **Power Rail Decoupling (Crucial for Wi-Fi stabilization):**
    *   1x 47 µF to 100 µF Electrolytic Capacitor (Placed directly across the ESP32 3.3V/GND rails)
    *   3x 0.1 µF (100nF) Ceramic Capacitors (Bypass filtering for the ESP32, 555, and OLED)
*   **Prototyping Base:**
    *   1x **Green Double-Sided FR-4 Glass Fiber Prototype PCB Board** (Multi-size variety pack recommended, e.g., 4x6cm or 5x7cm to fit inside a 3D-printed enclosure).
