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
spi:
  - id: my_display
    clk_pin: GPIO14
    mosi_pin: GPIO13
    miso_pin: GPIO12
  - id: my_touch
    clk_pin: GPIO25
    mosi_pin: GPIO32
    miso_pin: GPIO39
