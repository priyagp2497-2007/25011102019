# 16×2 LCD Interfacing with 8051 Using 4-Bit Mode

## Experiment Details

- **Experiment No.:** CAT8
- **Experiment Name:** 16×2 LCD Interfacing and Display Control
- **Microcontroller:** AT89S52
- **Programming Language:** Embedded C
- **IDE:** Keil µVision
- **Display:** 16×2 LCD
- **Communication Mode:** 4-bit mode

---

## 1. Circuit Diagram

![Circuit Diagram](circuit.png)

*Replace `circuit.png` with the actual filename of your circuit diagram.*

## 2. Hardware Connections

| LCD Pin | Function | 8051 Connection |
|---|---|---|
| VSS | Ground | GND |
| VDD | Power supply | +5V |
| V0 | Contrast adjustment | Middle pin of 10K potentiometer |
| RS | Register Select | P3.2 |
| RW | Read/Write | GND |
| E | Enable | P3.3 |
| D4 | Data bit 4 | P3.4 |
| D5 | Data bit 5 | P3.5 |
| D6 | Data bit 6 | P3.6 |
| D7 | Data bit 7 | P3.7 |
| LED+ | Backlight anode | +5V through suitable resistor, if required |
| LED- | Backlight cathode | GND |

**8051 minimum connections:**

| Component | Connection |
|---|---|
| VCC (Pin 40) | +5V |
| GND (Pin 20) | Ground |
| XTAL1 (Pin 19), XTAL2 (Pin 18) | 11.0592 MHz crystal |
| RST (Pin 9) | Reset circuit |
| EA (Pin 31) | +5V |

**Important:** The LCD data pins D4–D7 use P3.4–P3.7. Ensure that the LCD is configured for 4-bit operation. The RW pin is grounded because the program only writes to the LCD.

## 3. Working Principle

1. The 8051 microcontroller initializes the 16×2 LCD.
2. The LCD is configured to operate in 4-bit mode, using only four data lines (D4–D7).
3. The RS pin selects whether the LCD receives a command or display data.
4. The Enable (E) pin provides the signal required to latch the data or command.
5. LCD commands are sent with RS = 0.
6. Display characters are sent with RS = 1.
7. Each 8-bit command or character is divided into two 4-bit nibbles:
   - Higher nibble is sent first.
   - Lower nibble is sent next.
8. The LCD displays the message **HELLO FROM 8051**.
9. The display is cleared after the message is written, and the process repeats continuously.

## 4. Pseudocode

1. Start the program.
2. Initialize the LCD:
   - Set RS = 0 and E = 0.
   - Wait for the LCD power-up delay.
   - Configure the LCD for 4-bit mode.
   - Select 2-line display mode.
   - Turn ON the display and turn OFF the cursor.
   - Set the cursor to move right after each character.
   - Clear the display.
3. Enter an infinite loop.
4. Send each character of the message "HELLO FROM 8051" to the LCD.
5. Clear the LCD.
6. Repeat the process continuously.

## 5. Code

```c
#include <reg51.h>

#define LCD_PORT P3

sbit RS = P3^2;
sbit E  = P3^3;

void delay_ms(unsigned int ms)
{
    unsigned int i, j;

    for(i = 0; i < ms; i++)
    {
        for(j = 0; j < 1275; j++);
    }
}

void lcd_enable()
{
    E = 1;
    delay_ms(1);
    E = 0;
}

void lcd_send_nibble(unsigned char nibble)
{
    LCD_PORT = (LCD_PORT & 0x0F) | (nibble & 0xF0);
    lcd_enable();
}

void lcd_cmd(unsigned char cmd)
{
    RS = 0;

    lcd_send_nibble(cmd);
    lcd_send_nibble(cmd << 4);

    delay_ms(2);
}

void lcd_data(unsigned char datas)
{
    RS = 1;

    lcd_send_nibble(datas);
    lcd_send_nibble(datas << 4);

    delay_ms(1);
}

void lcd_init()
{
    RS = 0;
    E = 0;

    delay_ms(20);

    lcd_send_nibble(0x20);
    lcd_cmd(0x28);
    lcd_cmd(0x0C);
    lcd_cmd(0x06);
    lcd_cmd(0x01);

    delay_ms(2);
}

void main()
{
    lcd_init();

    while(1)
    {
        lcd_data('H');
        lcd_data('E');
        lcd_data('L');
        lcd_data('L');
        lcd_data('O');
        lcd_data(' ');
        lcd_data('F');
        lcd_data('R');
        lcd_data('O');
        lcd_data('M');
        lcd_data(' ');
        lcd_data('8');
        lcd_data('0');
        lcd_data('5');
        lcd_data('1');
        lcd_data(' ');
        lcd_data(' ');
        lcd_data(' ');
        lcd_data(' ');
        lcd_data(' ');

        lcd_cmd(0x01);
    }
}
```

## 6. Code Explanation

### `delay_ms()`
Generates a software delay required for LCD operations and timing.

### `lcd_enable()`
- Sets the Enable pin HIGH.
- Waits briefly.
- Sets the Enable pin LOW.
- This allows the LCD to latch the supplied nibble.

### `lcd_send_nibble()`
- Sends a 4-bit data portion through P3.4–P3.7.
- Preserves the lower four bits of Port 3.
- Generates an Enable pulse.

### `lcd_cmd()`
- Sets RS = 0 to select the command register.
- Sends the higher nibble followed by the lower nibble.
- Used for LCD configuration and control instructions.

### `lcd_data()`
- Sets RS = 1 to select the data register.
- Sends the higher and lower nibbles of a character.
- Displays the character on the LCD.

### `lcd_init()`
Initializes the LCD:
- `0x20` – Selects 4-bit interface initialization.
- `0x28` – Selects 4-bit mode, 2-line display and 5×8 character font.
- `0x0C` – Turns ON the display and turns OFF the cursor.
- `0x06` – Sets the cursor to move right after each character.
- `0x01` – Clears the display.

### `main()`
- Calls `lcd_init()` to initialize the LCD.
- Continuously sends the characters of the message to the LCD.
- Clears the display after sending the message.

## 7. Expected Output

<text>LCD displays:</text>

```text
HELLO FROM 8051
```

The message is displayed on the first row of the 16×2 LCD. The display is then cleared and the process repeats continuously.

## 8. Inference

Thus, a 16×2 LCD was interfaced with the AT89S52 microcontroller using 4-bit communication mode. LCD commands and character data were transmitted through the data and control pins to display the message **HELLO FROM 8051**.