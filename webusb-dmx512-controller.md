# WebUSB DMX512 Controller

TK webusb_dmx512_controller.jpg
![WebUSB DMX512 controller](images/webusb_dmx512_controller.jpg)

I love to control all the lights and I always try to find new ways of doing so. Currently I'm in love with DMX512 and WebUSB.

This article describes how you can build your own DMX512 controller using an Arduino and how this controller can be used directly in the browser by leveraging WebUSB.


TK Do more references and ToC

If you already know what DMX512 is and you just want to build your device, jump to

TK Need super good pictures of lights and controller: Ask a photographer


## DMX512

[Wikipedia defines DMX512](https://en.wikipedia.org/wiki/DMX512) like this:

> DMX512 (Digital Multiplex) is a standard for digital communication networks that are commonly used to control stage lighting and effects. It was originally intended as a standardized method for controlling light dimmers, which, prior to DMX512, had employed various incompatible proprietary protocols.

DMX512 is used in many theaters, clubs and festivals to control all the lights and beyond: From static lights to moving heads and lasers to fog machines.

In my basement I have a collection of DMX512 lights:

TK: newyears_eve_2017.jpg
![New Years Eve 2017](images/newyears_eve_2017.jpg)

### How does it work?

DMX512 combines a set of lights (called fixtures) into a [bus network](https://en.wikipedia.org/wiki/Bus_network) of fixtures (called Universe). Each fixture has one ore more functionalities, such as

* RGB (Red, Green, Blue) color
* Dimmer
* UV
* Move the fixture horizontally or vertically
* Blinking effect
* Generate fog
* Blow bubbles

Each functionality can be controlled by one or more channels. This means that each fixture has a different amount of channels, depending which functionality the fixture has. It also means that you can't change the functionality of a fixture, you can only configure which of the predefined set of channels is used.

Each channel can be filled with a value between 0 and 255. In most situations it has the following meaning:

* `0 = 0%: no intensity, zero speed, no blinking position`
* `255 = 100%: full color, full speed, fast blinking`

Every fixture comes with a manual that holds all the informations you need to use the it. It includes a list all predefined channel sets + the meaning of the channel values and general information about the fixture.

#### Basic fixtures

I will describe two of my favorites fixtures to get started with DMX512.

##### Fog Machine

TK Image of fog machine
![Fog machine](images/fog_machine.jpg)

Is producing smoke to improve the visibility of the lights and the overall atmosphere.

```csv
Channel, Functionality, Description
1, Fan Speed, Regulate the speed of the fan to blow out the smoke
```

* 0 = fan is turned off
* 255 = fan is at highest speed

##### Flat PAR

TK Image of Flat PAR
![Flat PAR](images/flat_par.jpg)

Let's take a look at a very basic fixture:
The ["SePar Quad LED RGBUV IR" by Fun Generation](https://www.thomann.de/gb/fun_generation_separ_quad_led_rgb_uv_ir.htm). What does all of this mean?

* SePar = This is a [PAR light](https://en.wikipedia.org/wiki/Parabolic_aluminized_reflector_light) with 5 LEDs
* Quad LED RGBUV = The LEDs can create any RGB color and UV light at the same time
* IR = Fixture can be controlled via infrared

The [Flat PAR's manual](https://images.static-thomann.de/pics/atg/atgdata/document/manual/399604_c_399539_399604_r4_en_online.pdf) is more exciting than to the one of the Fog Machine, because the PAR has 8 channels in 4 different sets (2, 4, 6 or 8):


##### 2 Channels

```csv
Channel, Functionality, Description
1, Color or Program, Set a specific color out of a list of colors or activate a automatic show
2, Special function, Speed of the automatic show or microphone sensitivity
```

##### 4 Channels

```csv
Channel, Functionality, Description
1, Red, Intensity of Red
2, Green, Intensity of Green
3, Blue, Intensity of Blue
4, UV, Intensity of UV
```

##### 6 Channels

```csv
Channel, Functionality, Description
1, Red, Intensity of Red
2, Green, Intensity of Green
3, Blue, Intensity of Blue
4, UV, Intensity of UV
5, Dimmer, Brightness of the LEDs
6, Strobe, Blinking effect
```

##### 8 Channels

```csv
Channel, Functionality, Description
1, Red, Intensity of Red
2, Green, Intensity of Green
3, Blue, Intensity of Blue
4, UV, Intensity of UV
5, Dimmer, Brightness of the LEDs
6, Strobe, Blinking effect
7, Color or Program, Set a specific color out of a list of colors or activate a automatic show
8, Special function, Speed of the automatic show or microphone sensitivity
```

Which set of channels should you use? That depends on what you actually want to do with the fixture. In this case I want to set the RGB color and use the dimmer, so I will use the **6 channels** set.

But where can we set that? Every fixture has some kind of interface that can be used to configure it. In many cases you will find a display and some buttons:

TK flat_par_configure_channel_set_using_display.jpg
![Flat PAR configure channel_set using display](images/flat_par_configure_channel_set_using_display.jpg)


This display let's you change the configuration of the fixture, for example the set of channels that should be used.

It's also possible to set the address.

TK flat_par_configure_address_using_display.jpg
![Flat PAR configure address using display](images/flat_par_configure_address_using_display.jpg)

### Fixtures in a universe

The 512 in DMX512 stands for 512 channels. A combination of fixtures in a network is called a universe. Every fixture in the universe can have a specific address, so that it's possible to set the channels of a specific device in the universe.

This means that the more fixtures we have in a universe, the more channels are blocked by a fixture. That means that we should always choose a set of channels based on what we want to achieve. If we don't want to specific functionality, we choose the set that doesn't contain it.

To calculate the next free address in a universe we can do it like this: `address of fixture a + # channels of fixture a = address of fixture b`

**Attention**: This is not required, but it's always a good idea to give every fixture it's own space. Only use the same address more than once if you want to have fixtures of the same type that are in sync with each other. Because in the end they all get the same channel values if they share the same address.

### Example

Let's say we have three fixtures:

* 2 x Flat PAR with 6 channels
* 1 x Fog machine with 1 channel

The first fixture is a Flat PAR with the address of 1. Using the formula **1 + 6**, then the next fixture can start at **7**. Then we add the second Flat PAR at address **7** and the Fog Machine at address **13**.

TK Image dmx512_universe_webusb_controller.png
![DMX512 Universe](images/dmx512_universe_webusb_controller.png)

The next thing we need is a WebUSB DMX512 controller to send the data (= an Array of 512 values) into the universe.


## Arduino

[Wikipedia defines Arduino](https://en.wikipedia.org/wiki/Arduino) like this:

> Arduino is an open source computer hardware and software company, project, and user community that designs and manufactures single-board microcontrollers and microcontroller kits for building digital devices and interactive objects that can sense and control objects in the physical and digital world.

> The project's products are distributed as open-source hardware and software, which are licensed under the GNU Lesser General Public License (LGPL) or the GNU General Public License (GPL),[1] permitting the manufacture of Arduino boards and software distribution by anyone.

> Arduino boards are available commercially in preassembled form, or as do-it-yourself (DIY) kits.

This is the perfect foundation to create our own WebUSB DMX512 Controller, because some Arduino (like the Arduino Leonardo) have the ability to be recognized by the computer as an external USB device (for example with the ATmega32u4 chip). This makes it possible to use the Arduino over WebUSB.

KT arduino_leonardo_explained.jpg
![Arduino Leonardo](images/arduino_leonardo_explained.jpg)

The Arduino has a set of female headers at the top with 18 connectors and the bottom with 14 connectors. They can be used to attach all kind of electronic devices or shields to the Arduino.

An Arduino shield can extend the functionality of the Arduino by adding it on top of the headers. This makes it possible for everyone to use all kind of devices without having to know anything about electrical engineering or soldering.


### Arduino DMX512 shield

In order to control a DMX512 universe with the Arduino we use the ["2.5kV Isolated DMX512 Shield for Arduino - R2"
on tindie](https://www.tindie.com/products/Conceptinetics/25kv-isolated-dmx-512-shield-for-arduino-r2/).

Let's put it on top of the Arduino:

1. The DMX512 shield, with two DMX connectors (I and II). I is the output to send data into the universe. II is empty.
2. The Arduino Leonardo, connected over MicroUSB to the computer.

TK webusb_dmx512_controller_explained.jpg
![Arduino with DMX512 shield attached](images/webusb_dxm512_controller_explained.jpg)

**Attention**: As of writing this article there is no DMX512 shield for the small Arduino boards (like the Arduino Micro). This means that in order to use this shield you have to at least get an Arduino Leonardo or similar in terms of the size, because the position of the headers must be the same.

A list of Arduino(-ish) boards that can be used in combination with the shield:

* Arduino Leonardo / Arduino Leonardo ETH
* Seeeduino Lite


TK Separator

The hardware is ready, so let's jump into the software.



### Setup

In order to be able to upload the code to the Arduino you have to setup the following stuff:

1. Checkout the [WebUSB DMX512 Controller Repository](https://github.com/NERDDISCO/WebUSB-DMX512-Controller)
  ```bash
  git clone git@github.com:NERDDISCO/WebUSB-DMX512-Controller.git
  ```
2. Install [npm](https://www.npmjs.com) dependencies
  ```bash
  # go into directory of repository
  cd WebUSB-DMX512-Controller

  # install dependencies
  npm install
  ```
3. Download & install the [Arduino IDE](https://www.arduino.cc/en/Main/Software#download) >= 1.8.5, so you are able to write & push code onto the Arduino
4. Open the Arduino IDE
5. Open the preferences: *Arduino / Preferences*
6. In the preferences dialog you have to change the *Sketchbook location* so that it points to the *sketchbook* folder that comes with the repository:

  ![Arduino IDE: Change sketchbook location](images/arduino_ide_preferences_sketchbook_location.png)
7. Close the Arduino IDE and then open it again (this is needed to load the new sketchbook that we selected in the step before)
8. Now we need to configure the Arduino IDE so that it can recognize our Arduino Leonardo:
   1. Select the model: *Tools / Board / Arduino Leonardo (WebUSB)*
   2. Select the USB port: *Tools / Port / /dev/tty.usbmodemWUAR1* (This should be something with *usb* in the name and can be different on your computer.

    **Attention**: This can only be selected if your Arduino is actually attached to your computer!
9. Open the sketch (if it's not already open): *File / Sketchbook / webusb_dmx512_controller*

If all the steps are done, you should now be able to verify the sketch by pushing the *verify* button (check mark icon) in the top left of the sketch:

![Arduino IDE: Verify Sketch](images/arduino_ide_verify_sketch.png)

Which will produce an output like this:

```bash
Sketch uses 8258 bytes (28%) of program storage space. Maximum is 28672 bytes.
Global variables use 888 bytes (34%) of dynamic memory, leaving 1672 bytes for local variables. Maximum is 2560 bytes.
```

The C-like code than runs on the Arduino is very simple, because we can use two awesome libraries:

* [WebUSB](https://github.com/webusb/arduino)
* [Conceptinetics](sketchbook/libraries/Conceptinetics)

These are already part of the sketchbook that came with the repository.

Let's take a look at the code:

### WebUSB DMX512 Controller Sketch

```arduino
#include <WebUSB.h>
#include <Conceptinetics.h>

// Whitelisted URLs
WebUSB WebUSBSerial(1, "nerddisco.github.io/WebUSB-DMX512-Controller");
#define Serial WebUSBSerial

// Amount of channels in the universe
#define universe 512

// dmx_master(channels , pin);
// channels: Amount of channels in the universe
// pin: Pin to do read / write operations on the DMX shield
DMX_Master dmx_master(universe, 2);

// Store the incoming WebUSB bytes
byte incoming[universe];

// Run once on startup
void setup() {
  // Initialize incoming with 0
  memset(incoming, 0, sizeof(incoming));

  // Wait until WebUSB connection is established
  while (!Serial) {
    ;
  }

  // Start transmission to DMXShield
  dmx_master.enable();
}

// Run over and over again
void loop() {
  // WebUSB is available
  if (Serial.available() > 0) {

    // Read 512 incoming bytes
    Serial.readBytes(incoming, universe);

    // Iterate over all universe
    for (int i = 0; i < universe; i++) {
      // Set the value for each channel
      dmx_master.setChannelValue(i + 1, incoming[i]);
    }
  }
}
```

### Test the device

We are done with setting up the actual WebUSB device. If you want to test everything you can open the [demo page](https://nerddisco.github.io/WebUSB-DMX512-Controller). This will open a development console to test the WebUSB DMX512 Controller.


## WebUSB

We will use WebUSB to send data directly from the browser via USB to the Arduino to control the DMX512 universe.

There are only just two thinks you have to take care when dealing with WebUSB:

1. You can run in it on localhost or via https online.
2. You need a device that can handle URL whitelisting, meaning that the device has to allow on which URLs it can be used.

But before we can jump into the code you have to understand what USB descriptors are, because otherwise the code will make no sense.

### USB Descriptors

TK usb_descriptors.png
![USB Descriptors](images/usb_descriptors.png)

https://wicg.github.io/webusb/#appendix-descriptors

USB devices identify themselves to the host by providing a set of binary structures known as descriptors. The first one read by the host is the device descriptor which contains basic information such as the vendor and product IDs assigned by the USB-IF and the manufacturer.


### WebUsbDmx512Controller.js

You can use this module like this:

* Trigger enable() by a user gesture (for example click or touch)
* This will open
* Choose your Arduino (filtered)
* send() your universe to it
* When you are done you can disconnect() (the device will still be paired, it's just the connection that is destroyed)

Let's jump into the code to see all of this happening:

```javascript
export default class WebUsbDmx512Controller {
  constructor(args) {
    this.device = null
  }

  /*
   * Enable WebUSB, which has to be triggered by a user gesture
   * When the device was selected, try to create a connection to the device
   */
  enable() {
    // Only request the port for specific devices
    const filters = [
      // Arduino LLC (10755), Leonardo ETH (32832)
      { vendorId: 0x2a03, productId: 0x8040 }
    ]

    // Request access to the USB device
    navigator.usb.requestDevice({ filters })
      // Open session to selected USB device
      .then(selectedDevice => {
        this.device = selectedDevice

        // Open connection
        return this.device.open()
      })

      // Select #1 configuration if not automatially set by OS
      .then(() => {
        if (this.device.configuration === null) {
          return this.device.selectConfiguration(1)
        }
      })

      // Get exclusive access to the #2 interface
      .then(() => this.device.claimInterface(2))

      // Tell the USB device that we are ready to send data
      .then(() => this.device.controlTransferOut({
          // It's a USB class request
          'requestType': 'class',
          // The destination of this request is the interface
          'recipient': 'interface',
          // CDC: Communication Device Class
          // 0x22: SET_CONTROL_LINE_STATE
          // RS-232 signal used to tell the USB device that the computer is now present.
          'request': 0x22,
          // Yes
          'value': 0x01,
          // Interface #2
          'index': 0x02
        })
      )

      .catch(error => console.log(error))
  }

  /*
   * Send data to the USB device
   */
  send(data) {
    // Create array with 512 x 0 and then concat with the data
    const universe = data.concat(new Array(512).fill(0).slice(data.length, 512))

    // Create an ArrayBuffer, because that is needed for WebUSB
    const buffer = Uint8Array.from(universe)

    // Send data on Endpoint #4
    return this.device.transferOut(4, buffer)
  }

  /*
   * Disconnect from the USB device
   */
  disconnect() {
    // Declare that we don't want to receive data anymore
    return this.device.controlTransferOut({
        // It's a USB class request
        'requestType': 'class',
        // The destination of this request is the interface
        'recipient': 'interface',
        // CDC: Communication Device Class
        // 0x22: SET_CONTROL_LINE_STATE
        // RS-232 signal used to tell the USB device that the computer is now present.
        'request': 0x22,
        // No
        'value': 0x01,
        // Interface #2
        'index': 0x02
      })
    )

    // Close the connection to the USB device
    .then(() => this.device.close())
  }
}
```




## What now?

### luminave
JO JO

### fivetwelve
FIVE TWELVE



---

## Resources

* [WebUSB Spec](https://wicg.github.io/webusb/)
* [Access USB devices on the Web](https://developers.google.com/web/updates/2016/03/access-usb-devices-on-the-web)
* [WebUSB demos running on Arduino](https://github.com/webusb/arduino)
* [WebUSB <3 Arduino](http://slides.com/andreastagi/webusbarduino)
* [Web enabling legacy devices](https://medium.com/@larsgk/web-enabling-legacy-devices-dc3ecb9400ed)
* [Programm a smart device directly, no install needed](https://medium.com/@kennethrohde/program-a-smart-device-directly-no-install-needed-cd8b29320d76)
* [Universal Serial Bus Class Definitions for Communication Devices](https://cscott.net/usb_dev/data/devclass/usbcdc10.pdf)
* [USB in a nutshell](http://www.beyondlogic.org/usbnutshell/usb1.shtml)
* [USB in a nutshell: Super good introduction to the USB standard](http://www.beyondlogic.org/usbnutshell/usb1.shtml)
* [USB made simple](http://www.usbmadesimple.co.uk)
