
# SC3610 LCD Driver Repurposing

> Repurposing an undocumented SC3610 radio frequency counter IC as a generic 4-digit LCD display — driven by a single ESP32 GPIO.

## 📋 Project Overview

Following the HT1621 reverse engineering project, this project takes the reverse engineering further by reusing the **SC3610 frequency display controller** inside a **Thomson RCD-203U radio** as a dumb 4-digit LCD driver. Instead of driving every LCD segment directly, the goal was to make the existing frequency display circuitry show **arbitrary 4-digit numbers using only one GPIO** from an ESP32.

By keeping the original display circuitry, the project avoids 16 segment lines and 3 common lines — reducing control to **a single GPIO** while preserving the original look.

- Understand the SC3610's operating modes and which were actually implemented in the radio
- Reverse-engineer the hardware wiring without modifying the PCB
- Fake the frequency input signal to display arbitrary numbers
- Drive the LCD from an ESP32 using only one pin

## 🎨 Documentation

> 🎨 **Interactive Documentation:** [**🔴 View the Canva version 🔴**](https://canva.link/hcvckzcn1l8z89q)
> The Canva version is recommended because it includes the project videos and the full visual experience.

## 🛠️ What Was Built

| Component | Description |
|-----------|-------------|
| **SC3610 Driver** | Reused the original SC3610 IC as a generic 4-digit LCD controller |
| **Single-GPIO Control** | Full 4-digit display control from one ESP32 GPIO |
| **Signal Faking** | Generates a PWM square wave to spoof the AM frequency input |
| **Hardware Reverse Engineering** | Identified PCB test points, pinout, and IC configuration |

## 🔑 Key Technical Challenges & Solutions

### 1. Undocumented IC Behavior
The SC3610 is a multifunction radio display controller that normally counts an input frequency and displays it on its 4-digit 7-segment LCD. The datasheet describes AM/FM/clock modes, but the manufacturer only implemented a subset of features.

**Solution:** Studied the datasheet in detail and compared it against the actual radio hardware to identify which functions were wired up and which were not.

### 2. Locked Mode/Range Combination
The AM/FM selector and range selector were shorted together on the PCB, meaning only **AM-low** and **FM-high** were selectable.

**Solution:** Chose **AM mode**, which offers the higher display range (955–9999) and requires lower input frequencies the ESP32 can generate reliably.

### 3. Signal Input Requirements
The AMIN pin expects a frequency input, but the exact voltage levels were unclear.

**Solution:** Tested voltage dividers first (unreliable), then settled on a simple **300Ω series resistor** — with the pin's internal 2 kΩ impedance this gives ~2.87 V at the pin, well within the safe range and strong enough to trigger the internal comparator.

### 4. PWM Frequency Limitations
The ESP32's PWM peripheral was designed for motors and LEDs, not precise high-frequency square waves. Jitter increases at higher frequencies, and the SC3610 rounds internally — visible error appears around ~8500 kHz.

**Solution:** Used 1-bit PWM resolution to maximize the achievable frequency range, and characterized the error to keep the display reliable below 7000.

## 🚀 Getting Started

### Hardware
- ESP32 (any dev board)
- Thomson RCD-203U LCD module with SC3610 (or equivalent SC3610-based display)
- 300Ω series resistor
- Jumper wires

### Wiring
| ESP32 Pin | SC3610 / PCB |
|-----------|--------------|
| GPIO 25   | AMIN (via 300Ω) |
| 3.3V      | VDD / +3V    |
| GND       | GND          |
| —         | F.A tied HIGH (AM/FM-SEL + RANGE) |

### Software
1. Install the Arduino IDE or PlatformIO
2. Clone this repository
3. Open the demo sketch in `src/`
4. Upload to your ESP32

### Example Usage

```cpp
int display = 1234;

void setup() {
    int freq = (display + 455) * 1000;
    ledcAttach(25, freq, 1);
    ledcWrite(25, 1);
    uint32_t actual = ledcChangeFrequency(25, freq, 1);
}

void loop() {
    // Display stays at 1234
}
```

**How it works:** the SC3610 displays `frequency − 455 kHz`, so to show `N` you feed it `(N + 455) kHz`.

## 📚 Full Documentation

> **📄 Documentation:** The complete project documentation is provided as a PDF. For the best viewing experience, the **Canva version is recommended**, as it preserves the original layout, visuals, and interactive elements.

[Project Documentation (PDF)](docs/SC3610%20reverse%20engineering.pdf)

> The PDF contains detailed reverse engineering notes, pinout diagrams, circuit photos, and step-by-step methodology use canva for vids.

## 🎥 Project Demo

*(https://canva.link/iqmyz3xzq5s92ai)*

## 🧰 Technologies Used

- C++ / Embedded C++
- ESP32
- SC3610 / ET1621
- LEDC / PWM
- LCD Multiplexing (COM/SEG)
- Hardware Reverse Engineering
- Frequency Counting



## 📖 What I Learned

- How radio frequency counter ICs work internally
- Faking analog input signals with digital peripherals
- Single-GPIO display control
- Working outside datasheet conditions (the IC displayed values below its stated 500 kHz minimum, down to 0)
- Hardware debugging without manufacturer support

## 📬 Contact

**Ahmed Bouhlel** — movingbreadd@gmail.com

---

*Built as part of an engineering portfolio project.*

---


