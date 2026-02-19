# Schematic V2 - Integrated LM2596S-5.0 Power Section

## Design Overview

This version integrates the LM2596S-5.0 buck converter circuit directly onto the main PCB, eliminating the need for a separate voltage regulator module. This design is more cost-effective and space-efficient while maintaining the same performance specifications.

---

## Power Supply Section (Integrated)

### Input Stage
```
12V DC Input (from SMPS)
    ↓
[Terminal Block J1] (2-pin, 5.08mm)
    ↓
[C1: 220µF/25V Electrolytic] ← Input filtering
    ↓
[LM2596S-5.0 IC (U1)]
    ├─ VIN  (Pin 1)
    ├─ OUT  (Pin 2)
    ├─ GND  (Pin 3)
    └─ FB   (Pin 4) - Fixed 5V version
```

### Switching Stage
```
U1 (LM2596S-5.0) Pin 2 (OUT)
    ↓
[L1: 33µH Inductor, 3A rated]
    ↓
    ├─→ [D1: SS34 Schottky Diode] → GND (freewheeling)
    ↓
[C2: 100µF/50V Electrolytic] ← Output filtering
[C3: 100µF/50V Electrolytic] ← Additional filtering
[C4: 10µF/16V Ceramic 0805]  ← High-frequency filtering
    ↓
5V Output to ESP32 and Peripherals
```

---

## Complete System Block Diagram

```
┌─────────────────────────────────────────────────────────────┐
│               220V AC Mains Input                           │
└────────────────────────┬────────────────────────────────────┘
                         ↓
                  [12V 3A SMPS]
                         ↓
         ┌───────────────┴───────────────┐
         │                               │
         ↓ 12V                           ↓ 12V
  ┌─────────────────┐           ┌───────────────┐
  │ LM2596S Circuit │           │ Coin Acceptor │
  │   (Integrated)  │           │   (12V Type)  │
  │                 │           └───────┬───────┘
  │ C1 (220µF) ─┐   │                   │
  │             ├─→ U1 (LM2596S)        │ Pulse Output
  │ L1 (33µH) ──┤   │                   ↓
  │ D1 (SS34) ──┤   │          ┌────────────────┐
  │ C2,C3,C4 ───┘   │          │   ESP32-WROOM  │
  └────────┬────────┘          │   GPIO27 ← Pulse
           │ 5V                │   GPIO23 → Relay
           ↓                   │   GPIO21 ↔ SDA
  ┌─────────────────────────┐ │   GPIO22 → SCL
  │       5V Rail           │ │   GPIO2  → LED
  │  ┌──────────────────┐   │ │                │
  │  │  ESP32 Module    │←──┘ └────┬───────┬───┘
  │  │  Relay Circuit   │          │       │
  │  │  LED Indicator   │          ↓       ↓
  │  └──────────────────┘    ┌─────────┐ ┌──────────┐
  └─────────────────────────→│LCD I2C  │ │  Relay   │
                              │Display  │ │  Module  │
                              └─────────┘ └──────────┘
```

---

## Critical PCB Layout Guidelines

### 1. Power Section Placement
- **Place U1 (LM2596S) near the input terminals**
- **Keep C1 (input cap) as close as possible to U1 VIN pin** (< 5mm)
- **L1 should be directly after U1 output pin** (< 3mm trace)
- **D1 cathode connects to L1/U1 junction, anode to GND**

### 2. Trace Width Requirements
| Connection | Min Width | Recommended | Max Current |
|------------|-----------|-------------|-------------|
| 12V Input  | 0.5mm     | 1.0mm       | 3A          |
| 5V Output  | 0.5mm     | 1.0mm       | 3A          |
| GND        | 1.0mm     | 2.0mm       | 3A+         |
| Switching Node (U1-L1-D1) | 0.3mm | 0.5mm | Minimize area |

### 3. Ground Plane Strategy
```
                Top Layer View
    ┌─────────────────────────────────┐
    │  ┌─[U1]─[L1]─[D1]─────┐         │
    │  │                     │         │
    │ [C1]        [C2][C3][C4]        │
    │  │              │               │
    │  └──────[GND]───┘               │
    │         ↓↓↓↓↓                   │
    │    [Solid Ground Plane]         │
    └─────────────────────────────────┘
```

- **Use a solid ground plane on bottom layer**
- **Connect all GND points with vias to bottom plane**
- **Keep switching node (between U1, L1, D1) small and tight**

### 4. Thermal Management
```
        Top View of U1 (TO-263)
    ┌─────────────────────────┐
    │    Tab (connected to    │
    │     internal GND)       │
    └───┬─────────┬─────────┬─┘
        │         │         │
       VIN       OUT       GND
```

- **Add thermal vias under U1 tab** (minimum 4x 0.8mm vias)
- **Connect tab to bottom ground plane**
- **Copper pour around U1 for heat dissipation** (25mm²)
- **Consider adding a small heatsink for continuous 3A operation**

### 5. Component Orientation
```
    J1 (12V IN)
        ↓
      [C1]
        ↓
      [U1] →→→ [L1] →→→ [Output]
        ↑              ↓
        └──── [D1] ←───┘
              ↓
            [GND]
```

---

## Bill of Materials (Power Section Only)

| Ref | Component | Value | Package | Qty | LCSC Part # | Notes |
|-----|-----------|-------|---------|-----|-------------|-------|
| U1  | LM2596S-5.0 | Fixed 5V | TO-263 | 1 | C9865 | Buck converter IC |
| L1  | Inductor | 33µH | SMD 1210 | 1 | C408412 | 3A saturation current |
| D1  | Schottky Diode | SS34 (3A/40V) | DO-214AB | 1 | C8678 | Low Vf |
| C1  | Electrolytic | 220µF/25V | 8x12mm | 1 | C59339 | Input filter |
| C2  | Electrolytic | 100µF/50V | 8x12mm | 1 | C134509 | Output filter |
| C3  | Electrolytic | 100µF/50V | 8x12mm | 1 | C134509 | Output filter |
| C4  | Ceramic | 10µF/16V | 0805 | 1 | C15850 | HF filter |
| J1  | Terminal Block | 2-pin 5.08mm | Screw | 1 | C8465 | 12V input |

**Total Cost (Power Section):** ~$1.15 vs. ~$2.00 for LM2596 module

---

## Design Specifications

### Electrical Characteristics
- **Input Voltage:** 12V DC (±10%)
- **Output Voltage:** 5.0V (±2%)
- **Output Current:** 3A maximum (continuous)
- **Efficiency:** ~85% @ 1A load
- **Ripple Voltage:** < 50mV peak-to-peak
- **Switching Frequency:** 150 kHz (internal to LM2596S)

### Protection Features
- **Over-current protection:** Built-in to LM2596S
- **Thermal shutdown:** 150°C junction temperature
- **Input reverse polarity:** Add optional diode if required

---

## Testing and Validation

### Power-Up Sequence
1. Apply 12V to J1 input terminals
2. Measure voltage at C1 (should be ~12V)
3. Measure voltage at C2/C3 output (should be 5.0V ±0.1V)
4. Check ripple voltage with oscilloscope (< 50mV)
5. Verify U1 temperature under load (< 80°C at 1A)

### Load Testing
| Load Current | Expected Vout | Max U1 Temp | Pass/Fail |
|--------------|---------------|-------------|-----------|
| 100mA        | 5.00V ±0.05V  | < 40°C      |           |
| 500mA        | 5.00V ±0.05V  | < 50°C      |           |
| 1A           | 4.95V ±0.05V  | < 65°C      |           |
| 2A           | 4.90V ±0.10V  | < 80°C      |           |

---

## Advantages of V2 Design

✅ **Cost Savings:** $0.85 per unit vs. using LM2596 module  
✅ **Space Efficiency:** ~40% smaller footprint  
✅ **Better Control:** Custom layout optimized for your board  
✅ **Improved Reliability:** Fewer connectors and wires  
✅ **Professional Appearance:** Single integrated PCB  

---

## Migration from V1 to V2

### Changes Required:
1. **PCB Layout:** Add power section to main board
2. **BOM:** Replace LM2596 module with discrete components
3. **Assembly:** SMD soldering required for U1, L1, D1
4. **Testing:** Add power section validation steps

### Backward Compatibility:
- ESP32 GPIO assignments unchanged
- 5V output specifications identical
- All peripheral connections remain the same

---

## Next Steps

1. **Create PCB layout** in KiCAD/EasyEDA
2. **Order components** from LCSC/JLCPCB
3. **Prototype and test** power section first
4. **Full system integration** once power validated
5. **Update main README.md** with V2 information

---

**Version:** 2.0  
**Date:** 2026-02-19 05:49:24  
**Author:** mcrey11  
**Status:** Design Ready for Prototyping