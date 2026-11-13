# EXPERIMENT 2

**EXP:** 2

**DATE:** 31/7/2026

**NAME OF EXP:** Timer-Based LED Pattern Generator and Button Interrupt Controller

**HARDWARE SETUP / PLATFORM:** AT89S52 (8051 Microcontroller)

**TASK NO:** TASK A

---

## 1. Circuit Diagram

### Hardware Connections

| Component | Connection |
|---|---|
| AT89S52 P1.0 (Pin 1) | LED 1 control |
| AT89S52 P1.1 (Pin 2) | LED 2 control |
| AT89S52 P1.2 (Pin 3) | LED 3 control |
| AT89S52 P1.3 (Pin 4) | LED 4 control |
| LED anodes (+) | Connected to respective P1 pins through 330Ω resistors |
| LED cathodes (−) | Connected to GND |
| P2.7 (Pin 28) | Switch row control |
| P2.3 (Pin 24) | Switch column input |
| Push button | Connected between P2.3 and the LOW-driven switch row (P2.7) |
| XTAL1 (Pin 19), XTAL2 (Pin 18) | 11.0592 MHz crystal |
| RST (Pin 9) | Reset circuit |
| VCC (Pin 40) | +5V |
| GND (Pin 20) | Ground |
| EA (Pin 31) | +5V |

**Clock Frequency:** 11.0592 MHz

**LED Configuration:** Active-HIGH

**Switch Configuration:** Active-LOW

The LEDs turn ON when their corresponding P1 pins are HIGH (1) and turn OFF when the pins are LOW (0).

The push button is detected when P2.3 becomes LOW. P2.7 is kept LOW to provide the switch reference.

### Circuit Diagram

![Circuit Diagram](AT89S52%20CAT2%20Circuit%20Diagram.png)

---

## 2. Core Logic / Pseudocode / Code Base

### Working Principle

The AT89S52 microcontroller controls four LEDs connected to pins P1.0, P1.1, P1.2, and P1.3. A push button connected to Port 2 is used to change the LED operating mode.

The program consists of four different LED patterns:

**Mode 0 – Running LED Pattern (Chaser)**

- LEDs turn ON one after another.
- Only one LED remains ON at a time.
- The pattern moves from LED 1 to LED 4 and repeats.

**Mode 1 – 4-bit Binary Counter**

- The four LEDs display binary numbers from 0000 to 1111.
- Each LED represents one bit of the binary number.
- The pattern increments after every delay.

**Mode 2 – Alternating LED Pattern**

- LED 1 and LED 3 turn ON while LED 2 and LED 4 remain OFF.
- Then LED 2 and LED 4 turn ON while LED 1 and LED 3 remain OFF.
- The two patterns alternate continuously.

**Mode 3 – Flashing All LEDs**

- All four LEDs turn ON simultaneously.
- After a delay, all four LEDs turn OFF.
- This pattern repeats continuously.

### Push Button Operation

The push button is used to switch between the four LED modes.

- Initially, the program starts in Mode 0.
- When the push button is pressed, the program detects the LOW signal at P2.3.
- The current LED pattern is interrupted when the button press is detected during the delay.
- The mode variable is incremented to select the next pattern.
- After Mode 3, the program returns to Mode 0.
- The program waits until the button is released before accepting another press.

### Delay and Button Detection

The `delay_and_check()` function generates a software delay using nested `for` loops.

During the delay, the function continuously checks whether the push button is pressed.

- If P2.3 becomes LOW, the function returns 1 immediately.
- If the delay completes without a button press, the function returns 0.

This allows the program to respond to button presses without waiting for the entire LED pattern delay to finish.

### Pseudocode

1. Start the program.
2. Configure P2.7 as LOW for switch row control.
3. Set P2.3 HIGH to prepare it for switch input.
4. Initialize the lower four bits of Port 1 to LOW.
5. Set the initial mode to 0.
6. Execute the LED pattern based on the current mode:
   - Mode 0: Run the running LED pattern.
   - Mode 1: Display the 4-bit binary counter.
   - Mode 2: Display the alternating LED pattern.
   - Mode 3: Flash all four LEDs.
7. During each pattern delay, continuously check the push button.
8. If the button is pressed, interrupt the current pattern.
9. Check the button again after a debounce delay.
10. Increment the mode variable.
11. If the mode exceeds 3, reset it to 0.
12. Wait until the push button is released.
13. Repeat the process continuously.

### Code Base

The C program is written using Keil C51 for the AT89S52 microcontroller.

```c
#include <reg51.h>

// LED Pin Definitions (P1.0 to P1.3)
sbit ledpin1 = P1^0;
sbit ledpin2 = P1^1;
sbit ledpin3 = P1^2;
sbit ledpin4 = P1^3;

// Switch Pin Definitions
sbit swrow  = P2^7;
sbit swicol = P2^3;

unsigned char mode = 0;

// Delay function with button detection
unsigned char delay_and_check()
{
    unsigned int i, j;

    for (i = 0; i < 300; i++)
    {
        for (j = 0; j < 300; j++)
        {
            if (swicol == 0)
            {
                return 1;
            }
        }
    }

    return 0;
}

// Mode 0: Running LED Pattern
void runningled()
{
    ledpin1 = 1; ledpin2 = 0; ledpin3 = 0; ledpin4 = 0;
    if (delay_and_check()) return;

    ledpin1 = 0; ledpin2 = 1; ledpin3 = 0; ledpin4 = 0;
    if (delay_and_check()) return;

    ledpin1 = 0; ledpin2 = 0; ledpin3 = 1; ledpin4 = 0;
    if (delay_and_check()) return;

    ledpin1 = 0; ledpin2 = 0; ledpin3 = 0; ledpin4 = 1;
    if (delay_and_check()) return;
}

// Mode 1: 4-bit Binary Counter
void binarycounter()
{
    unsigned char count;

    for (count = 0; count < 16; count++)
    {
        P1 = (P1 & 0xF0) | (count & 0x0F);

        if (delay_and_check()) return;
    }
}

// Mode 2: Alternating Pattern
void alternatingpattern()
{
    ledpin1 = 1; ledpin2 = 0; ledpin3 = 1; ledpin4 = 0;
    if (delay_and_check()) return;

    ledpin1 = 0; ledpin2 = 1; ledpin3 = 0; ledpin4 = 1;
    if (delay_and_check()) return;
}

// Mode 3: Flashing All LEDs
void flash()
{
    ledpin1 = 1; ledpin2 = 1; ledpin3 = 1; ledpin4 = 1;
    if (delay_and_check()) return;

    ledpin1 = 0; ledpin2 = 0; ledpin3 = 0; ledpin4 = 0;
    if (delay_and_check()) return;
}

void main()
{
    swrow = 0;
    swicol = 1;

    P1 = P1 & 0xF0;

    while (1)
    {
        switch (mode)
        {
            case 0: runningled(); break;
            case 1: binarycounter(); break;
            case 2: alternatingpattern(); break;
            case 3: flash(); break;
        }

        // Button handler
        if (swicol == 0)
        {
            delay_and_check();

            if (swicol == 0)
            {
                mode++;

                if (mode > 3)
                {
                    mode = 0;
                }

                while (swicol == 0);
            }
        }
    }
}
```

### Code Explanation

- `#include <reg51.h>`: Includes the 8051 microcontroller register definitions.
- `sbit`: Assigns individual microcontroller pins to meaningful variable names.
- `mode`: Stores the currently selected LED operating mode.
- `delay_and_check()`: Generates a software delay and checks for a button press during the delay.
- `runningled()`: Produces the running LED pattern.
- `binarycounter()`: Displays binary values from 0000 to 1111 using four LEDs.
- `alternatingpattern()`: Alternates between two LED combinations.
- `flash()`: Turns all four LEDs ON and OFF together.
- `switch(mode)`: Selects and executes the LED pattern according to the current mode.
- `mode++`: Moves to the next LED pattern after a valid button press.
- `while (swicol == 0);`: Waits until the button is released before accepting another press.

---

## 3. Inference

The four LED patterns were controlled successfully using GPIO programming on the AT89S52 microcontroller. The push button allowed switching between the running LED pattern, 4-bit binary counter, alternating pattern, and flashing pattern.

The experiment demonstrates digital output control, software delay generation, button interfacing, polling-based input detection, and implementation of multiple operating modes using Embedded C.