# PicoPD
Pico PD is a development board for USB Power Delivery development with Raspberry Pi Pico footprint. The board can help you negotiate voltage from USB PD 2.0, 3.0, and 3.1 with PPS (programable power supply) up to 5A of current at 20V max. The board feature input current/voltage/temperture reading as well as power switch to cut of the current to external devices.

# Quick links
[AP33772 Library](https://github.com/CentyLab/AP33772-Cpp)

[PicoPD Pinout](#picopd-pinout)

[Setup for AP33772 objects](#setup-for-ap33772-objects)

[Basic AP33772 functions](#basic-ap33772-functions)

[Important note](#important-note)

[Arduino IDE setup](#arduino-ide-setup)

[VSCode PlatformIO setup](#vscode-platformio-setup)

[Enable Serial print for debugging](#enable-serial-debug)

# Pin out
![pinout](https://github.com/CentyLab/PicoPD/blob/main/Hardware/PicoPD%20Pin%20Out.png?raw=true)


# Setup for AP33772 objects
To use AP33772 IC with the library, first you need to create a `usbpd` object. Then set up your I2C pins and start collecting the power profiles from your charger. Ensure your delay after `Wire.begin()` is more than 500ms for stable PDOs request. Enable pin 23 to allow VBUS pass-through.
```
AP33772 usbpd; // Automatically wire to Wire0

void setup()
{
  Wire.setSDA(0); // Set the correct SDA0 pin
  Wire.setSCL(1); // Set the correct SCL0 pin
  Wire.begin();
  Serial.begin(115200);
  delay(1000); 		// Delay is needed for the IC to obtain PDOs
  usbpd.begin(); 	// Start pulling the PDOs from power supply

  pinMode(25, OUTPUT); // Built in LED
  pinMode(23, OUTPUT); // Load Switch
  digitalWrite(23, HIGH); // Turn on VBUS pin

  usbpd.printPDO(); //Print out PDO to Serial port
}
```

# Basic AP33772 functions
Common functions are:
```
setVoltage(int targetVoltage)       //Set voltage, unit in mV, read libary for logic flow
setMaxCurrent(int targetMaxCurrent) //Allow current limit if source has that capability
readVoltage()                       //Return voltage in mV
readCurrent()                       //Return current in mA
printPDO()                          //Print out PDOs in Serial
```

Set over tempearture protection
```
usbpd.setOTPTHR(100);               //Turn off output if NTC detect 100C or above
usbpd.setMask(OTP_EN);              //Enable Over Temp Protection flag
```

# Important note
After flashing, the latest AP33772-CPP has been updated to auto request PDO after bootup without reconnecting the charger. Thank to @Patronics contribution.

Calling `usbpd.printPDO()` without `Serial.begin()` can soft-brick your PicoPD. Causing the IC to be non-recognizable by your computer. You will have to use PicoProbe to reflash the chip. Check out @Patronics [PicoProbePCB](https://github.com/Patronics/PicoProbePCB) if you would like to make a stable fixture for firmware debugging.

# Enable Serial Debug

To enable Serial debug, there are 3 main methods:

**Using USB-C Power/Data splitter (easiest):** Use a USB-C Splitter that separates the USB 2.0 signal into another USB-C port. You only need USB 2.0 FullSpeed (12Mbps) for the job, but never hurts to get extra speed if needed later. 

* [USB-C Power/Data Splitter](https://lectronz.com/products/usb-c-splitter) (USB-HighSpeed)
* [USB-C PD Power Splitter](https://www.tindie.com/products/essenceeng/usb-c-pd-power-splitter/) (USB-HighSpeed)


**Using the provided solder pads at the back:** Desolder the front "USB Jumper" resistors. They are just 2x 0 Ohm resistors that bridge between the RP2040 IC and the USB-C port. Then use a [USB Type A Plug Breakout Cable](https://www.adafruit.com/product/4448?gclid=Cj0KCQjwoK2mBhDzARIsADGbjeo7nigl8f-8mf8ojsnPvi8TiROBC7YXmMKpGVau8VJHeIO8NyU5nncaAs9qEALw_wcB), or make your own from any USB cable and connect to the existing pad at the back of PicoPD


<p align="center">
  <img width="360" height="330" src="https://cdn.hackaday.io/images/5618501705180911409.png">
</p>
<p align="center">
  <img width="360" height="360" src="https://cdn.hackaday.io/images/2298711708838132236.png">
</p>

**Using UART communication**: Make sure you are not using GP0 and GP1 as they are reserved for I2C0 channel for AP33772 IC. Check out Youtube tutorial: [Talk to Your Pico Over Serial](https://www.youtube.com/watch?v=pbWhoJdYA1s)

# Arduino IDE setup
Open Arduino IDE, select File -> Preference
![Preference](https://github.com/CentyLab/PicoPD/blob/main/Documentation/doc1.png?raw=true)

Copy this line below and place in "Aditional Boards Manager URLs" -> OK. This will allow you to pull the Earlephilhower Pico core that work with the AP33772 library.

```
https://github.com/earlephilhower/arduino-pico/releases/download/global/package_rp2040_index.json
```
![boardmanagerURL](https://github.com/CentyLab/PicoPD/blob/main/Documentation/doc2.png?raw=true)

From Board Manager, search for "Pico" and install the "Raspberry Pi Pico/RP2040" core by Earlephilhower

![boardmanager](https://github.com/CentyLab/PicoPD/blob/main/Documentation/doc3.png?raw=true)

Now from Tool -> Board -> Raspberry Pi Pico/RP2040 -> Raspberry Pi Pico

![boardmanager](https://github.com/CentyLab/PicoPD/blob/main/Documentation/doc4.png?raw=true)

Now download the latest AP33772 library as a Zip file

```
https://github.com/CentyLab/AP33772-Cpp
```
![ap33772lib](https://github.com/CentyLab/PicoPD/blob/main/Documentation/doc5.png?raw=true)

Import the AP33772 library into your Arduino IDE. Sketch -> Include library -> Add .ZIP Library ...

![importlib](https://github.com/CentyLab/PicoPD/blob/main/Documentation/doc6.png?raw=true)

Now you start using the Example code in File -> Examples -> AP33772-Cpp-main. The example is ready to compile and flash on to your PicoPD.
![importlib](https://github.com/CentyLab/PicoPD/blob/main/Documentation/doc8.png?raw=true)

Now you can plug in your USB-C cable with Power Delivery 2.0/3.0/3.1. If you also like to have serial print out, check out [Enable Serial print for debugging](#enable-serial-debug).

# VSCode PlatformIO setup
After [VSCode](https://code.visualstudio.com/) installation, you will need to install PlatformIO extension.

![installplatform](https://github.com/CentyLab/PicoPD/blob/main/Documentation/doc9.png?raw=true)

Now create a new PlatformIO project
+ Name:WhatEverYouWantToCallit
+ Board: Pico (RasberryPi)
+ Framework: Arduino (default)

Click "Finish"

![newproject](https://github.com/CentyLab/PicoPD/blob/main/Documentation/doc10.png?raw=true)

![boardandframework](https://github.com/CentyLab/PicoPD/blob/main/Documentation/doc11.png?raw=true)

In your new project, copy this and replace everything in platformio.ini. This is how you ask PlatformIO to compile using Earlephilhower Pico core, and enable upload via USB.


```
[env:pico]
platform = https://github.com/maxgerhardt/platform-raspberrypi.git
board = pico
framework = arduino
board_build.core = earlephilhower

;; If directly upload from PC
upload_protocol = picotool
```

Now download the latest AP33772 library as a Zip file. Unzip and place the folder in `lib`

```
https://github.com/CentyLab/AP33772-Cpp
```

![ap33772lib](https://github.com/CentyLab/PicoPD/blob/main/Documentation/doc5.png?raw=true)

Your lib folder should look like this

![libstructur](https://github.com/CentyLab/PicoPD/blob/main/Documentation/doc12.png?raw=true)

To try out example code, you copy the content inside `lib/Compiled for PicoPD/PPSCycling/PPSCycling.ino`, And replace everything in your `src/main.cpp`

![maincpp](https://github.com/CentyLab/PicoPD/blob/main/Documentation/doc13.png?raw=true)

Now you can plug the PicoPD to your computer via USB-C and hit the upload button. After this, you will have to connect a charger to PicoPD to see the voltage cycling example. If you also like to have serial print out, check out [Enable Serial print for debugging](#enable-serial-debug).

![maincpp](https://github.com/CentyLab/PicoPD/blob/main/Documentation/doc14.png?raw=true)