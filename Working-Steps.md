# Smart Pulse Oximeter -- Working Steps

This document explains how to assemble, build, run, and test the Smart
Pulse Oximeter project.

## 1. Project Overview

The project uses a MAX30102 optical sensor to collect red and infrared
photoplethysmography (PPG) signals. An STM32F401CC microcontroller
processes the sensor readings and displays the calculated values on a
16x2 LCD. It also sends waveform samples and vital readings to a web
dashboard through UART.

**Data flow:**

`MAX30102 Sensor → STM32F401CC → LCD`

`STM32F401CC → UART / USB-to-TTL Adapter → Web Dashboard`

The dashboard can also run in demo mode using simulated data. Demo
readings are not measurements from the sensor.

## 2. Hardware and Software Requirements

### Hardware

-   STM32F401CC development board
-   MAX30102 pulse oximeter sensor module
-   16x2 LCD
-   USB-to-TTL serial adapter (3.3 V logic)
-   ST-Link programmer/debugger
-   Jumper wires and breadboard
-   USB cable and a computer

### Software

-   STM32CubeIDE
-   STM32 device support/CMSIS files for STM32F401CC
-   Visual Studio Code
-   Live Server extension for VS Code
-   Google Chrome or Microsoft Edge on desktop

## 3. Hardware Connections

Make all connections with the power switched off. Check the voltage
requirements of your particular MAX30102 and LCD modules before
connecting power. Connect all grounds together.

### 3.1 MAX30102 to STM32F401CC (I2C1)

  MAX30102 pin   STM32F401CC pin
  -------------- --------------------------------------------------------
  SCL            PB8
  SDA            PB9
  GND            GND
  VIN            Suitable supply for the specific sensor breakout board

### 3.2 16x2 LCD to STM32F401CC

The firmware uses the following LCD pin mapping:

  LCD pin/signal   STM32F401CC pin
  ---------------- ------------------------
  RS               PA0
  E                PA1
  D4               PA4
  D5               PA5
  D6               PA6
  D7               PA7
  RW               GND
  VSS              GND
  VDD              Suitable LCD supply
  V0               Contrast potentiometer

Use a contrast potentiometer on V0. Ensure that the LCD logic voltage is
compatible with the STM32 pins.

### 3.3 UART connection to USB-to-TTL adapter

  STM32F401CC       USB-to-TTL adapter
  ----------------- --------------------
  PA2 (USART2 TX)   RX
  PA3 (USART2 RX)   TX
  GND               GND

Use a USB-to-TTL adapter with **3.3 V logic levels**. Do not connect the
adapter's 5 V output to an STM32 signal pin. The firmware is configured
for a baud rate of **115200**.

### 3.4 Programming connection

Connect the ST-Link to the board's SWD interface (SWDIO, SWCLK, and GND;
target voltage/reference as required by the programmer and board).

## 4. Build and Flash the STM32 Firmware

1.  Install and open **STM32CubeIDE**.
2.  Create a project for the exact STM32F401CC device used by your
    board.
3.  Make sure the project includes the matching STM32 CMSIS device
    header, startup file, and linker configuration.
4.  Add the supplied firmware source file to the project's source folder
    (for example, `Core/Src/main.c`).
5.  Check that the device header included by the source is available in
    the project. The supplied source uses `stm32f401xc.h` and
    `<stdint.h>`.
6.  Review the clock configuration before building. The supplied I2C and
    UART timing values assume a **16 MHz PCLK1**. If your project uses a
    different peripheral clock, update the timing values in the firmware
    accordingly.
7.  Build the project and resolve any compiler or linker errors.
8.  Connect the ST-Link and flash the firmware to the STM32F401CC.
9.  Reset the board.

During startup, the LCD should show the project initialization message.
If the MAX30102 is detected, the firmware proceeds to its
finger-placement and data-collection messages. If initialization fails,
check the sensor wiring and power.

## 5. Run the Web Dashboard

1.  Create a folder for the dashboard on your computer.
2.  Copy the supplied `index.html` file into that folder.
3.  Open the folder in **Visual Studio Code**.
4.  Install the **Live Server** extension if it is not already
    installed.
5.  Right-click `index.html` and select **Open with Live Server**.
6.  Open the page in Google Chrome or Microsoft Edge on a desktop
    computer.

The dashboard starts in demo mode automatically. It displays simulated
readings and a simulated waveform so that the interface can be checked
without connecting the hardware.

## 6. Connect the Dashboard to the Hardware

1.  Connect the STM32 USART2 pins to the USB-to-TTL adapter as described
    in Section 3.3.
2.  Connect the adapter to the computer using USB.
3.  Close any serial monitor or terminal program currently using the
    adapter's COM port.
4.  In the dashboard, click **Connect UART**.
5.  Approve the browser's serial-port permission request.
6.  Select the COM port corresponding to the USB-to-TTL adapter.
7.  Connect to the port at **115200 baud**.
8.  After a successful connection, the dashboard stops demo mode and
    displays data received from the STM32.

The dashboard uses the browser's Web Serial API. Use a supported desktop
version of Chrome or Edge and serve the page from localhost (for
example, through Live Server). A browser permission prompt is expected.

## 7. Data Sent to the Dashboard

The firmware sends waveform samples and vital-sign records over UART.

-   Waveform records use the `W:` prefix, followed by a sample value.
    Example: `W:100245`
-   Vital records use the `V:` prefix and comma-separated fields.
    Example:

`V:HR:75,SPO2:98,PI:24,SQI:92,HRV:42,RR:16,BEAT:8`

The dashboard parses the received records and displays the supported
values, including:

  Parameter      Meaning
  -------------- ----------------------------------------------------
  HR             Heart rate in beats per minute
  SPO2           Estimated peripheral oxygen saturation
  PI             Perfusion index
  SQI            Signal quality indicator
  HRV            Heart rate variability value (RMSSD, milliseconds)
  RR             Estimated respiration rate per minute
  BEAT           Beat-related counter/value sent by the firmware
  PPG waveform   Incoming infrared waveform samples

Values are estimates produced by the supplied firmware. Their accuracy
has not been clinically validated.

## 8. Testing Procedure

1.  First run the dashboard without hardware and confirm that demo
    readings and the waveform appear.
2.  Build and flash the firmware.
3.  Check that the LCD initializes correctly.
4.  Check that the MAX30102 is detected. If the sensor initialization
    fails, recheck its power, ground, SCL, and SDA connections.
5.  Place a finger gently over the sensor and keep it still while
    readings are collected.
6.  Connect the USB-to-TTL adapter and select its COM port in the
    dashboard.
7.  Confirm that the dashboard changes from demo mode to live serial
    data.
8.  Check that the waveform and vital-sign fields update as UART records
    arrive.

## 9. Troubleshooting

  -----------------------------------------------------------------------
  Problem                             What to check
  ----------------------------------- -----------------------------------
  LCD is blank                        Check LCD power, GND, contrast
                                      potentiometer, and RS/E/data-pin
                                      wiring.

  MAX30102 initialization fails       Check VIN, GND, PB8/SCL, PB9/SDA,
                                      and the sensor breakout's voltage
                                      requirements.

  No serial port appears              Reconnect the USB-to-TTL adapter
                                      and check that its driver is
                                      installed.

  Browser cannot connect to serial    Use desktop Chrome/Edge, serve from
                                      localhost, grant permission, and
                                      close other programs using the COM
                                      port.

  Dashboard connects but no live      Confirm TX-to-RX and RX-to-TX
  values appear                       wiring, common GND, 115200 baud,
                                      and that the STM32 firmware is
                                      running.

  Readings are unstable               Keep the finger still, ensure good
                                      sensor contact, and check the
                                      sensor wiring and power.

  Firmware timing or communication is Verify the actual PCLK1 frequency;
  incorrect                           the supplied timing values assume
                                      16 MHz.
  -----------------------------------------------------------------------

## 10. Important Notes and Limitations

-   The dashboard's demo mode uses simulated data. Do not report demo
    values as sensor measurements.
-   The live dashboard receives data through a wired UART connection
    using a USB-to-TTL adapter; this implementation does not use Wi-Fi
    or Bluetooth.
-   The supplied firmware uses experimental signal-processing
    calculations. The readings should not be treated as medical advice
    or used for diagnosis.
-   Validate the device against an approved reference instrument and
    obtain appropriate technical and clinical validation before making
    any medical-use claims.
-   The project files should be kept together in the GitHub repository.
    Update this document if the pin mapping, clock configuration, serial
    format, or hardware changes.

## 11. Suggested Repository Structure

``` text
Smart-Pulse-Oximeter/
├── README.md
├── Working-Steps.md
├── Firmware/
│   └── main.c
└── Web-Dashboard/
    └── index.html
```

Adjust the folder names to match the actual files in your repository.
