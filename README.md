# ECE528_Homework-2
## Section I: Review Questions
**1. What is a hardware timer? What kinds of tasks do embedded systems commonly use hardware timers for?**
    A hardware timer are timer peripherals in the MSP432 such as SysTick, Timer_A, Timer32, Watchdog Timer, and Real-Time Clock (RTC). These hardware timers are used for creating precise delays and can also interrupt at specific times to perform tasks.

**2. What is a clock signal used for in embedded systems? List the primary clock sources available on the MSP432P401R microcontroller and briefly describe each clock source.**
    The clock signal is used to determine how many clock cycles occurs per second, it also helps drive hardware timers and counters. Five primary system clock signals are:
    ACLK: Auxilary clock, is software selectable by individual peripheral modules.
    MCLK: Master clock, used by the CPU and peripheral module interfaces.
    HSMCLK: Subsystem master clock, is software selectable by individual peripheral modules.
    SMCLK: Low-speed subsystem master clock, can be divided independently from HSMCLK.
    BCLK: Low-speed backup domain clock, is used primarily in the backup domain.

**3. Consider a 16-bit timer (e.g. Timer_A configured in up mode). If it counts upward, what is the maximum counnt value before it rolls over?**
    Using 2^b - 1 we can determine the maximum count value before it rolls over. Maximum count = 2^16 - 1 = 65535.

**4. If a 24-bit timer counts upward from zero at 10 MHz, what is its overflow period?**
    Overflow period = 2^b / f
    2^24 / 10MHz = 1.6777 seconds if the overflow period.

**5. Describe the SysTick timer on the MSP432P401R microcontroller. Provide two example applications for SysTick in embedded systems. Which clock sources can the SysTick timer use and how can they be selected?**
    The SysTick timer is a simple 24-bit clear on write decrementing wrap on zero counter with a flexible control mechanism. Two applications for a SysTick timer is for a high speed alarm timer using the system clock and another application is a simple counter used to measure time to completion and time used. It can use FCLK and CLKSOURCE bit in the SYSTICK control and Status register must always be 1 to indicate CPU clock as well as reference clock.

**6. How many registers does the SysTick timer have on the MSP432P401R microcontroller and what is the purpose of each? Name them and provide a short description per register.**
    The SysTick timer has 3 registers.
    STCSR: Control and Status Register, A control and status register to configure clock, enable counter, enable SysTick interrupt, and determine counter status.
    STRVR: Reload Value Register, used to provide the counter's wrap value.
    STCVR: Current Value Register, The current value of the counter.

**7. Describe the Timer_A peripheral on the MSP432P401R microcontroller. List the main operating modes of Timer_A and briefly explain each.**
    The Timer_A peripheral is a 16-bit timer/counter and can support multiple capute/compares, PWM outputs, and interval timing. The four main operating modes are:
    Stop Mode: timer is halted.
    Up Mode: timer repeatedly counts from zero to the value of TAxCCR0.
    Continous Mode: timer repeatedly counts from zero to 0FFFFh.
    Up/down Mode: timer repeatedly counts from zero up to the value of TAxCCR0 and back down to zero. 

**8. Briefly state the purpose of the following Timer_A registers. TAxCTL, TAxCCTLn, TAxR, TAxCCRn, TaxIV, and TAxEX0.**
    TAxCTL: Control Register, configures the clock source, clock divider, operating mode, timer clearing, and enables timer interrupt flags.
    TAxCCTLn: Capture/Compare Control, configures a capture/compare channel, output behavior, and channel interrupt setting.
    TAxR: Counter Register, contains the current 16-bit counter value.
    TAxCCRn: Capture/Compare Register, stores a compare value used to trigger an event, control PWM timing, or capture the counter value.
    TAxIV: Interrupt Vector Register, identifies highest priority pending Timer_A interrupt source.
    TAxEX0: Expansion 0 Register, select the divider for the input clock.

**9. What is PWM? Provide at least three example applications that use PWM in embedded systems.**
    The Pulse width modulation (PWM) the technique that uses pulse widths to encode or modulate a signal. One application that uses PWM is by controling the LED brightness by decreasing the PWM signal it can lower the LED's brightness and vise versa. A second example is by applying a PWM signal to EN to drive the motor.A third example is by generating timing pulses that can be used to control the servo position.

**10. How many Timer_A modules are available and how many PWM outputs can each module provide?**
    There are 4 Timer_A modules available and and each one can provide up to 4 PWM outputs.

**11. Describe the steps needed to initialize P2.4 as a PWM output driven by Timer_A0 CCR1 (TA0.1).**

    void Timer_A0_PWM_Init()
    {
        P2->SEL0 |= 0x10;
        P2->SEL1 &= ~(0x10);
        P2->DIR |= 0x10;

        TIMER_A0->CTL |= 0x0274; // SMCLK, divide by 2, up/down, clear
        TIMER_A0->EX0 = 0;

        TIMER_A0->CCR[0] = 60000; // Period=(2*60000)/(12MHz/2)

        TIMER_A0->CCTL[1] |= 0x0040;
        TIMER_A0->CCR[1] |= 30000;
        TIMER_A0->CTL |= 0x30;
    }

**12. Describe the steps needed to configue Timer_A0 to generate a periodic interrupt every 2 ms with an interrupt priority level of 2. Note that Timer_A0 has an interrupt number of 8.**

    void Timer_A0_Interrupt_Init()
    {
        TIMER_A0->CTL &= ~(0x0030); // Halt timer
        TIMER_A0->CTL |= 0x0200; // ID=0
        TIMER_A0->CCTL[0] |= 0x0010; // Interrupt
        TIMER_A0->CCR[0] = 24000-1; // 12Mz*2ms
        TIMER_A0->EX0 = 0;
        NVIC->IP[2] = (NVIC->IP[2] & 0xFFFFFF00) | 0x00000040;
        NVIC->ISER[0] |= 0x00000100;
        TIMER_A0->CTL |= 0x0014;
    }