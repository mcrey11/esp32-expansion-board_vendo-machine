# ESP32 Expansion Board – Vending Machine Controller

> An ESP32-based expansion board designed for vending machine automation, featuring coin acceptance, LCD display, relay control, and multiple I/O capabilities.

![ESP32 Expansion Board Diagram](esp32_expansion_board.svg)

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Hardware Components](#hardware-components)
- [GPIO Mapping](#gpio-mapping)
- [System Architecture](#system-architecture)
- [Power Supply](#power-supply)
- [Getting Started](#getting-started)
- [Wiring Diagram](#wiring-diagram)
- [Troubleshooting](#troubleshooting)
- [Future Improvements](#future-improvements)
- [Contributing](#contributing)
- [License](#license)

## 🎯 Overview

This ESP32 expansion board is designed specifically for vending machine applications, providing a complete control solution for coin-operated dispensing systems. The board integrates power regulation, coin detection, LCD display, relay switching, and status indication in a single, compact design.

## ✨ Features

- **Coin Acceptance**: Universal coin slot interface (12V type) with pulse detection
- **LCD Display**: I2C-compatible LCD for user feedback and system status
- **Relay Control**: Output switching for dispensing mechanism
- **Status Indication**: Built-in LED for system status monitoring
- **Power Management**: Integrated buck converter for stable 5V supply
- **ESP32 WiFi/BLE**: Optional wireless connectivity for remote monitoring
- **Expandable I/O**: Multiple GPIO pins available for future enhancements

## 🔧 Hardware Components

| Component | Model/Type | Specifications |
|-----------|------------|----------------|
| **Microcontroller** | ESP32 Dev Module | Dual-core, WiFi, Bluetooth |
| **Primary Power Supply** | 12V SMPS | 3A output |
| **Buck Converter** | LM2596 | 12V → 5V, adjustable |
| **Coin Acceptor** | Universal Coin Slot | 12V type, pulse output |
| **Display** | LCD I2C | 16x2 or 20x4 (optional) |
| **Output Switch** | Relay Module | Currently relay (optocoupler upgrade planned) |
| **Status LED** | LED + 330Ω Resistor | Connected to GPIO2 |
| **I2C Pull-ups** | 4.7kΩ Resistors | For LCD communication |

## 📌 GPIO Mapping

| Function | GPIO Pin | Direction | Description |
|----------|----------|-----------|-------------|
| **Coin Input** | GPIO27 | Input | Pulse signal from coin acceptor |
| **Output Control** | GPIO23 | Output | Relay/optocoupler trigger for dispenser |
| **I2C SDA** | GPIO21 | Bidirectional | I2C data line for LCD |
| **I2C SCL** | GPIO22 | Output | I2C clock line for LCD |
| **System LED** | GPIO2 | Output | Status indicator (built-in LED) |

### Available for Future Use
- GPIO4, GPIO5, GPIO12, GPIO13, GPIO14, GPIO15, GPIO16, GPIO17, GPIO18, GPIO19, GPIO25, GPIO26, GPIO32, GPIO33

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────┐
│                  12V SMPS (3A)                  │
└────────────┬────────────────────────┬───────────┘
             │                        │
             │ 12V                    │ 12V
             ▼                        ▼
      ┌──────────────┐         ┌─────────────┐
      │   LM2596     │         │ Coin Slot   │
      │ Buck Conv.   │         │  (12V)      │
      │  12V → 5V    │         └──────┬──────┘
      └──────┬───────┘                │
             │ 5V                     │ Pulse
             ▼                        ▼
      ┌─────────────────────────────────┐
      │         ESP32 Module            │
      │  ┌──────────────────────────┐   │
      │  │ GPIO27 ← Coin Pulse      │   │
      │  │ GPIO23 → Relay Control   │   │
      │  │ GPIO21 ↔ I2C SDA        │   │
      │  │ GPIO22 → I2C SCL        │   │
      │  │ GPIO2  → Status LED     │   │
      │  └──────────────────────────┘   │
      └────┬─────────────────┬──────────┘
           │                 │
           ▼                 ▼
     ┌──────────┐      ┌──────────────┐
     │ LCD I2C  │      │ Relay Module │
     │ Display  │      │  → Dispenser │
     └──────────┘      └──────────────┘
```

## ⚡ Power Supply

### Primary Power Rail
- **Input**: 220V AC (mains)
- **SMPS Output**: 12V DC @ 3A
- **Distribution**: 
  - Coin acceptor: 12V direct
  - LM2596 input: 12V

### Secondary Power Rail
- **Buck Converter**: LM2596 (12V → 5V)
- **Output**: 5V regulated for ESP32
- **Current**: Up to 3A available (ESP32 typically uses 80-260mA)

### Power Consumption Estimates
| Component | Current Draw |
|-----------|--------------|
| ESP32 (active WiFi) | ~160-260mA |
| ESP32 (sleep mode) | ~5-10mA |
| LCD I2C Display | ~20-50mA |
| Coin Acceptor | ~100mA |
| Relay Module | ~70-80mA |
| **Total (max)** | ~500-600mA |

## 🚀 Getting Started

### Prerequisites
- ESP32 development board
- USB cable for programming
- Arduino IDE or PlatformIO
- Basic soldering skills (for connections)

### Software Requirements
- **Arduino IDE**: Version 1.8.13 or higher (OR)
- **PlatformIO**: Latest version
- **ESP32 Board Support**: Arduino-ESP32 core
- **Libraries**:
  - `LiquidCrystal_I2C` (for LCD display)
  - `Wire.h` (I2C communication, built-in)

### Hardware Assembly
1. Connect 12V SMPS to LM2596 buck converter input
2. Adjust LM2596 output to exactly 5V (use multimeter)
3. Connect ESP32 to 5V and GND from buck converter
4. Wire coin acceptor pulse output to GPIO27
5. Connect relay module signal pin to GPIO23
6. Wire I2C LCD (SDA→GPIO21, SCL→GPIO22)
7. Add 4.7kΩ pull-up resistors on I2C lines

### Software Setup

#### Using Arduino IDE:
```bash
# 1. Install ESP32 board support
# Go to File > Preferences
# Add to Additional Board Manager URLs:
https://dl.espressif.com/dl/package_esp32_index.json

# 2. Install boards: Tools > Board > Boards Manager
# Search "ESP32" and install

# 3. Install libraries: Tools > Manage Libraries
# Install: LiquidCrystal_I2C
```

#### Using PlatformIO:
```ini
[env:esp32dev]
platform = espressif32
board = esp32dev
framework = arduino
lib_deps = 
    marcoschwartz/LiquidCrystal_I2C@^1.1.4
```

### Upload Firmware
1. Connect ESP32 via USB
2. Select correct COM port
3. Upload sketch/firmware

### Basic Example Code

```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

// GPIO Pin Definitions
#define COIN_PIN 27
#define RELAY_PIN 23
#define LED_PIN 2

// I2C LCD (adjust address if needed: 0x27 or 0x3F)
LiquidCrystal_I2C lcd(0x27, 16, 2);

volatile int coinCount = 0;

void IRAM_ATTR coinInterrupt() {
  coinCount++;
}

void setup() {
  Serial.begin(115200);
  
  // Initialize pins
  pinMode(COIN_PIN, INPUT_PULLUP);
  pinMode(RELAY_PIN, OUTPUT);
  pinMode(LED_PIN, OUTPUT);
  
  digitalWrite(RELAY_PIN, LOW);
  digitalWrite(LED_PIN, HIGH);
  
  // Initialize LCD
  lcd.init();
  lcd.backlight();
  lcd.setCursor(0, 0);
  lcd.print("Vending Machine");
  lcd.setCursor(0, 1);
  lcd.print("Ready...");
  
  // Attach interrupt for coin detection
  attachInterrupt(digitalPinToInterrupt(COIN_PIN), coinInterrupt, FALLING);
}

void loop() {
  if (coinCount > 0) {
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Coin Detected!");
    lcd.setCursor(0, 1);
    lcd.print("Dispensing...");
    
    // Activate dispenser
    digitalWrite(RELAY_PIN, HIGH);
    delay(2000); // Dispense duration
    digitalWrite(RELAY_PIN, LOW);
    
    coinCount = 0;
    
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Thank You!");
    delay(2000);
    
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Insert Coin");
  }
  delay(100);
}
```

## 🔌 Wiring Diagram

See the included SVG schematic: [esp32_expansion_board.svg](esp32_expansion_board.svg)

### Quick Connection Reference
```
12V SMPS → LM2596 (IN+/IN-)
LM2596 (OUT+/OUT-) → ESP32 (5V/GND)
Coin Acceptor (12V/GND) → 12V SMPS
Coin Acceptor (PULSE) → ESP32 GPIO27
Relay Module (VCC/GND) → ESP32 (5V/GND)
Relay Module (IN) → ESP32 GPIO23
LCD (VCC/GND) → ESP32 (5V/GND)
LCD (SDA) → ESP32 GPIO21 (+ 4.7kΩ to 5V)
LCD (SCL) → ESP32 GPIO22 (+ 4.7kΩ to 5V)
```

## 🛠️ Troubleshooting

### LCD Not Displaying
- Check I2C address (common: 0x27, 0x3F)
- Verify pull-up resistors (4.7kΩ) on SDA/SCL
- Check 5V power supply to LCD
- Run I2C scanner sketch to detect address

### Coin Not Detected
- Verify 12V power to coin acceptor
- Check GPIO27 connection
- Ensure INPUT_PULLUP is enabled
- Test coin acceptor separately with multimeter

### Relay Not Switching
- Check 5V power to relay module
- Verify GPIO23 connection
- Test relay with direct 5V/GND (bypassing ESP32)
- Check relay module type (active HIGH vs LOW)

### ESP32 Won't Boot
- Check 5V supply voltage (should be 4.75-5.25V)
- Ensure LM2596 output is correctly adjusted
- Verify GND connections
- Try disconnecting all peripherals and boot ESP32 alone

## 🔮 Future Improvements

### Planned Upgrades
- [ ] Replace relay with optocoupler for better isolation
- [ ] Add RFID/NFC for cashless payment
- [ ] Implement WiFi connectivity for remote monitoring
- [ ] Add inventory tracking system
- [ ] Create web interface for configuration
- [ ] Integrate multiple coin denominations
- [ ] Add thermal printer for receipts
- [ ] Implement mobile app control

### Hardware Enhancements
- Custom PCB design
- Additional sensor inputs
- Temperature monitoring
- Security features (tamper detection)

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

### How to Contribute
1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 📄 License

This project is open source and available under the [MIT License](LICENSE).

## 📧 Contact

Project Maintainer: [@mcrey11](https://github.com/mcrey11)

Project Link: [https://github.com/mcrey11/esp32-expansion-board_vendo-machine](https://github.com/mcrey11/esp32-expansion-board_vendo-machine)

---

**⭐ If you find this project useful, please consider giving it a star!**