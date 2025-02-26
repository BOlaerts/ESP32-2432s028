# ESP32-2432s028
## Introduction
This repository contains some experimental setup for using the esp32-2432s028 develoment board with tft display within Home Assistant.
Only a few gpio pins are exposed and usable, but by using I²C you could use a MCP23017 module to add additional ports.

## Hardware
### Wiring esp32-2432s028 + BME280 sensor
All credits for extended pinout explanation go to macsbug, see more on: https://macsbug.wordpress.com/2022/08/17/esp32-2432s028/

<img src="/../main/Pictures/esp32_2432s028_i2c.jpg" width="40%" alt= "Schematic" height="40%">

Flashing instructions: keep the boot button pressed when you plug in the USB cable/flasher, this will allow to flash the firmware.

Remark: some boards have IO22 next to IO27 on CN1, so a single plug and cable can be used.
[Credits to Dave W](https://github.com/dbuggz)

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
The esp32-2432s028 has an ili9341 display that can be used via the lcd SPI bus:
``` yaml
display:
  - platform: ili9xxx
    model: ILI9341
    spi_id: lcd
    cs_pin: 15
    dc_pin: 2
    lambda: |-
      ...

# Former led pin in the display config
output:
  - platform: ledc
    pin: 21
    id: former_led_pin

# Define a monochromatic, dimmable light for the backlight
light:
  - platform: monochromatic
    output: former_led_pin
    name: "Display Backlight"
    id: back_light
    restore_mode: ALWAYS_ON
```
#### Touchscreen
To add touch functionalities, a touchscreen component should be added via the touch SPI bus:
``` yaml
touchscreen:
  platform: xpt2046
  spi_id: touch
  cs_pin: 33
  interrupt_pin: 36
  update_interval: 50ms
  threshold: 400
  calibration:
    x_min: 280
    x_max: 3860
    y_min: 340
    y_max: 3860
  transform:
    mirror_x: true
    mirror_y: false
    swap_xy: false

```

### I2C
The default gpio pins for I2C on an ESP32 are GPIO21 and GPIO22. Both of them are exposed in the extended IO connector (P3), but apparently GPIO21 is also used as a led pin in the lcd SPI bus, so it looks as if this gpio pin is somehow useless on the extended IO connector. GPIO35 which is also exposed in the extended IO connector can only be used for input, so also useless for I2C. Luckily there is another connector with GPIO27, which makes it possible to setup an I2C bus.
``` yaml
i2c:
  sda: GPIO27
  scl: GPIO22
  scan: true
  id: bus_a
```
### Force LED on back of module to stay off
``` yaml
# Define pins for backlight display and back LED1
output:
  - platform: ledc
    pin: GPIO21
    id: former_led_pin
  - platform: ledc
    id: output_red
    pin: GPIO4
    inverted: true
  - platform: ledc
    id: output_green
    pin: GPIO16
    inverted: true
  - platform: ledc
    id: output_blue
    pin: GPIO17
    inverted: true

# Define a monochromatic, dimmable light for the backlight
light:
  - platform: monochromatic
    output: former_led_pin
    name: "Display Backlight"
    id: back_light
    restore_mode: ALWAYS_ON
  - platform: rgb
    name: LED
    red: output_red
    id: led
    green: output_green
    blue: output_blue
    restore_mode: ALWAYS_OFF
```
### Demo
#### YAML
``` yaml
substitutions:
  devicename: "esp32-2432s028"
  ssid: Esp32-2432S028 Fallback Hotspot
  static_ip: 192.168.0.44
  gateway: 192.168.0.1
  subnet: 255.255.255.0

packages:
  wifi: !include common/wifi.yaml
  
esphome:
  name: $devicename
  build_path: ./build/$devicename

esp32:
  board: esp32dev
  framework:
    type: arduino

logger:

api:

ota:
  - platform: esphome

captive_portal:

spi:
  - id: lcd
    clk_pin: GPIO14
    mosi_pin: GPIO13
    miso_pin: GPIO12
  - id: touch
    clk_pin: GPIO25
    mosi_pin: GPIO32
    miso_pin: GPIO39

i2c:
  sda: GPIO27
  scl: GPIO22
  scan: true
  id: bus_a

color:
  - id: my_red
    red: 100%
    green: 0%
    blue: 0%
  - id: my_orange
    red: 100%
    green: 50%
    blue: 0%
  - id: my_yellow
    red: 100%
    green: 100%
    blue: 0%
  - id: my_green
    red: 0%
    green: 100%
    blue: 0%
  - id: my_blue
    red: 0%
    green: 0%
    blue: 100%
  - id: my_teal
    red: 0%
    green: 100%
    blue: 100%
  - id: my_gray
    red: 70%
    green: 70%
    blue: 70%
  - id: my_white
    red: 100%
    green: 100%
    blue: 100%

font:
  - file: "Helvetica.ttf"
    id: helvetica_48
    size: 48
  - file: "Helvetica.ttf"
    id: helvetica_36
    size: 36
  - file: "Helvetica.ttf"
    id: helvetica_24
    size: 24
  - file: "Helvetica.ttf"
    id: helvetica_12
    size: 12

image:
  - file: "power-off-button.png"
    id: on_off_button
  - file: "radio-station.png"
    id: radio

binary_sensor:
  - platform: status
    name: "Node Status"
    id: system_status

  - platform: touchscreen
    name: Power
    x_min: 30
    x_max: 90
    y_min: 20
    y_max: 80

  - platform: touchscreen
    name: Music
    x_min: 150
    x_max: 210
    y_min: 20
    y_max: 80

time:
  - platform: homeassistant
    id: esptime

text_sensor:
  - platform: template
    # name: Uptime Human Readable
    id: uptime_human
    icon: mdi:clock-start
    internal: True

  - platform: wifi_info
    ip_address:
      # name: ESP IP Address
      id: ip_address

sensor:
  - platform: uptime
    # name: Uptime Sensor
    internal: True
    id: uptime_sensor
    update_interval: 1s
    on_raw_value:
      then:
        - text_sensor.template.publish:
            id: uptime_human
            state: !lambda |-
              int seconds = round(id(uptime_sensor).raw_state);
              int days = seconds / (24 * 3600);
              seconds = seconds % (24 * 3600);
              int hours = seconds / 3600;
              seconds = seconds % 3600;
              int minutes = seconds /  60;
              seconds = seconds % 60;
              return (
                ("Uptime ") +
                (days ? to_string(days) + "d " : "") +
                (hours ? to_string(hours) + "h " : "") +
                (minutes ? to_string(minutes) + "m " : "") +
                (to_string(seconds) + "s")
              ).c_str();

  - platform: wifi_signal
    # name: "WiFi Signal Sensor"
    internal: True
    id: wifi_signal_sensor
    update_interval: 5s

  - platform: bmp280_i2c
    temperature:
      name: "Temperatuur"
      unit_of_measurement: °C
      accuracy_decimals: 1
      id: "bme_temperature"
    pressure:
      name: "Luchtdruk"
      unit_of_measurement: hPa
      accuracy_decimals: 0
      id: "bme_humidity"
    address: 0x76
    update_interval: 30s        

display:
  - platform: ili9xxx
    model: ILI9341
    spi_id: lcd
    cs_pin: 15
    dc_pin: 2
    invert_colors: false
    lambda: |-
      int hs = it.get_width() / 2; // Horizontal Spacing = text data horizontal center point
      int hq = it.get_width() / 4; // text data horizontal center for two vertical lines
      int vs = it.get_height() / 8; // Vertical Center = text data vertical center point = how many lines
      int line_gap = 21; // distance of line from center of data text
      it.rectangle(0,  0, it.get_width(), it.get_height(), id(my_blue));
      it.rectangle(0, 20, it.get_width(), it.get_height(), id(my_blue));
      
      it.strftime(5, 5, id(helvetica_12), id(my_white), TextAlign::TOP_LEFT, "%H:%M:%S", id(esptime).now());
      it.print(hs, 5, id(helvetica_12), id(my_blue), TextAlign::TOP_CENTER, "${devicename}"); //print title  
      
      if (id(system_status).state) {
        it.print(it.get_width()-5, 5, id(helvetica_12), id(my_green), TextAlign::TOP_RIGHT, "Online");
      }
      else {
        it.print(it.get_width()-5, 5, id(helvetica_12), id(my_red), TextAlign::TOP_RIGHT, "Offline");
      }
      it.line(0, it.get_height()-20, it.get_width(), it.get_height()-20, id(my_blue)); // line across bottom above footer text
      it.printf(5, it.get_height()-3, id(helvetica_12), id(my_gray), TextAlign::BOTTOM_LEFT, "%s", id(uptime_human).state.c_str());
      it.printf(hs, it.get_height()-3, id(helvetica_12), id(my_gray), TextAlign::BOTTOM_CENTER, "%.0fdBm", id(wifi_signal_sensor).state);
      it.printf(it.get_width()-5, it.get_height()-3, id(helvetica_12), id(my_gray), TextAlign::BOTTOM_RIGHT, "%s", id(ip_address).state.c_str());
      it.image(hq - 25, 50, id(on_off_button));
      it.image(3*hq - 25, 50, id(radio));

      it.line(0, vs * 3, it.get_width(), vs * 3, id(my_blue));
      it.print(hs, vs * 3 + line_gap, id(helvetica_24), id(my_white), TextAlign::CENTER, "BME280 sensor");
      it.line(0, vs * 4, it.get_width(), vs * 4, id(my_blue));
      it.print(hq, vs * 5, id(helvetica_12), id(my_white), TextAlign::CENTER, "Temperature:");
      it.printf(3*hq, vs * 5, id(helvetica_12), id(my_white), TextAlign::CENTER, "%.1f°C", id(bme_temperature).state);
      it.print(hq, vs * 6, id(helvetica_12), id(my_white), TextAlign::CENTER, "Pressure:");
      it.printf(3*hq, vs * 6, id(helvetica_12), id(my_white), TextAlign::CENTER, "%.0fhPa", id(bme_humidity).state);

# Define pins for backlight display and back LED1
output:
  - platform: ledc
    pin: GPIO21
    id: former_led_pin
  - platform: ledc
    id: output_red
    pin: GPIO4
    inverted: true
  - platform: ledc
    id: output_green
    pin: GPIO16
    inverted: true
  - platform: ledc
    id: output_blue
    pin: GPIO17
    inverted: true

# Define a monochromatic, dimmable light for the backlight
light:
  - platform: monochromatic
    output: former_led_pin
    name: "Display Backlight"
    id: back_light
    restore_mode: ALWAYS_ON
  - platform: rgb
    name: LED
    red: output_red
    id: led
    green: output_green
    blue: output_blue
    restore_mode: ALWAYS_OFF

touchscreen:
  platform: xpt2046
  spi_id: touch
  cs_pin: 33
  interrupt_pin: 36
  update_interval: 50ms
  report_interval: 1s
  threshold: 400
  calibration:
    x_min: 280
    x_max: 3860
    y_min: 340
    y_max: 3860
  transform:
    mirror_x: true
    mirror_y: false
    swap_xy: false

# Exposed switches.
switch:
  - platform: restart
    name: ESP32-2432S028 Restart
```
#### Screenshot
<img src="/../main/Pictures/esp32_2432s028_demo.jpg" width="40%" alt= "Schematic" height="40%">

#### Home Assistant integration
<img src="/../main/Pictures/esp32_2432s028_home_assistant.jpg" width="80%" alt= "Schematic" height="80%">

### Enclosure options
- https://www.thingiverse.com/thing:5680106
- https://www.thingiverse.com/thing:5905855
 
### Update 19.07.2023: attempt to make a Home Assistant remote control
Using one of the above enclosure options, a 600 MAh 3.7V battery and a MT3608 step-up convertor, I tried to make a Home Assistant remote control.
Conclusion: it works, but unfortunately I didn't manage to keep it working for an acceptable period of time, even with deep sleep implemented.

<img src="/../main/Pictures/esp32_2432s028_remote.jpg" width="40%" alt= "Remote control" height="40%">

<img src="/../main/Pictures/esp32_2432s028_remote_open.jpg" width="40%" alt= "Remote control under the hood" height="40%">
