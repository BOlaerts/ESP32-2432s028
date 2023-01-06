# esp32-2432s028
## Introduction
This repository contains some experimental setup for using the esp32-2432s028 develoment board with tft display within Home Assistant.
Only a few gpio pins are exposed and usable, but by using I²C you could use a MCP23017 module to add additional ports.

## Hardware
### Wiring esp32-2432s028 + BME280 sensor
<img src="/../main/Pictures/esp32_2432s028_i2c.jpg" width="40%" alt= "Schematic" height="40%">

## Software
### SPI
Two separate SPI busses are being used for the display and the touchscreen.
``` yaml
spi:
  - id: my_display
    clk_pin: GPIO14
    mosi_pin: GPIO13
    miso_pin: GPIO12
  - id: my_touch
    clk_pin: GPIO25
    mosi_pin: GPIO32
    miso_pin: GPIO39
```
### I2C
The default gpio pins for I2C on an ESP32 are GPIO21 and GPIO22. Both of them are exposed in the extended IO connector (P3), but GPIO21 is also used as a led pin, so it looks to be useless. GPIO35 which is also exposed in the extended IO connector can only be used for input, so also useless for I2C. Luckily there is another connector with GPIO27, which makes it possible to setup an I2C bus.
``` yaml
i2c:
  sda: GPIO27
  scl: GPIO22
  scan: true
  id: bus_a
```
