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
  - id: lcd
    clk_pin: GPIO14
    mosi_pin: GPIO13
    miso_pin: GPIO12
  - id: touch
    clk_pin: GPIO25
    mosi_pin: GPIO32
    miso_pin: GPIO39
```
#### Display
The esp32-2432s028 has an ili9341 display that can be used via ther LCD SPI bus:
``` yaml
display:
  - platform: ili9341
    model: TFT 2.4
    spi_id: lcd
    cs_pin: 15
    dc_pin: 2
    lambda: |-
      ...
```
#### Touchscreen
To be able to use the touchscreen interface, a touchscreen 
The esp32-2432s028 has an ili9341 display that can be used via ther LCD SPI bus:
``` yaml
touchscreen:
  platform: xpt2046
  spi_id: touch
  cs_pin: 33
  interrupt_pin: 36
  update_interval: 50ms
  report_interval: 1s
  threshold: 400
  calibration_x_min: 3860
  calibration_x_max: 280
  calibration_y_min: 340
  calibration_y_max: 3860
  swap_x_y: false

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
