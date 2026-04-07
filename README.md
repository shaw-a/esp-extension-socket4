# Overview

project to convert a 4-socket 240v extension lead to 4x smart socket 

* extension lead came with manual switches for each socket - preserved as OFF override
* extension lead came with 4x usb port and so has 240-5v power converter which is salvaged to power esp (via 5-3.3v converter) and 4x relay module
* esp8266-01s
* home assistant and esp home

possibly don't need to program anything, just configure yamls for ESPHome and run installer tool

pain point with likely be getting physical USB connection up and running

test setup breadboard should verify pinout is working (LEDs to verify, relay control signal is active low)

## ESP8266-01S microcontroller

[esp8266-01s](https://makershop.ie/ESP-01S)

or ESP8266 ESP-01 

[pretty sure I used this instructables before for inputs](https://www.instructables.com/How-to-use-the-ESP8266-01-pins/)

[this instructables is new and looks handy](https://www.instructables.com/Getting-Started-With-the-ESP8266-ESP-01/)

[I remember using this tutorial as a pinout reference](https://randomnerdtutorials.com/esp8266-pinout-reference-gpios/)

## home assistant

[home assistant Getting Started guide](https://www.home-assistant.io/installation/)

* home assistant runs on a Pi or VM or container somewhere and hosts "integrations" and "devices" etc
* examples include 
    - sensors
    - switches
    - workday based automation
    - location based automation

## ESP home

[ESPhome is a Home Assistant integration](https://www.home-assistant.io/integrations/esphome/)

[project of Open Home Foundation](https://www.openhomefoundation.org/projects/)

[ESPHome Getting Started Guide](https://esphome.io/guides/getting_started_hassio/)

* create devices through home assistant
* edit yaml files to configure switches and sensors

