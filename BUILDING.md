# Building

The build of this device happens in a a few stages. First, the MCU is soldered
to the board. Then, `micronucleus` is flashed to it using the `ICSP` header.
Next, the USB components are soldered, exposing the MCU to your host machine
and allowing us to flash the application firmware `I2C Tiny USB`.

Then, the 5V I2C components are soldered. Finally, the 3.3V I2C components
are soldered (which is optional, and only needed if you intend to use the
3.3V I2C bus.

## The MCU

Solder `IC1` (the ATTiny85) onto the board.

### Flashing Micronucleus

Using `avrdude`, flash micronucleus bootloader onto the MCU. There are many
ways to drive `avrdude`, for example using an `usbasp` or `usbtiny` device,
or using an Arduino ISP with `arduino`, or using a Raspberry Pi with
`linuxgpio`.

See [this instructable](https://www.instructables.com/id/Refresh-Your-DigiSpark-clone-With-a-Smaller-Bootlo/)
for some background information.

Also, several eFuses have to be set. For information on fuses, take a look
[here](http://www.engbedded.com/fusecalc/).

Assuming some knowledge with `avrdude`, here's an example
commandline:

```
cd src/micronucleus/firmware
make t85_default
avrdude -c linuxgpio -p t85 -U flash:w:main.hex \
  -U lfuse:w:0xe1:m -U hfuse:w:0xdd:m -U efuse:w:0xfe:m
```

## USB Components

Solder `D1,D2` zener diodes. They will pull the data lines of the USB bus
to 3.6V (the USB specification wants 3.3V datalines, while our MCU is operating
at 5V). 

Solder `R1,R2`, which are current limiters to the MCU's input pins.

Solder `R4`, which pulls the `USB-` dataline to 5V, signalling to the host
that we are a low speed USB device.

### Flashing I2C Tiny USB

Connecting the device to a host computer will now show the Micronucleus USB
device. You may want to install the [udev rules](https://github.com/micronucleus/micronucleus/blob/master/commandline/49-micronucleus.rules)
so that you can run `micronucleus` as user.

```
sudo cp src/micronucleus/commandline/49-micronucleus.rules /etc/udev/rules.d/
sudo service udev restart

cd src/I2C-Tiny-USB/digispark
make hex
micronucleus --run --dump-progress --type intel-hex main.hex
```

Plug in your device at the prompt, and the application will be flashed. Shortly
after finishing, the device will be recognized as a `i2c-tiny-usb` and connected
to a Linux `i2c` bus. Running `i2cdetect -l` will show it.

## 5V I2C Components

Solder `J1` to the board. This is the 5V I2C bus Qwiic connector.
Solder `R6, R8` to the board. These are the I2C pullup resistors for the 5V bus.

You should now be able to communicate with 5V I2C devices!

## Optional Components

### 3.3V I2C Components

Solder `J2` to the board. This is the 3.3V I2C bus Qwiic connector.
Solder `R5, R7` to the board. These are the I2C pullup resistors for the 3.3V
bus.
Solder `Q1, Q2` to the board. These are level shifters between the 5V logic on
the board and the 3.3V I2C bus lines.
Solder `IC2` to the board. This is a 3.3V voltage LDO that powers the 3.3V I2C
bus.
Solder `C1, C2, C3` to the board. These are capacitors for the input and output
voltage lines of the LDO.

You should now be able to communicate with 3.3V I2C devices!

### Power LED

If you'd like to see if the device is powered, solder `D3` and `R3` to the
board.
