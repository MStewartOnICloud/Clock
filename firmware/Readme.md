## Firmware

The **ESP-IDF** framework will be the basis of the firmware development. This will be used either natively or as part of **PlatformIO**, whichever delivers the most efficient code based on space and speed. (Similarly the programming language will be either C or C++ based on the same criteria.)

The Arduino framework maybe considered as a fallback.

The firmware will have at least two components:
1. **Web Server**: The web server will provide two functions: [For full details see Web Readme.md`](../web/Readme.md)
   1. **Wi-Fi Manager**: The Wi-Fi manager will provide a web page to allow the user to configure the Wi-Fi connection. This will be used when the device is first powered on and when the Wi-Fi connection is lost.
   2. **Clock Config**: The clock config will provide a web page to allow the user to control the clock functions.
2. **Clock control**: The clock control will manage the LED ring and other hardware components to display the time and respond to user inputs.
   1. Periodically (and on first use) use the Wi-Fi to read the current time based on ip-address (or configured time-zone). Set/update the RTC if required. Adjust for daylight savings, if necessary.
   2. Read temperature sensor and update colour to be displayed to indicate if temperature is "too hot", "too cold" or "just right".
   3. Update the display LEDs to display current time. 