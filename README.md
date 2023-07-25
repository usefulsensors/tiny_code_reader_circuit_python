# Person Sensor in CircuitPython
How to access Useful Sensor's Tiny Code Reader from CircuitPython.

## Introduction

The Tiny Code Reader is a small hardware module that's intended to make it easy
to scan QR codes. It has an image sensor and a microcontroller with pretrained
software and outputs information from any identified codes over I2C.

There's a [detailed developer guide](https://usfl.ink/tcr_dev)
available, but this project has sample code that shows you specifically how to 
get the module up and running using CircuitPython. It has been tested with
Raspberry Pi Pico and Trinkey boards, but should hopefully work on other
CircuitPython-compatible platforms. If it isn't working on a board that isn't
listed, it's likely to be a problem with either the pin numbers assigned to the
I2C interface, or some other I2C initialization issue.

## Setting up Circuit Python

You should read the [official guide to setting up CircuitPython on a Pico](https://learn.adafruit.com/getting-started-with-raspberry-pi-pico-circuitpython)
for the latest information, but here is a summary:

 - Download CircuitPython for the for your board from circuitpython.org. For
 example the Pico version is available at https://circuitpython.org/board/raspberry_pi_pico/.
 This project has been tested using the `8.2.0` version.
 - Hold down the `bootsel` button on the Pico and plug it into a USB port.
 - Drag the CircuitPython uf2 file onto the `RPI-RP2` drive that appears.

Once you've followed those steps, you should see a new `CIRCUITPY` drive appear.
You can now drag `code.py` files onto that drive and the Pico should run them.

## Wiring Information

Wiring up the device requires 4 jumpers, to connect VDD, GND, SDA and SCL. The 
example here uses I2C port 0, which is assigned to GPIO4 (SDA, pin 6) and GPIO5
(SCL, pin 7) in software. Power is supplied from 3V3(OUT) (pin 36), with ground
attached to GND (pin 38).

If you're using a device that comes with a standard connector like a Qwiic or
Grove interface, you should be able to use the right cable to connect to the
Qwiic connector on the sensor. You can do this on the Wio Terminal with a
[Qwiic to Grove cable](https://www.sparkfun.com/products/15109) plugged into the
left port (below the speaker). 

### Pico Wiring

For the Pico you'll need to connect directly to pins, following the wiring
scheme shown below:

![Wiring diagram for Person Sensor/Pico](pico_tiny_code_reader_bb.png)

If you're using [Qwiic connectors](https://www.sparkfun.com/qwiic), the colors 
will be black for GND, red for 3.3V, blue for SDA, and yellow for SDC.

## Detecting QR Codes

Copy the `code.py` file in this folder over to the `CIRCUITPY` drive. If you
connect over `minicom` or a similar program to view the logs, you should see
information about the codes seen by the sensor printed out, and messages about
any errors encountered. If no codes are found, there should be output like this:

```bash
0
```

The number is the length of the content message read from the QR code. When 
it's zero, no code was detected. Here's an example of the output with a code
present:

```
25
http://en.m.wikipedia.org
```

To get this output, I opened [an example QR code](https://en.wikipedia.org/wiki/QR_code#/media/File:QR_code_for_mobile_English_Wikipedia.svg)
on my phone and held it about fifteen centimeters or six inches from the 
module, facing the camera.

## Troubleshooting

### Power

The first thing to check is that the sensor is receiving power through the
`VDD` and `GND` wires. The simplest way to test this is to look at the LED on
the front of the module. Whenever it has power, it should be flashing blue
multiple times a second. If this isn't happening then there's likely to be a
wiring issue with the power connections.

### Communication

If you see connection errors when running the code detection example, you may
have an issue with your wiring. Connection errors often look like this:

```bash
Traceback (most recent call last):                                                    
  File "code.py", line 50, in <module>                                                
OSError: [Errno 19] No such device 
```

To help track down what's going wrong, you can uncomment the following lines in
the script:

```python
while True:
    print(i2c.scan())
    time.sleep(PERSON_SENSOR_DELAY)
```

This will display which I2C devices are available in the logs. You want to make
sure that address number `12` is present, because that's the one used by the
person sensor. Here's an example from a board that's working:

```bash
[12]  
```

You can see that `12` is shown. If it isn't present then it means the sensor
isn't responding to I2C messages as it should be. The most likely cause is that 
there's a wiring problem, so if you hit this you should double-check that the 
SDA and SCL wires are going to the right pins.

## Going Further

This script is designed to be a simple standalone example of reading data from
the Tiny Code Reader, but you should check out the sample code in the [Developer Guide](https://usfl.ink/tcr_dev)
to see more advanced use cases like provisioning wifi or keyboard emulation.