# CS-350: Emerging Systems Architectures

This repository contains coursework, lab guides, assignments, and Python source code developed for **CS-350: Emerging Systems Architectures** at Southern New Hampshire University (SNHU).

---

## 📁 Repository Structure

```
CS-350/
├── Mod1/             # Module 1: Raspberry Pi Setup & GPIO Intro
├── Mod2/             # Module 2: Pulse Width Modulation (PWM) & Milestone 1
├── Mod3/             # Module 3: GPIO UART Communications & Milestone 2
├── Mod4/             # Module 4: 16x2 Character LCD Interfacing
├── Mod5/             # Module 5: Pushbutton GPIO Interrupts & State Machine
├── Mod6/             # Module 6: I2C Sensor Interfacing (AHT20)
├── Mod7/             # Module 7: Final Project Smart Thermostat (Report, Code & Diagram)
└── code/             # Complete categorized Python scripts and examples
    ├── Module-1/     # Basic GPIO LED control
    ├── Module-2/     # PWM LED fading scripts
    ├── Module-3/     # Serial UART client & server telemetry scripts
    ├── Module-4/     # 16x2 LCD display management
    ├── Module-5/     # Button interrupts & Morse code state machines
    ├── Module-6/     # I2C temperature/humidity sensor integration
    ├── Module-7/     # Full cyber-physical Smart Thermostat system
    └── Resources/    # Reusable state machine examples
```

---

## 🌡️ Final Project: Smart Thermostat

The Module 7 final project implements a production-style cyber-physical embedded thermostat on a Raspberry Pi 4B:

- **I2C Bus**: Reads ambient temperature from the Adafruit AHT20 sensor.
- **GPIO Interrupts**: 3-button controls (Mode cycle, Setpoint increment, Setpoint decrement).
- **PWM Output**: Visual status indicators (Red LED pulses for heating; Blue LED pulses for cooling; solid LED when setpoint reached).
- **16x2 Character LCD**: Displays real-time date/time on line 1 and alternates between current temperature and active state/setpoint on line 2.
- **UART Serial Telemetry**: Periodically broadcasts comma-delimited status updates (`state,temp,setpoint\n`) over the serial port.
