# EXPERIMENT 5

**EXP:** 5

**DATE:** [Enter Date]

**NAME OF EXP:** Timer-Based Programmable Waveform Generator Using Push Buttons

**HARDWARE SETUP / PLATFORM:** AT89S52 (8051 Microcontroller)

**TASK NO:** TASK A

---

## 1. Circuit Diagram

### Hardware Connections

| Component | AT89S52 Connection |
|---|---|
| Waveform Output Signal | P1.0 (Pin 1) |
| Frequency Selection Push Button | P3.4 (Pin 14) |
| Waveform Selection Push Button | P3.5 (Pin 15) |
| Push Button 1 | Connected between P3.4 and GND |
| Push Button 2 | Connected between P3.5 and GND |
| XTAL1 (Pin 19), XTAL2 (Pin 18) | 11.0592 MHz crystal |
| Crystal capacitors | Connected between each crystal terminal and GND |
| RST (Pin 9) | Reset circuit |
| VCC (Pin 40) | +5V |
| GND (Pin 20) | Ground |
| EA (Pin 31) | +5V |

**Clock Frequency:** 11.0592 MHz

**Timer Configuration:** Timer 0, Mode 2 (8-bit Auto-Reload Mode)

**Timer Interrupt Interval:** Approximately 50 µs

**Output Pin:** P1.0

**Push Button Configuration:** Active-LOW

The waveform is generated at P1.0. The two push buttons are used to change the output frequency and waveform pattern.

### Circuit Diagram

![Circuit Diagram](AT89S52%20CAT5%20Circuit%20Diagram.png)

---

## 2. Core Logic / Pseudocode / Code Base

### Working Principle

The AT89S52 microcontroller is used to generate different digital waveform patterns using Timer 0 interrupts.

The output signal is generated through P1.0. Two push buttons connected to P3.4 and P3.5 allow the user to change the waveform frequency and select different waveform patterns.

Timer 0 is configured in Mode 2, which is an 8-bit auto-reload mode. It generates periodic interrupts at approximately 50 µs intervals.

Inside the Timer 0 interrupt service routine, a counter is incremented. When the counter reaches the selected `bit_ticks` value, the next value from the selected waveform pattern is sent to P1.0.

This process continuously generates the selected waveform without using software delay loops.

### Timer 0 Configuration

Timer 0 is configured using the following registers:

- `TMOD`: Configures Timer 0 in Mode 2.
- `TH0`: Stores the timer auto-reload value.
- `TL0`: Stores the initial timer value.
- `ET0`: Enables Timer 0 interrupts.
- `EA`: Enables global interrupts.
- `TR0`: Starts Timer 0.

The timer reload value is set to `0xD2`.

For an 11.0592 MHz crystal, assuming a standard 12-clock 8051 timer:

\[
T_{machine}=\frac{12}{11.0592\times10^6}
\]

\[
T_{machine}\approx1.085\ \mu s
\]

The timer counts 46 increments before overflow.

\[
T_{interrupt}=46\times1.085
\]

\[
T_{interrupt}\approx50\ \mu s
\]

### Frequency Selection

The `half_ticks[]` array stores five different timing values.

```c
unsigned char code half_ticks[5] = {100, 20, 10, 5, 2};
```

The frequency selection button connected to P3.4 changes the `f_mode` variable.

Each button press selects the next timing value.

| Frequency Mode | bit_ticks | Square Wave Frequency (Approx.) |
|---|---:|---:|
| 0 | 100 | 100 Hz |
| 1 | 20 | 500 Hz |
| 2 | 10 | 1 kHz |
| 3 | 5 | 2 kHz |
| 4 | 2 | 5 kHz |

These frequency values apply to the square waveform. Other waveform patterns have different repetition frequencies because they contain different numbers of pattern elements.

### Waveform Selection

The program supports four waveform patterns.

**1. Square Wave**

- Alternates between HIGH and LOW.
- Contains equal HIGH and LOW intervals.
- Produces a square waveform with approximately 50% duty cycle.

**2. Variable Pattern Wave**

- Uses four HIGH values followed by four LOW values.
- Repeats the pattern continuously.
- In the current code, this pattern also has a 50% duty cycle.

**3. Pulse Wave**

- Produces a short HIGH pulse followed by a longer LOW interval.
- Repeats the pulse pattern continuously.
- The pattern has a 12.5% HIGH duty cycle.

**4. Burst Wave**

- Generates a sequence of alternating HIGH and LOW values.
- Produces a group of pulses followed by a longer LOW interval.
- Repeats the burst pattern continuously.

### Push Button Operation

Two push buttons are used to control the waveform generator.

**Frequency Selection Button (P3.4)**

- Changes the frequency timing value.
- Each valid press increments `f_mode`.
- After the fifth frequency setting, it returns to the first setting.

**Waveform Selection Button (P3.5)**

- Changes the waveform pattern.
- Each valid press increments `s_mode`.
- The sequence moves through square, variable pattern, pulse, and burst waveforms.
- After the fourth waveform, it returns to the square waveform.

The buttons are detected using falling-edge detection. A HIGH-to-LOW transition indicates a button press.

### Pseudocode

1. Start the program.
2. Initialize the waveform output pin P1.0 to LOW.
3. Configure P3.4 and P3.5 as input pins.
4. Initialize the frequency and waveform selection variables.
5. Load the initial waveform pattern.
6. Configure Timer 0 in Mode 2.
7. Load `0xD2` into TH0 and TL0.
8. Enable Timer 0 interrupts and global interrupts.
9. Start Timer 0.
10. In the Timer 0 interrupt routine:
    - Increment the timing counter.
    - Check whether the selected timing interval is reached.
    - Reset the counter when the interval is reached.
    - Output the next waveform value through P1.0.
    - Repeat the waveform pattern when its end is reached.
    - Generate a button-scanning flag every 20 ms.
11. In the main loop, check the scanning flag.
12. Read the two push button inputs.
13. If P3.4 is pressed:
    - Increment the frequency mode.
    - Select the next timing value.
14. If P3.5 is pressed:
    - Increment the waveform mode.
    - Load the next waveform pattern.
15. Repeat the process continuously.

### Code Base

The C program is written using Keil C51 for the AT89S52 microcontroller.

```c
#include <reg52.h>

sbit SIG   = P1^0;
sbit KEY_F = P3^4;
sbit KEY_S = P3^5;

unsigned char code half_ticks[5] = {100, 20, 10, 5, 2};

unsigned char code pat_square[] = {
    1,0,1,0,1,0,1,0
};

unsigned char code pat_vduty[] = {
    1,1,1,1,0,0,0,0,
    1,1,1,1,0,0,0,0
};

unsigned char code pat_pulse[] = {
    1,0,0,0,0,0,0,0,
    1,0,0,0,0,0,0,0,
    1,0,0,0,0,0,0,0
};

unsigned char code pat_burst[] = {
    1,0,1,0,1,0,1,0,1,0,
    0,0,0,0,0,0,0,0,0,0,
    1,0,1,0,1,0,1,0,1,0
};

unsigned char code *pat;
unsigned char pat_len = 8;
unsigned char pat_idx = 0;
unsigned char bit_ticks = 20;
unsigned char cnt = 0;

unsigned int scan_cnt = 0;
bit scan_flag = 0;

/* Timer 0 Interrupt Service Routine */
void Timer0_ISR(void) interrupt 1
{
    cnt++;

    if (cnt >= bit_ticks)
    {
        cnt = 0;

        SIG = pat[pat_idx];

        pat_idx++;

        if (pat_idx >= pat_len)
            pat_idx = 0;
    }

    scan_cnt++;

    if (scan_cnt >= 400)
    {
        scan_cnt = 0;
        scan_flag = 1;
    }
}

/* Load Selected Waveform */
void load_pattern(unsigned char t)
{
    EA = 0;

    switch (t)
    {
        case 0:
            pat = pat_square;
            pat_len = sizeof(pat_square);
            break;

        case 1:
            pat = pat_vduty;
            pat_len = sizeof(pat_vduty);
            break;

        case 2:
            pat = pat_pulse;
            pat_len = sizeof(pat_pulse);
            break;

        default:
            pat = pat_burst;
            pat_len = sizeof(pat_burst);
            break;
    }

    pat_idx = 0;
    cnt = 0;

    EA = 1;
}

/* Main Function */
void main(void)
{
    unsigned char f_mode = 0, s_mode = 0;
    unsigned char f_prev = 1, s_prev = 1;
    unsigned char f_now, s_now;

    SIG = 0;

    KEY_F = 1;
    KEY_S = 1;

    bit_ticks = half_ticks[f_mode];

    load_pattern(s_mode);

    TMOD = (TMOD & 0xF0) | 0x02;

    TH0 = 0xD2;
    TL0 = 0xD2;

    ET0 = 1;
    EA = 1;
    TR0 = 1;

    while (1)
    {
        if (scan_flag)
        {
            scan_flag = 0;

            f_now = KEY_F;
            s_now = KEY_S;

            if (f_prev == 1 && f_now == 0)
            {
                f_mode++;

                if (f_mode >= 5)
                    f_mode = 0;

                EA = 0;
                bit_ticks = half_ticks[f_mode];
                cnt = 0;
                EA = 1;
            }

            if (s_prev == 1 && s_now == 0)
            {
                s_mode++;

                if (s_mode >= 4)
                    s_mode = 0;

                load_pattern(s_mode);
            }

            f_prev = f_now;
            s_prev = s_now;
        }
    }
}
```

### Code Explanation

- `#include <reg52.h>`: Includes the 8051 microcontroller register definitions.
- `SIG`: Assigns P1.0 as the waveform output pin.
- `KEY_F`: Assigns P3.4 as the frequency selection button input.
- `KEY_S`: Assigns P3.5 as the waveform selection button input.
- `half_ticks[]`: Stores the five timing values used for frequency selection.
- `pat_square[]`: Stores the square waveform pattern.
- `pat_vduty[]`: Stores the variable waveform pattern.
- `pat_pulse[]`: Stores the pulse waveform pattern.
- `pat_burst[]`: Stores the burst waveform pattern.
- `pat`: Points to the currently selected waveform pattern.
- `pat_len`: Stores the number of elements in the selected pattern.
- `pat_idx`: Tracks the current position in the waveform pattern.
- `bit_ticks`: Determines the time interval between successive waveform elements.
- `Timer0_ISR()`: Generates the waveform output and sets the button-scanning flag.
- `load_pattern()`: Selects and loads the required waveform pattern.
- `scan_cnt`: Counts timer interrupts to generate a scanning interval of approximately 20 ms.
- `scan_flag`: Indicates when the main program should read the push buttons.
- `f_mode`: Stores the selected frequency mode.
- `s_mode`: Stores the selected waveform mode.
- `f_prev` and `s_prev`: Store the previous button states for falling-edge detection.

---

## 3. Inference

The AT89S52 microcontroller was used to generate different digital waveform patterns using Timer 0 interrupts.

Four waveform patterns, namely square, variable pattern, pulse, and burst, were implemented. Two push buttons allowed the user to select the waveform pattern and change its timing settings.

The experiment demonstrates hardware timer configuration, interrupt-driven waveform generation, frequency selection, digital signal generation, and push-button interfacing using Embedded C.