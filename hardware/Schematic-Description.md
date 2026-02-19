# JuanFi DC Control Architecture Schematic Description

## Overview
The JuanFi control architecture utilizes an isolated optocoupler and an IRLZ44N MOSFET for low-side switching of DC loads. This architecture replaces the traditional relay-based design, providing better isolation and faster switching times.

## Components
- **Optocoupler**: PC817
- **MOSFET**: IRLZ44N
- **Microcontroller**: ESP32
  - **GPIO P12**: Coin input
  - **GPIO P13**: Control signal for the PC817
- **Power Supply Sections**: Designed to handle the required load, optimizing performance and thermal efficiency.
- **I2C LCD Interface**: Enables easy communication between the ESP32 and the LCD.

## Circuit Description
1. **Isolated Switching**: The isolation provided by the PC817 allows for safe switching of higher voltages by the MOSFET from the low-voltage ESP32.
2. **Power Supply Design**: Ensures that the system operates within specified voltage and current limits, maintaining reliability.
3. **GPIO Connections**: 
   - GPIO P12 is used for detecting coin insertion, using an interrupt-driven approach to ensure fast response times.
   - GPIO P13 sends a control signal to the optocoupler, which in turn switches the MOSFET.
4. **I2C LCD Communication**: Allows for displaying status information of the control system, using only two wires for communication (SDA and SCL).

## Circuit Calculations
### Load Calculation
- Determine the load requirements based on the application's specifications. Calculate the current and voltage requirements to select appropriate MOSFETs and provide adequate power supply sizing.

### Power Budget Analysis
- Analyze the total power consumption of the system to ensure the power supply can handle the maximum load.
- Consider potential efficiency losses, especially in the switching components.

### Testing Points
- Incorporate test points within the circuit for easy access to measure voltages and currents during development and troubleshooting.

## Conclusion
The JuanFi DC control architecture provides an efficient, reliable, and safe method for DC motor control, leveraging modern components for enhanced functionality. 

This design promotes a more streamlined, safer approach to operating DC loads, improving overall system performance.
