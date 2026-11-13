# UART-Based Serial Communication Using 8051

## Experiment Details

- **Experiment No.:** CAT7
- **Experiment Name:** UART-Based Serial Communication Using 8051
- **Microcontroller:** AT89S52
- **Programming Language:** Embedded C
- **IDE:** Keil µVision
- **Communication Protocol:** UART
- **Baud Rate:** 9600 bps

---

## 1. Circuit Diagram

![Circuit Diagram](circuit.png)

*Note: Replace `circuit.png` with the exact filename of your circuit diagram.*

## 2. Hardware Connections

| Component | Connection |
|---|---|
| AT89S52 P3.1 (TXD, Pin 11) | USB-to-TTL Converter RXD |
| AT89S52 P3.0 (RXD, Pin 10) | Not used in this transmitter-only program |
| AT89S52 GND (Pin 20) | USB-to-TTL Converter GND |
| AT89S52 VCC (Pin 40) | +5V |
| XTAL1 (Pin 19), XTAL2 (Pin 18) | 11.0592 MHz crystal |
| RST (Pin 9) | Reset circuit |
| EA (Pin 31) | +5V |

**Important:**
- Connect the USB-to-TTL converter's RXD to the microcontroller's TXD pin (P3.1).
- Connect both grounds together.
- Use a USB-to-TTL converter compatible with the microcontroller's TTL voltage levels.
- Do not connect the 8051 directly to an RS-232 port.

## 3. Working Principle

1. The UART is initialized in Mode 1, which supports 8-bit serial communication.
2. The receiver enable bit (REN) is set, although this program only transmits data.
3. Timer 1 is configured in Mode 2 (8-bit auto-reload mode) to generate the baud rate.
4. TH1 is loaded with `0xFD` to obtain a baud rate of 9600 bps using an 11.0592 MHz crystal.
5. The `uart_tx()` function sends one character through the serial port using the SBUF register.
6. The program waits until the TI flag becomes 1, indicating that transmission is complete.
7. The `uart_puts()` function sends a complete string character by character.
8. The main function continuously transmits the message **"Hello from 8051"** followed by carriage return and line feed.

## 4. Pseudocode

1. Start the program.
2. Initialize UART:
   - Configure serial port in Mode 1.
   - Configure Timer 1 in Mode 2.
   - Load TH1 and TL1 with `0xFD`.
   - Start Timer 1.
   - Clear TI and RI flags.
3. Enter an infinite loop.
4. Transmit the string "Hello from 8051".
5. For every character:
   - Load the character into SBUF.
   - Wait until TI becomes 1.
   - Clear TI.
6. Repeat the transmission continuously.

## 5. Code

```c
#include <reg52.h>

void uart_init(void)
{
    SCON = 0x50;
    PCON &= 0x7F;
    TMOD &= 0x0F;
    TMOD |= 0x20;
    TH1 = 0xFD;
    TL1 = 0xFD;
    TR1 = 1;
    TI = 0;
    RI = 0;
}

void uart_tx(char c)
{
    SBUF = c;
    while (TI == 0);
    TI = 0;
}

void uart_puts(char *s)
{
    while (*s)
    {
        uart_tx(*s);
        s++;
    }
}

void main(void)
{
    uart_init();

    while (1)
    {
        uart_puts("Hello from 8051\r\n");
    }
}
```

## 6. Code Explanation

### `uart_init()`
- `SCON = 0x50` configures UART in Mode 1 and enables reception.
- `PCON &= 0x7F` keeps SMOD equal to 0.
- `TMOD &= 0x0F` clears the Timer 1 configuration bits.
- `TMOD |= 0x20` selects Timer 1 Mode 2.
- `TH1 = 0xFD` and `TL1 = 0xFD` configure the baud rate.
- `TR1 = 1` starts Timer 1.
- `TI = 0` and `RI = 0` clear the transmission and reception flags.

### `uart_tx(char c)`
- Loads one character into the SBUF register.
- Transmission starts automatically.
- Waits until the TI flag becomes 1.
- Clears TI after transmission.

### `uart_puts(char *s)`
- Transmits a complete string.
- Sends each character individually using `uart_tx()`.
- Continues until the null character (`'\0'`) is encountered.

### `main()`
- Initializes UART communication.
- Continuously transmits the message `"Hello from 8051\r\n"`.

## 7. Baud Rate Calculation

For Timer 1 Mode 2:

\[
Baud\ Rate = \frac{2^{SMOD}}{32}
\times \frac{F_{osc}}{12(256-TH1)}
\]

Given:

- Crystal frequency = 11.0592 MHz
- SMOD = 0
- TH1 = FD (253)

\[
Baud\ Rate = \frac{11.0592 \times 10^6}{384(256-253)}
\]

\[
Baud\ Rate = 9600\ bps
\]

## 8. Expected Output

Configure the serial terminal with:

- **Baud Rate:** 9600
- **Data Bits:** 8
- **Parity:** None
- **Stop Bits:** 1
- **Flow Control:** None

Expected terminal output:

```text
Hello from 8051
Hello from 8051
Hello from 8051
Hello from 8051
...
```

The message is transmitted continuously until the microcontroller is stopped or reset.

## 9. Inference

Thus, UART serial communication was successfully implemented using the AT89S52 microcontroller. Timer 1 was configured in Mode 2 to generate a baud rate of 9600 bps, and the message was continuously transmitted through the TXD pin using the SBUF register.