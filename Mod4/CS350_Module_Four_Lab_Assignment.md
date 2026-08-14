# CS 350 Module Four Lab Assignment: LCD Display Integration & Reflection

**Course**: CS 350 - Emerging Systems Architecture  
**Module**: Four Lab Assignment  

---

## Part 1: Lab Questions

### Question 1: Why do you have a sleep command in your loop?

**Answer:**  
The `sleep(1)` command pauses loop execution for one second between iterations to prevent the Python `while` loop from consuming 100% of CPU capacity (busy-waiting), which saves power and reduces thermal output on the Raspberry Pi. Additionally, pacing the loop at one-second intervals matches the hardware refresh kinetics of the 16x2 LCD display and synchronizes updates with the system time format (`%H:%M:%S`), preventing screen flickering, visual artifacts, and GPIO bus saturation.

---

### Question 2: What is the purpose of having a text display on an embedded device?

**Answer:**  
A text display serves as a low-cost, resource-efficient Human-Machine Interface (HMI) and diagnostic tool for embedded devices. It allows "headless" systems—which lack traditional monitors or graphical interfaces—to communicate real-time operational status, system metrics (such as IP address, temperature, or timestamp), and error codes directly to users and technicians without requiring an active network connection or SSH terminal.

---

### Question 3: How can you think of the display device as something that could relate to a state machine?

**Answer:**  
The display device functions as a visual output observer for an embedded system's Finite State Machine (FSM), where each internal state (e.g., initialization, active monitoring, alert, or error) maps directly to a specific message or layout rendered on the LCD screen during state transitions. Furthermore, the LCD's underlying HD44780 controller operates as an internal hardware state machine, moving sequentially through power-on initialization, command configuration, and data-entry modes as character bytes are received.

---

## Part 2: Reflection on Circuit Setup & Testing

### Installation Process Overview
The Module Four lab involved expanding our solderless breadboard circuit from a simple single-LED setup (Module 1) to incorporate a **16x2 Character LCD Display** and a **10kΩ Potentiometer** for contrast adjustment.

The assembly process followed a systematic approach:
1. **Package Management & Environment Setup**:  
   Installed `python3-pip` and configured global break-system-packages options (`sudo python3 -m pip config set global.break-system-packages true`) to install the required Adafruit libraries (`adafruit-blinka` and `adafruit-circuitpython-charlcd`).

2. **Physical Component Placement**:  
   Positioned the 16x2 LCD display starting at Pin 1 in column C-44 of the breadboard. The 10k potentiometer was inserted into column G across rows 45–47 to provide an adjustable voltage divider for LCD contrast tuning ($V_0$).

3. **Wiring Connections**:
   - **Ground Plane Bus**: Connected LCD Pin 1 ($V_{SS}$), LCD Pin 5 ($R/\bar{W}$), LCD Pin 16 ($K$ / LED backlight ground), and Potentiometer Pin 1 to the Pi ground rail.
   - **+5V Power Bus**: Connected LCD Pin 2 ($V_{DD}$), LCD Pin 15 ($A$ / LED backlight power), and Potentiometer Pin 3 to the +5V power rail.
   - **Contrast Control Signal**: Wired Potentiometer Pin 2 (wiper signal) directly to LCD Pin 3 ($V_0$).
   - **GPIO Signal Lines**: Wired control and 4-bit data lines to the Raspberry Pi GPIO headers:
     - LCD Pin 4 ($RS$) $\rightarrow$ GPIO 17
     - LCD Pin 6 ($EN$) $\rightarrow$ GPIO 27
     - LCD Pin 11 ($D4$) $\rightarrow$ GPIO 5
     - LCD Pin 12 ($D5$) $\rightarrow$ GPIO 6
     - LCD Pin 13 ($D6$) $\rightarrow$ GPIO 13
     - LCD Pin 14 ($D7$) $\rightarrow$ GPIO 26

4. **Testing and Software Execution**:  
   Executed `DisplayTest.py` to drive character outputs to the screen, verifying line 1 timestamp generation (`datetime.now()`) and line 2 static message formatting.

---

### Challenges Encountered and Resolutions

1. **Voltage Level Compatibility (5V vs. 3.3V Safety)**:  
   *Challenge*: The LCD logic and backlight operate on +5V, whereas the Raspberry Pi GPIO pins operate on +3.3V logic. Connecting a 5V output signal directly to a Raspberry Pi input GPIO pin would risk permanently damaging the Pi SoC.  
   *Resolution*: The circuit design safely mitigates this risk by tying LCD Pin 5 ($R/\bar{W}$ - Read/Write selection) directly to Ground ($0\text{V}$). This forces the LCD to operate permanently in **Write-Only mode**, ensuring the LCD never sends 5V signals back to the Raspberry Pi pins. The Pi only drives 3.3V logic signals *to* the LCD inputs, which the LCD correctly recognizes as High logic.

2. **Display Contrast & Solid Character Blocks**:  
   *Challenge*: Upon initial power-up, the LCD screen displayed either a blank blue screen or 16 solid white rectangle blocks across the top row, making text unreadable.  
   *Resolution*: This behavior occurs when the contrast voltage at Pin 3 ($V_0$) is not tuned. By rotating the potentiometer dial, the voltage level on LCD Pin 3 was adjusted between 0V and 5V until the liquid crystal background cleared and crisp, legible characters appeared.

3. **Breadboard Pin Density & Wiring Verification**:  
   *Challenge*: Routing 14 jumper cables in close proximity around the 16-pin LCD header creates visual clutter, making it easy to misalign pins (e.g., confusing $D4$ on GPIO 5 with $D5$ on GPIO 6).  
   *Resolution*: Verified each connection against the pinout table prior to powering on the Raspberry Pi, ensuring color-coded wires were used for Ground (black/brown), Power (red/orange), Control (green), and Data lines (blue/purple).

---

## Part 3: Code Reference (`DisplayTest.py`)

Below is the verified test script configuration used to validate the LCD hardware:

```python
from datetime import datetime
from time import sleep

import board
import digitalio
import adafruit_character_lcd.character_lcd as characterlcd

def cleanupDisplay(a, b, c, d, e, f):
    lcd.clear()
    a.deinit()
    b.deinit()
    c.deinit()
    d.deinit()
    e.deinit()
    f.deinit()

# Define LCD display dimensions
lcd_columns = 16
lcd_rows = 2

# Configure GPIO line mappings matching physical circuit
lcd_rs = digitalio.DigitalInOut(board.D17)
lcd_en = digitalio.DigitalInOut(board.D27)
lcd_d4 = digitalio.DigitalInOut(board.D5)
lcd_d5 = digitalio.DigitalInOut(board.D6)
lcd_d6 = digitalio.DigitalInOut(board.D13)
lcd_d7 = digitalio.DigitalInOut(board.D26)

# Initialize Character LCD class in 4-bit mode
lcd = characterlcd.Character_LCD_Mono(
    lcd_rs, lcd_en, lcd_d4, lcd_d5, lcd_d6, lcd_d7, lcd_columns, lcd_rows
)

# Clear LCD screen before main loop
lcd.clear()

repeat = True
while repeat:
    try:
        # Line 1: Dynamic Date & Time (Month Day  HH:MM:SS)
        lcd_line_1 = datetime.now().strftime('%b %d  %H:%M:%S\n')
        
        # Line 2: Status Message
        lcd_line_2 = 'Display Day!'

        # Write combined message to LCD
        lcd.message = lcd_line_1 + lcd_line_2

        # Pacing sleep interval
        sleep(1)
    except KeyboardInterrupt:
        # Cleanup GPIO pins on exit
        cleanupDisplay(lcd_rs, lcd_en, lcd_d4, lcd_d5, lcd_d6, lcd_d7)
        repeat = False
```
