---
title: "Duckuino"
date: 2020-01-08T14:42:08+02:00
draft: false
---
![](/images/duckuino/duck.jpeg)
**It came to my mind to make a rubber ducky using arduino uno which i already have .. so let's get started.**
* Rubber Ducky is used to perform HID attacks >> search it out.

Arduino Uno has a chip "Atmegau16u2" or "8u2", But mine is Arduino Uno R3 with Atmega16u2 chip.
* The ATmega16U2 chip on your Arduino board acts as a bridge between the computer's USB port and the main processor's serial port.
# Steps:
### 1. **We need to install dfu-programmer:**
* [About dfu](https://dfu-programmer.github.io/)
* for linux users:
```yml
sudo apt-get install dfu-programmer
```

### 2. **Download hid keyboard library from [here](https://github.com/SFE-Chris/UNO-HIDKeyboard-Library):**
* Download zip file and extract the file.
* Open the directory and copy (hidkeyboard.ccp and hidkeyboard.h) 
* then go to the directory where you downloaded the Arduino IDE (where the library files are) 
* Open 'path'/hardware/arduino/avr/arduino/ 
* And paste them there.

### 3. **Download the [firmware](https://github.com/unkwneuser/aurdino-uno-as-hid) to be flashed to the Arduino:**
* Download zip file and extract it
* Open directory you will find (Arduino-usbserial-uno.hex and Arduino-keyboard-0.3.hex and flash)
* ***note:*** "flash" is the script to run these files with the dfu-programmer we've just installed
* Open the flash script and check for your device name "your chip name" (maybe something different than Atmega16u2), and you will find it written on the chip or open IDE>tools>Get board info.
* Open terminal in that directory to run the flash script.

* *note: run the script with root privileges

### The script will work on three phases:

* you will reset Arduino and the script will burn “Arduino-usbserial-uno.hex”. 
* you will put your payload with IDE. 
* continue the script to burn “Arduino-keyboard-0.3.hex”

* ***note:*** after every phase you will unplug and plug the arduino from your PC.
---
## In Detail:
![](/images/duckuino/flash.png)
1. It will ask you to reset the Atmega16u2:
briefly bridge the reset pin with the ground. The pins are located near the USB connector, as shown in this picture.connect them with wire or any metal briefly.
![](/images/duckuino/pins.png)
* ***To verify you have reset the 8u2 or 16u2 chip:*** 
In the IDE: check the list of serial ports. The serial port for your board should no longer show up.
also a led will blink when connecting the pins.
2. Press enter (and leave the terminal open with the flash script running)
3. Unplug and plug the Arduino (then put your code/payload using the IDE and upload it as usual).
4. **In terminal:** Press enter again (the script will ask for Arduino reset again)
* so reset it as done before then press enter again it will be done.
* you will get two validatings as the picture.
### **Now** unplug and plug the Arduino and it will do whatever the payload is for.
---
## Payload Example:
* will write hello world 10 times in an opened text editor
```java
// note that these two sentences below are what we had put in Arduino's library for hid keyboard   
#include <HIDKeyboard.h>
HIDKeyboard Keyboard;

void setup() {
Keyboard.begin(); // Initializes keyboard
} void loop() {
for(int i=1;i<=10;i++){
Keyboard.println("Hello World!"); // Types "Hello World!"
Keyboard.pressSpecialKey(ENTER); // Sends an "Enter" keypress
Keyboard.releaseKey(); // Releases "Enter"
}
while(1); // Hold forever
}
```
---
* Search about Rubber Ducky payloads you will find a lot, but you need to convert them into Arduino code.
* And here is a [script](https://github.com/kimocoder/rtl8188eus) will convert them for you or this other one [online](https://dukweeno.github.io/Duckuino/).

### References:
* https://www.youtube.com/watch?v=hry8t-gEqy8
* http://mitchtech.net/arduino-usb-hid-keyboard/
* https://www.arduino.cc/en/Hacking/DFUProgramming8U2
* https://www.youtube.com/watch?v=Tj9FpaRPmUw&t=603s