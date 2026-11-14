## LED-ring Clock with Wi-Fi and browser based front-end.

This project is to build a digital clock with an "analogue" display based on a ring of 60 RGB leds. The clock will use varying colours to indicate the "hour" and "minute" "hands". There will be hour markers with the makers at the 12, 3, 6, and 9 hour positions will be used to indicate the temperature as "too hot", "too cold" or "just right" by using different colours.

### Hardware

The project will be based on (currently available):

- **Processor**: ESP-32 (currently **SparkFun Thing Plus - ESP32 WROOM (Micro-B)**)
- **LED Ring**: equivalent to 4x **NeoPixel 1/4 60 Ring - 5050 RGB LED w/ Integrated Drivers**
- **RTC**: **ChronoDot V3 High Precision RTC**
- **Temperature Sensor**: **DS18B20+ One Wire Digital Temperature Sensor**

(everything in the above list is open to change if newer/better hardware is available)

#### Firmware

1. **Web Server**: Provides **Wi-Fi Manager** and **Clock Config Management**
2. **Clock Control**: Manages the LED ring and other hardware components to display the time and temeperature.

[See Firmware Readme](./firmware/Readme.md)
