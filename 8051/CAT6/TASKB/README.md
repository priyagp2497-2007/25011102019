# **EXPERIMENT DETAILS**

**EXP:** CONTINUOUS ASSESMENT 2

**DATE:** [Enter Date]

**NAME OF EXP:** 16×2 LCD Driver Development and Dynamic Display Control

**HARDWARE SETUP / PLATFORM:** AT89S52 Microcontroller + 16×2 LCD + Keil µVision

---

## **TASK 7B : Dynamic LCD Display with Cursor Positioning and Number Printing**

### **1. Circuit Diagram**

The experiment demonstrates dynamic message and counter display using a 16×2 LCD interfaced with the 8051 microcontroller in 4-bit mode.

- **Microcontroller:** AT89S52
- **Display:** 16×2 LCD
- **Communication Mode:** 4-bit mode
- **RS Pin:** P3.2
- **Enable Pin:** P3.3
- **Data Pins:** P3.4–P3.7 (D4–D7)
- **RW Pin:** Connected to GND
- **Power Supply:** 5V DC
- **Crystal Frequency:** 11.0592 MHz

### **2. Core Logic / Pseudocode / Code Base**

**Working Principle:**

The 16×2 LCD is connected to Port 3 of the 8051 microcontroller. The LCD operates in 4-bit mode, where data is transmitted through four data lines (D4–D7).

The program initializes the LCD and displays the message "Hello 8051" on the first row. The second row displays "Count: " followed by a numerical counter value.

The cursor positioning function places the cursor at the required row and column. A number-printing function converts the integer counter into individual decimal digits before displaying them.

The counter increments every 500 milliseconds, continuously updating the second row while the first-row message remains unchanged.

**Pseudocode:**

1. Include the 8051 register definition header file.
2. Define the LCD data port as Port 3.
3. Assign P3.2 as RS and P3.3 as Enable.
4. Create a delay function for LCD timing.
5. Create functions to send LCD commands and character data using 4-bit mode.
6. Initialize the LCD in 4-bit, 2-line display mode.
7. Create a cursor positioning function to select the required row and column.
8. Create a string display function to print messages.
9. Create a number-printing function to convert integers into decimal digits.
10. Display "Hello 8051" on the first row.
11. Display "Count: " on the second row.
12. Continuously update the counter value every 500 milliseconds.
13. Repeat the process indefinitely.

**Code Base:**

The program is written in Embedded C using Keil C51.

- `reg51.h` provides the 8051 register definitions.
- `LCD_PORT = P3` assigns Port 3 for LCD interfacing.
- `sbit RS = P3^2` assigns the Register Select pin.
- `sbit E = P3^3` assigns the Enable pin.
- `lcd_send_nibble()` transmits 4-bit data to the LCD.
- `lcd_cmd()` sends commands to configure and control the LCD.
- `lcd_data()` sends individual characters to the display.
- `lcd_init()` initializes the LCD in 4-bit mode.
- `lcd_goto()` positions the cursor at the specified row and column.
- `lcd_puts()` displays a complete string.
- `lcd_print_number()` converts an integer into decimal digits and displays it.
- `count++` increments the counter continuously.
- `delay_ms(500)` introduces a 500 ms delay between counter updates.

### **3. Inference**

The 16×2 LCD successfully displays the fixed message "Hello 8051" on the first row and a continuously increasing counter on the second row.

The experiment demonstrates 4-bit LCD interfacing, cursor positioning, string handling, integer-to-decimal conversion and dynamic display control using the 8051 microcontroller.