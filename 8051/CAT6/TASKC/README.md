# **EXPERIMENT DETAILS**

**EXP:** CONTINUOUS ASSESMENT 2

**DATE:** [Enter Date]

**NAME OF EXP:** 16×2 LCD Driver Development and Dynamic Display Control

**HARDWARE SETUP / PLATFORM:** AT89S52 Microcontroller + 16×2 LCD + 4×4 Matrix Keypad + Keil µVision

---

## **TASK 7C : LCD-Based Character Movement Using Matrix Keypad**

### **1. Circuit Diagram**

The experiment demonstrates interfacing a 4×4 matrix keypad with a 16×2 LCD to control the movement of a character in different directions.

- **Microcontroller:** AT89S52
- **Display:** 16×2 LCD
- **Keypad:** 4×4 Matrix Keypad
- **LCD Communication Mode:** 4-bit mode
- **LCD RS Pin:** P3.2
- **LCD Enable Pin:** P3.3
- **LCD Data Pins:** P3.4–P3.7 (D4–D7)
- **LCD RW Pin:** Connected to GND
- **Keypad Row Pins:** P1.0–P1.3 (Outputs)
- **Keypad Column Pins:** P1.4–P1.7 (Inputs)
- **Power Supply:** 5V DC
- **Crystal Frequency:** 11.0592 MHz

### **2. Core Logic / Pseudocode / Code Base**

**Working Principle:**

The 4×4 matrix keypad and 16×2 LCD are interfaced with the 8051 microcontroller. The LCD operates in 4-bit mode, while the keypad is connected to Port 1.

The keypad scanning function activates one row at a time by driving it LOW and checks the column pins to identify the pressed key. A keymap array is used to identify the corresponding character.

The character 'O' is initially displayed on the LCD. When the user presses a direction key, the microcontroller erases the character from its previous position and displays it at the new position.

The keys 2, 8, 4 and 6 are used for UP, DOWN, LEFT and RIGHT movements respectively. Debouncing and key-release detection are implemented to ensure one movement per key press.

**Pseudocode:**

1. Include the 8051 register definition header file.
2. Define Port 3 for LCD interfacing and Port 1 for keypad interfacing.
3. Assign P3.2 as RS and P3.3 as Enable.
4. Define the 4×4 keypad keymap.
5. Create a delay function for LCD timing and keypad debouncing.
6. Initialize the LCD in 4-bit, 2-line display mode.
7. Implement functions to send LCD commands and character data.
8. Implement a cursor positioning function to select the required row and column.
9. Implement the keypad scanning function:
   - Activate one keypad row at a time by driving it LOW.
   - Read the column pins.
   - Identify the pressed key using the keymap.
   - Apply a debounce delay and verify the key press.
10. Initialize the keypad and LCD.
11. Set the initial character position to row 0 and column 0.
12. Display the character 'O' on the LCD.
13. Continuously scan the keypad.
14. If key 2 is pressed, move the character UP.
15. If key 8 is pressed, move the character DOWN.
16. If key 4 is pressed, move the character LEFT.
17. If key 6 is pressed, move the character RIGHT.
18. Erase the previous character position and display 'O' at the new position.
19. Wait until the key is released before accepting another movement.
20. Repeat continuously.

**Code Base:**

The program is written in Embedded C using Keil C51.

- `reg51.h` provides the 8051 register definitions.
- `LCD_PORT = P3` assigns Port 3 for LCD interfacing.
- `KEYPAD = P1` assigns Port 1 for keypad interfacing.
- `sbit RS = P3^2` assigns the LCD Register Select pin.
- `sbit E = P3^3` assigns the LCD Enable pin.
- `keymap[4][4]` stores the characters corresponding to the keypad buttons.
- `lcd_send_nibble()` transmits 4-bit data to the LCD.
- `lcd_cmd()` sends commands to the LCD.
- `lcd_data()` sends character data to the LCD.
- `lcd_init()` initializes the LCD in 4-bit mode.
- `lcd_goto()` positions the LCD cursor at the specified row and column.
- `keypad_scan()` scans the keypad and identifies the pressed key.
- `wait_key_release()` waits until the pressed key is released.
- `row` and `col` store the current position of the moving character.
- Keys `2`, `8`, `4` and `6` control UP, DOWN, LEFT and RIGHT movements respectively.
- The old character position is cleared before displaying the character at the new position.

### **3. Inference**

The 4×4 matrix keypad and 16×2 LCD were successfully interfaced with the AT89S52 microcontroller. The character 'O' moves across the LCD display according to the direction keys pressed on the keypad.

The experiment demonstrates LCD interfacing, matrix keypad scanning, switch debouncing, cursor positioning and interactive input-based display control using the 8051 microcontroller.