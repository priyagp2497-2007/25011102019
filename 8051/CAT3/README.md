# EXPERIMENT 3

**EXP:** 3

**DATE:** 1/9/2026

**NAME OF EXP:** Human-Machine Interface using Matrix Keypad, LEDs, and Seven-Segment Display

**HARDWARE SETUP / PLATFORM:** AT89S52 (8051 Microcontroller)

**TASK NO:** TASK A

---

## 1. Circuit Diagram

### Hardware Connections

| Component | AT89S52 Connection |
|---|---|
| LED 1 | P1.0 (Pin 1) through 330Ω resistor |
| LED 2 | P1.1 (Pin 2) through 330Ω resistor |
| LED 3 | P1.2 (Pin 3) through 330Ω resistor |
| LED 4 | P1.3 (Pin 4) through 330Ω resistor |
| LED cathodes | GND |
| Seven-Segment Display segments (a–g, dp) | P0.0–P0.7 through current-limiting resistors |
| Seven-Segment common cathode | GND |
| Keypad Row 1 | P2.7 (Pin 28) |
| Keypad Row 2 | P2.6 (Pin 27) |
| Keypad Row 3 | P2.5 (Pin 26) |
| Keypad Row 4 | P2.4 (Pin 25) |
| Keypad Column 1 | P2.3 (Pin 24) |
| Keypad Column 2 | P2.2 (Pin 23) |
| Keypad Column 3 | P2.1 (Pin 22) |
| XTAL1 (Pin 19), XTAL2 (Pin 18) | 11.0592 MHz crystal |
| RST (Pin 9) | Reset circuit |
| VCC (Pin 40) | +5V |
| GND (Pin 20) | Ground |
| EA (Pin 31) | +5V |

**Clock Frequency:** 11.0592 MHz

**LED Configuration:** Active-HIGH

**Seven-Segment Display:** Common Cathode

**Keypad Configuration:** 4×3 Matrix Keypad

**Important Hardware Note:** Port 0 of the AT89S52 is open-drain and requires external pull-up resistors for reliable HIGH-level output. Use suitable current-limiting resistors for all seven-segment segments.

### Circuit Diagram

![Circuit Diagram](AT89S52%20CAT3%20Circuit%20Diagram.png)

---

## 2. Core Logic / Pseudocode / Code Base

### Working Principle

The AT89S52 microcontroller interfaces with a 4×3 matrix keypad, four LEDs, and a common-cathode seven-segment display.

The matrix keypad is used to provide user input. The microcontroller scans the keypad row by row and identifies which key has been pressed by checking the column pins.

The pressed key is displayed on the seven-segment display, and the corresponding LED operation is performed.

### Keypad Scanning

The keypad consists of four rows and three columns.

- Rows are connected to P2.7, P2.6, P2.5, and P2.4.
- Columns are connected to P2.3, P2.2, and P2.1.
- One row is made LOW at a time.
- The column pins are checked to identify the pressed key.
- A software delay is used for switch debouncing.
- The program waits until the pressed key is released.

### Seven-Segment Display Operation

The seven-segment display is connected to Port 0.

A lookup table named `seg_cod[]` contains the hexadecimal patterns required to display digits from 0 to 9.

The `display_digit()` function sends the corresponding pattern to the seven-segment display.

Since a common-cathode display is used, a HIGH signal turns ON the corresponding segment.

### LED Operations Based on Keypad Input

The program performs different operations depending on the pressed key.

| Key Pressed | Operation |
|---|---|
| 1 | Turn ON LED 1 |
| 2 | Turn ON LED 2 |
| 3 | Turn ON LED 3 |
| 4 | Turn ON LED 4 |
| 5 | Running LED pattern |
| 6 | Alternating LED pattern |
| 7 | 4-bit binary counter |
| 8 | Flash all four LEDs |
| 9 | Turn OFF all LEDs |
| 0 | Turn OFF all LEDs and display 0 |

### LED Operating Modes

**1. Running LED Pattern**

- LEDs turn ON one after another.
- The sequence moves from LED 1 to LED 4.
- The pattern repeats with a delay between each LED.

**2. Binary Counter**

- The four LEDs display binary values from 0000 to 1111.
- Each LED represents one bit of the binary number.
- The count increases after every delay.

**3. Alternating LED Pattern**

- LED 2 and LED 4 turn ON while LED 1 and LED 3 remain OFF.
- Then LED 1 and LED 3 turn ON while LED 2 and LED 4 remain OFF.
- Both patterns alternate continuously.

**4. Flash All LEDs**

- All four LEDs turn ON simultaneously.
- After a delay, all four LEDs turn OFF.
- This operation repeats.

### Pseudocode

1. Start the program.
2. Include the 8051 register definitions.
3. Define Port 1 for LED control and Port 0 for the seven-segment display.
4. Initialize the keypad row and column connections.
5. Initialize all LEDs and the seven-segment display to OFF.
6. Scan the keypad row by row.
7. Identify the pressed key by checking the column inputs.
8. Apply a software delay for debouncing.
9. Display the corresponding digit on the seven-segment display.
10. Execute the operation based on the pressed key:
    - Keys 1–4: Control individual LEDs.
    - Key 5: Running LED pattern.
    - Key 6: Alternating LED pattern.
    - Key 7: Binary counter.
    - Key 8: Flash all LEDs.
    - Key 9: Turn OFF all LEDs.
    - Key 0: Turn OFF all LEDs and display zero.
11. Repeat the keypad scanning and control process continuously.

### Code Base

The C program is written using Keil C51 for the AT89S52 microcontroller.

```c
#include <reg51.h>

#define LED P1
#define SEG P0

// Keypad Column Pin Definitions
sbit col1 = P2^3;
sbit col2 = P2^2;
sbit col3 = P2^1;

// Seven-segment common-cathode codes
unsigned char code seg_cod[10] = {
    0x3F, 0x06, 0x5B, 0x4F, 0x66,
    0x6D, 0x7D, 0x07, 0x7F, 0x6F
};

// Software delay
void delay(void)
{
    unsigned int i, j;

    for(i = 0; i < 200; i++)
        for(j = 0; j < 200; j++);
}

// Display a digit
void display_digit(unsigned char value)
{
    if(value <= 9)
        SEG = seg_cod[value];
}

// Scan the matrix keypad
unsigned char keypad_scan(void)
{
    // Row 1
    P2 = 0x7F;

    if(col1 == 0) {
        delay();
        display_digit(1);
        while(col1 == 0);
        return 1;
    }

    if(col2 == 0) {
        delay();
        display_digit(2);
        while(col2 == 0);
        return 2;
    }

    if(col3 == 0) {
        delay();
        display_digit(3);
        while(col3 == 0);
        return 3;
    }

    // Row 2
    P2 = 0xBF;

    if(col1 == 0) {
        delay();
        display_digit(4);
        while(col1 == 0);
        return 4;
    }

    if(col2 == 0) {
        delay();
        display_digit(5);
        while(col2 == 0);
        return 5;
    }

    if(col3 == 0) {
        delay();
        display_digit(6);
        while(col3 == 0);
        return 6;
    }

    // Row 3
    P2 = 0xDF;

    if(col1 == 0) {
        delay();
        display_digit(7);
        while(col1 == 0);
        return 7;
    }

    if(col2 == 0) {
        delay();
        display_digit(8);
        while(col2 == 0);
        return 8;
    }

    if(col3 == 0) {
        delay();
        display_digit(9);
        while(col3 == 0);
        return 9;
    }

    // Row 4
    P2 = 0xEF;

    if(col2 == 0) {
        delay();
        display_digit(0);
        while(col2 == 0);
        return 0;
    }

    return 0xFF;
}

// Running LED pattern
void running_led(void)
{
    unsigned char k;

    for(k = 0; k < 4; k++)
    {
        LED = (1 << k);
        delay();
    }
}

// Binary counter
void binary_counter(void)
{
    unsigned char k;

    for(k = 0; k < 16; k++)
    {
        LED = k;
        delay();
    }
}

// Alternating LED pattern
void alternate_led(void)
{
    LED = 0x0A;
    delay();

    LED = 0x05;
    delay();
}

// Flash all LEDs
void blink_all(void)
{
    LED = 0x0F;
    delay();

    LED = 0x00;
    delay();
}

void main(void)
{
    unsigned char key;

    LED = 0x00;
    SEG = 0x00;

    while(1)
    {
        key = keypad_scan();

        switch(key)
        {
            case 1: LED = 0x01; break;
            case 2: LED = 0x02; break;
            case 3: LED = 0x04; break;
            case 4: LED = 0x08; break;

            case 5: running_led(); break;
            case 6: alternate_led(); break;
            case 7: binary_counter(); break;
            case 8: blink_all(); break;

            case 9: LED = 0x00; break;

            case 0:
                LED = 0x00;
                display_digit(0);
                break;

            default: break;
        }
    }
}
```

### Code Explanation

- `#include <reg51.h>`: Includes the 8051 microcontroller register definitions.
- `#define LED P1`: Assigns Port 1 for controlling the four LEDs.
- `#define SEG P0`: Assigns Port 0 for controlling the seven-segment display.
- `sbit col1`, `col2`, `col3`: Define the three keypad column input pins.
- `seg_cod[]`: Stores the hexadecimal patterns for displaying digits 0–9.
- `delay()`: Generates a software delay.
- `display_digit()`: Displays the selected digit on the seven-segment display.
- `keypad_scan()`: Scans the keypad row by row and identifies the pressed key.
- `running_led()`: Produces a running LED sequence.
- `binary_counter()`: Displays binary values from 0000 to 1111.
- `alternate_led()`: Alternates between two LED combinations.
- `blink_all()`: Turns all four LEDs ON and OFF together.
- `switch(key)`: Selects the appropriate LED operation based on the pressed key.

---

## 3. Inference

The matrix keypad was interfaced successfully with the AT89S52 microcontroller to provide user input. The pressed keys were identified using row-by-row scanning, and the corresponding digits were displayed on the common-cathode seven-segment display.

Different LED operations, including individual LED control, running LED patterns, binary counting, alternating patterns, and flashing, were implemented based on keypad input.

This experiment demonstrates matrix keypad interfacing, seven-segment display control, GPIO programming, software debouncing, and implementation of a basic human-machine interface using the 8051 microcontroller.