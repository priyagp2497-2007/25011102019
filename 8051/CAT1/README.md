# EXPERIMENT 1

**EXP:** 1

**DATE:** 31/08/2026

**NAME OF EXP:** Bare-Metal LED Bring-Up and LED Blink

**HARDWARE SETUP / PLATFORM:** AT89S52 (8051 Microcontroller)

**TASK NO:** TASK A

---

## 1. Circuit Diagram

### Hardware Connections

| Component | Connection |
|---|---|
| AT89S52 P1.0 (Pin 1) | Connected to LED through a 330Ω resistor |
| LED anode (+) | Connected to the resistor |
| LED cathode (−) | Connected to GND |
| XTAL1 (Pin 19) and XTAL2 (Pin 18) | Connected to an 11.0592 MHz crystal |
| Crystal capacitors | Connected between each crystal terminal and GND |
| RST (Pin 9) | Connected to the reset circuit |
| VCC (Pin 40) | +5V |
| GND (Pin 20) | Ground |
| EA (Pin 31) | +5V |

**Clock Frequency:** 11.0592 MHz

**LED Configuration:** Active-HIGH

The LED turns ON when P1.0 is HIGH (1) and turns OFF when P1.0 is LOW (0).

### Circuit Diagram

![Circuit Diagram](AT89S52%20LED%20Circuit%20Schematic.png)

---

## 2. Core Logic / Pseudocode / Code Base

### Working Principle

The AT89S52 microcontroller controls an LED through pin P1.0, which is used as a digital output pin.

The program uses the `sbit` keyword to assign the LED to P1.0. A nested `for` loop is used to generate a software delay.

Initially, P1.0 is set to HIGH, turning the LED ON. After a delay, P1.0 is set to LOW, turning the LED OFF. This process repeats continuously inside an infinite `while(1)` loop, producing a blinking LED.

The program uses direct register-level pin control without requiring external libraries.

### Pseudocode

1. Start the program.
2. Assign P1.0 to the LED using the `sbit` keyword.
3. Set P1.0 to HIGH to turn ON the LED.
4. Call the delay function.
5. Set P1.0 to LOW to turn OFF the LED.
6. Call the delay function again.
7. Repeat steps 3–6 continuously.

### Code Base

The C program is written using Keil C51 for the AT89S52 microcontroller.

```c
#include <stdio.h>
#include <reg51.h>

sbit ledpin = P1^0;

void delay()
{
    int i, j;

    for(i = 0; i < 500; i++)
    {
        for(j = 0; j < 500; j++)
        {
        }
    }
}

void main()
{
    while(1)
    {
        ledpin = 1;
        delay();

        ledpin = 0;
        delay();
    }
}
```

### Code Explanation

- `#include <stdio.h>`: Includes standard input/output declarations.
- `#include <reg51.h>`: Provides the 8051 microcontroller register definitions.
- `sbit ledpin = P1^0;`: Assigns the LED control signal to Port 1, pin 0.
- `delay()`: Generates a software delay using nested `for` loops.
- `ledpin = 1;`: Sets P1.0 HIGH and turns ON the LED.
- `ledpin = 0;`: Sets P1.0 LOW and turns OFF the LED.
- `while(1)`: Repeats the LED ON and OFF operations continuously.

---

## 3. Inference

The LED was successfully controlled using GPIO programming on the AT89S52 microcontroller. The LED blinked continuously through alternating HIGH and LOW logic levels at pin P1.0.

This experiment demonstrates basic digital output operation, software delay generation, and direct register-level programming of the 8051 microcontroller using Embedded C.