# STM32 Countdown Stopwatch  
Millisecond-Resolution Timer with SSD1306 OLED (I2C)

---

## Overview

This project implements a **countdown stopwatch** using an STM32 microcontroller and an SSD1306 128x64 OLED display over I2C.

The system features:

- Minute-based time setting (10 minutes maximum)
- Start
- Pause / Resume
- Reset
- Millisecond precision timing (hardware timer interrupt)
- State-machine driven architecture
- Clean display refresh logic

The project is developed using **STM32CubeIDE** and **HAL drivers**.

---

## Hardware

### Microcontroller Board

- STM32 Nucleo F446RE

### Other Peripherals

- 3 Tach Buttons (4 Pins)
- SSD1306 OLED Screen (0.96")
---

### Display

| Component | Specification |
|------------|--------------|
| Display IC | SSD1306 |
| Resolution | 128x64 |
| Interface | I2C |
| Voltage | 3.3V |

---

### Button Connections

All buttons are:

- Configured as `GPIO_MODE_INPUT`
- Internal `GPIO_PULLUP` enabled
- Active LOW (pressed = GND)

| Function        | MCU Pin | Mode | Pull | Active Level |
|---------------|---------|------|------|--------------|
| Time Set       | PA0 | Input | Pull-up | LOW |
| Start / Reset  | PA1 | Input | Pull-up | LOW |
| Pause / Resume | PA10 | Input | Pull-up | LOW |

---

### I2C Configuration

| Parameter | Value |
|------------|-------|
| Peripheral | I2C1 |
| Clock Speed | 100 kHz |
| Addressing Mode | 7-bit |
| Duty Cycle | 2 |

Default pin mapping:

| Signal | Pin |
|--------|------|
| SCL | PB8 |
| SDA | PB9 |

<img width="1152" height="986" alt="image" src="https://github.com/user-attachments/assets/8afa4888-0b41-4c3c-8e3d-828f1c9b2b85" />


---

### Timer Configuration (TIM2)

Used for 1 ms interrupt generation.

| Parameter | Value |
|------------|-------|
| Prescaler | 8399 |
| Period | 9 |
| Counter Mode | Up |
| Clock Division | 1 |
| Interrupt | Enabled |

This produces a **1 millisecond interrupt period**.

---

## Software Architecture

### State Machine

```c
typedef enum {
    STATE_IDLE,
    STATE_RUNNING,
    STATE_PAUSED,
    STATE_FINISHED
} State_t;
```

| State    | Description              |
| -------- | ------------------------ |
| IDLE     | Waiting for time setting |
| RUNNING  | Countdown active         |
| PAUSED   | Countdown frozen         |
| FINISHED | Countdown reached zero   |

### Button Logic

| Method    | Value         |
| --------- | ------------- |
| Type      | Software      |
| Duration  | 50 ms         |
| Mechanism | HAL_GetTick() |

### Button Behavior
#### Time Set (PA0)
| Condition      | Action                             |
| -------------- | ---------------------------------- |
| STATE_IDLE     | Increase minutes                   |
| STATE_FINISHED | Reset to IDLE and increase minutes |
| Max Minutes    | 10                                 |

#### Start / Reset (PA1)
| Condition       | Action                       |
| --------------- | ---------------------------- |
| IDLE & time > 0 | Start countdown              |
| Any other state | Reset to IDLE and clear time |

#### Pause (PA10)
| Current State | New State |
| ------------- | --------- |
| RUNNING       | PAUSED    |
| PAUSED        | RUNNING   |
| Others        | No effect |

---

## Build & Flash
### Requirements
- STM32CubeIDE
- ST-Link
- STM32CubeProgrammer (optional)

### Steps
1. Open project in STM32CubeIDE
2. Build
3. Flash via ST-Link
> If unexpected GPIO behavior occurs: Use STM32CubeProgrammer → Full Chip Erase (Mass Erase). Then re-flash firmware.

---

## Libraries Used
* **STM32 HAL Drivers** (via STM32CubeIDE)
* **ssd1306.h / ssd1306_fonts.h** ([SSD1306 OLED display library](https://github.com/afiskon/stm32-ssd1306/tree/master).)
* **stdio.h**, **stdlib.h** (Standard C libraries)

---

## Others

This section contains a general overview of the project on a breadboard and a sample video of the project in action.

<img width="1600" height="836" alt="image" src="https://github.com/user-attachments/assets/97ed8e6e-c404-43b4-9c70-878716679cb9" />

https://github.com/user-attachments/assets/b3c972bd-b1fd-41f7-b6a5-c2a814d2e757


