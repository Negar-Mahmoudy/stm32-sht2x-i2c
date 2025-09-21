# stm32-SHT20-I2C-temperature measurement

This project demonstrates how to interface an **STM32 microcontroller** with the **SHT2x temperature & humidity sensor** using the **I²C protocol**.  
In this example, the temperature is read from the sensor and printed via **UART** to a serial terminal.

---

## Overview

The SHT2x sensor provides temperature and humidity data through I²C commands.  
The general communication flow is:

1. Send the measurement command (e.g., `Trigger T measurement`).  
2. Receive 3 bytes from the sensor:  
   - **MSB** (most significant byte of data)  
   - **LSB** (least significant byte of data)  
   - **Checksum (CRC)**  
3. Convert the raw data into real temperature values using the datasheet formula.

---

## Sensor Commands

| Command                  | Comment       | Code (bin)  | Code (hex) |
|---------------------------|---------------|-------------|------------|
| Trigger T measurement     | hold master   | `1110 0011` | `0xE3`     |
| Trigger RH measurement    | hold master   | `1110 0101` | `0xE5`     |
| Trigger T measurement     | no hold       | `1111 0011` | `0xF3`     |
| Trigger RH measurement    | no hold       | `1111 0101` | `0xF5`     |
| Write user register       | —             | `1110 0110` | `0xE6`     |
| Read user register        | —             | `1110 0111` | `0xE7`     |
| Soft reset                | —             | `1111 1110` | `0xFE`     |

> In this project, only **`0xE3` (Trigger T measurement, hold master)** is used.

---

## Communication Sequence (Hold Master)

According to the datasheet, the **Hold Master** measurement sequence is:

1. Send sensor address + write bit.  
2. Send the measurement command (`0xE3`).  
3. Send sensor address + read bit.  
4. Receive 3 bytes: MSB, LSB, and CRC.  

![Hold master communication sequence](https://github.com/Negar-Mahmoudy/stm32-sht2x-i2c/blob/main/images/2.png?raw=true)

![Temperature Formula](https://github.com/Negar-Mahmoudy/stm32-sht2x-i2c/blob/main/images/3.png?raw=true)

---

## Temperature Conversion Formula

The raw sensor output (`ST`) is converted to °C using:

\[
T = -46.85 + 175.72 \cdot \frac{S_T}{2^{16}}
\]

Implemented in code as:

```c
temp = -46.85 + 175.72 * ((((uint16_t)(receive_array[0])) << 8) | receive_array[1]) / 65536.0;
````

---

## Main Code Snippet

The key part of the code in the `while(1)` loop:

```c
while (1)
{
    cmd = 0xE3;   // Temperature measurement command
    HAL_I2C_Master_Transmit(&hi2c1, 0x40<<1, &cmd, 1, 100); // Send command
    HAL_I2C_Master_Receive(&hi2c1, 0x40<<1, receive_array, 3, 100); // Receive data
    temp = -46.85 + 175.72 * ((((uint16_t)(receive_array[0])) << 8) | receive_array[1]) / 65536.0; // Convert to °C
    HAL_Delay(1000);

    printf("temperature = %f\r\n", temp); // Print result via UART
}
```

---

## Configuration

* **I²C1**

  * Speed: `100 kHz`
  * Mode: `7-bit addressing`
  * Sensor address: `0x40`

* **UART2**

  * Baudrate: `115200`
  * Mode: `TX/RX`

---
