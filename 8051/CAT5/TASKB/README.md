# EXPERIMENT 6

**EXP:** 6

**DATE:** 24/8/2026

**NAME OF EXP:** UART Driver and Serial Debugging

**HARDWARE SETUP / PLATFORM:** AT89S52 (8051 Microcontroller)

**TASK NO:** TASK A

---

## 1. Circuit Diagram

### Hardware Connections

| Component | AT89S52 Connection |
|---|---|
| UART Transmit (TXD) | P3.1 (Pin 11) |
| UART Receive (RXD) | P3.0 (Pin 10) |
| Crystal Oscillator | 11.0592 MHz connected between XTAL1 and XTAL2 |
| Crystal Capacitors | Connected between crystal terminals and GND |
| RST (Pin 9) | Reset circuit |
| VCC (Pin 40) | +5V |
| GND (Pin 20) | Ground |
| EA (Pin 31) | +5V |

**Clock Frequency:** 11.0592 MHz

**Baud Rate:** 9600 bps

**UART Configuration:** Mode 1, 8-N-1

**Communication Method:** Interrupt-driven UART reception and transmission

**Timer Configuration:** Timer 1, Mode 2 (8-bit Auto-Reload Mode)

### UART Communication Connections

For serial communication with a computer, use a USB-to-TTL serial converter.

| AT89S52 | USB-to-TTL Converter |
|---|---|
| P3.1 (TXD) | RXD |
| P3.0 (RXD) | TXD |
| GND | GND |

The USB-to-TTL converter is connected to the computer through USB. The serial terminal software is used to send and receive characters.

**Important:** Use a TTL converter compatible with the AT89S52's 5V logic levels. Do not connect the microcontroller UART pins directly to a computer's RS-232 port.

### Circuit Diagram

![Circuit Diagram](AT89S52%20CAT6%20Circuit%20Diagram.png)

---

## 2. Core Logic / Pseudocode / Code Base

### Working Principle

The AT89S52 microcontroller communicates with a computer using Universal Asynchronous Receiver Transmitter (UART) communication.

The UART module is configured in Mode 1 to support 8-bit serial communication with one start bit, one stop bit, and no parity bit.

Timer 1 is configured in Mode 2 (8-bit auto-reload mode) to generate the required baud rate of 9600 bps using an 11.0592 MHz crystal oscillator.

The program uses serial interrupts to handle incoming and outgoing data. Received characters are stored in a circular buffer, allowing the main program to process the data separately from the interrupt routine.

When a character is received, the main program reads it from the buffer and sends the same character back to the serial terminal. This operation is called UART echo.

### UART Initialization

The `uart_init()` function configures the UART communication settings.

The following registers are used:

- `SCON = 0x50`: Configures UART in Mode 1 and enables reception.
- `PCON &= 0x7F`: Clears the SMOD bit.
- `TMOD`: Configures Timer 1 in Mode 2.
- `TH1 = 0xFD`: Sets the reload value for 9600 baud.
- `TL1 = 0xFD`: Loads the initial timer value.
- `TR1 = 1`: Starts Timer 1.
- `ES = 1`: Enables serial interrupts.
- `EA = 1`: Enables global interrupts.

### Baud Rate Calculation

For an 8051 microcontroller using Timer 1 Mode 2:

\[
Baud=\frac{2^{SMOD}}{32}\times
\frac{F_{osc}}{12(256-TH1)}
\]

Given:

- Crystal frequency = 11.0592 MHz
- SMOD = 0
- TH1 = FD (253)

\[
Baud=\frac{11.0592\times10^6}
{384(256-253)}
\]

\[
Baud=\frac{11.0592\times10^6}{1152}
\]

\[
\boxed{Baud=9600\ bps}
\]

### Interrupt-Based UART Communication

The `uart_isr()` function is the serial interrupt service routine.

It handles two conditions:

**1. Receive Interrupt (RI)**

- Checks whether the RI flag is set.
- Clears the RI flag.
- Reads the received character from SBUF.
- Stores the character in the circular receive buffer.
- Updates the buffer head position.
- If the buffer is full, the newly received character is discarded.

**2. Transmit Interrupt (TI)**

- Checks whether the TI flag is set.
- Clears the TI flag.
- Sets `tx_done = 1` to indicate that transmission is complete.

### Circular Buffer Operation

A circular buffer is used to temporarily store received UART characters.

The program defines:

- `RX_SIZE = 16`: Buffer capacity.
- `RX_MASK = 15`: Used for circular buffer indexing.
- `rx_head`: Points to the next position where received data is stored.
- `rx_tail`: Points to the next position from which data is read.

The buffer uses a circular arrangement, so after reaching the last position, the index returns to the beginning.

One position is reserved to distinguish a full buffer from an empty buffer. Therefore, the buffer can store up to 15 characters at a time.

### UART Driver Functions

**1. uart_init()**

Initializes UART communication and configures Timer 1 to generate the required baud rate.

**2. uart_tx()**

Transmits a single character through UART. It waits until the previous transmission is completed before sending the next character.

**3. uart_puts()**

Transmits a complete string character by character.

**4. uart_available()**

Checks whether any received character is available in the circular buffer.

**5. uart_getc()**

Reads one character from the receive buffer and updates the tail position.

**6. uart_isr()**

Handles UART receive and transmit interrupts.

### Pseudocode

1. Start the program.
2. Initialize UART communication.
3. Configure UART in Mode 1.
4. Configure Timer 1 in Mode 2.
5. Set the baud rate to 9600 bps.
6. Enable serial and global interrupts.
7. Start Timer 1.
8. Transmit the message "UART Echo Ready".
9. Enter an infinite loop.
10. Check whether received data is available in the circular buffer.
11. If data is available:
    - Read one character from the buffer.
    - Check whether the received character is the Enter key (`\r`).
    - If Enter is pressed, transmit carriage return and newline.
    - Otherwise, transmit the received character back to the terminal.
12. In the serial interrupt routine:
    - If RI is set, store the received character in the circular buffer.
    - If TI is set, mark transmission as complete.
13. Repeat continuously.

### Code Base

The C program is written using Keil C51 for the AT89S52 microcontroller.

```c
#include <reg52.h>

#define RX_SIZE 16
#define RX_MASK (RX_SIZE - 1)

static unsigned char rx_buf[RX_SIZE];

static volatile unsigned char rx_head = 0;
static volatile unsigned char rx_tail = 0;

static volatile bit tx_done = 1;

/* UART Initialization */
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

    ES = 1;
    EA = 1;
}

/* Serial Interrupt Service Routine */
void uart_isr(void) interrupt 4
{
    unsigned char next;

    if (RI)
    {
        RI = 0;

        next = (rx_head + 1) & RX_MASK;

        if (next != rx_tail)
        {
            rx_buf[rx_head] = SBUF;
            rx_head = next;
        }
        else
        {
            next = SBUF;
        }
    }

    if (TI)
    {
        TI = 0;
        tx_done = 1;
    }
}

/* Transmit a Character */
void uart_tx(char c)
{
    while (!tx_done);

    tx_done = 0;
    SBUF = c;
}

/* Transmit a String */
void uart_puts(char *s)
{
    while (*s)
        uart_tx(*s++);
}

/* Check Received Data Availability */
bit uart_available(void)
{
    return (rx_head != rx_tail);
}

/* Read a Received Character */
char uart_getc(void)
{
    char c;

    while (rx_head == rx_tail);

    c = rx_buf[rx_tail];

    rx_tail = (rx_tail + 1) & RX_MASK;

    return c;
}

/* Main Function */
void main(void)
{
    char c;

    uart_init();

    uart_puts("UART Echo Ready\r\n");

    while (1)
    {
        if (uart_available())
        {
            c = uart_getc();

            if (c == '\r')
            {
                uart_tx('\r');
                uart_tx('\n');
            }
            else
            {
                uart_tx(c);
            }
        }
    }
}
```

### Code Explanation

- `#include <reg52.h>`: Includes the 8051 microcontroller register definitions.
- `RX_SIZE`: Defines the circular buffer size as 16.
- `RX_MASK`: Defines the mask used for circular buffer indexing.
- `rx_buf[]`: Stores the received UART characters.
- `rx_head`: Tracks the next position where incoming data is stored.
- `rx_tail`: Tracks the next position from which data is read.
- `tx_done`: Indicates whether the previous transmission has completed.
- `uart_init()`: Initializes UART communication and Timer 1.
- `uart_isr()`: Handles UART receive and transmit interrupts.
- `uart_tx()`: Sends a single character through UART.
- `uart_puts()`: Sends a complete string through UART.
- `uart_available()`: Checks whether received data is available.
- `uart_getc()`: Retrieves one character from the circular buffer.
- `main()`: Initializes the UART, displays the ready message, and continuously receives and echoes characters.

---

## 3. Inference

UART communication was implemented successfully using the AT89S52 microcontroller.

Timer 1 was configured in Mode 2 to generate a baud rate of 9600 bps. Serial interrupts were used to handle incoming and outgoing data, while a circular buffer was implemented to store received characters.

The microcontroller received characters from the serial terminal and transmitted the same characters back, demonstrating UART initialization, serial communication, interrupt handling, circular buffer management, and basic UART driver development.