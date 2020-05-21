ESP32 USB to UART bridge
========================

Many ESP32 boards, including Espressif's own [ESP32-DevKitC](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/hw-reference/esp32/get-started-devkitc.html), Adafruit's [HUZZAH32](https://www.adafruit.com/product/3405) and Sparkfun's [ESP32 Thing Plus](https://www.sparkfun.com/products/15663), use the SiLabs CP2104 USB to UART bridge.

Installing a driver for a new development board may be the norm when using Window, but this is one of the first for which I've had to do this on mac OS. The drivers for Windows and Mac can be found on the [SiLabs CP2104 driver page](http://www.silabs.com/products/development-tools/software/usb-to-uart-bridge-vcp-drivers). Nothing needs to be installed on Linux as the necessary [driver](https://github.com/torvalds/linux/blob/master/drivers/usb/serial/cp210x.c) has been part of the standard kernel for more than a decade.

Once the appropriate driver is installed and the board is connected, follow the Espressif [instructions](https://docs.espressif.com/projects/esp-idf/en/latest/get-started/establish-serial-connection.html) to determine the board's serial port or just try the following.

Mac
---

On Mac, once the board is plugged in, it's simple:

    $ ls /dev/cu.*
    /dev/cu.SLAB_USBtoUART ...

There's a seemingly identicall `tty` file: 

    $ ls /dev/tty.*
    /dev/tty.SLAB_USBtoUART ...

However if you use this then Espressif tools like `idf.py monitor` will complain:

    --- WARNING: Serial ports accessed as /dev/tty.* will hang gdb if launched.
    --- Using /dev/cu.SLAB_USBtoUART instead...

Linux
-----

On Linux the board can typically be accessed as `/dev/ttyUSB0`.

If you're already a member of the group that can access serial devices, `dialout` on Ubuntu, no additional setup is needed. But if you'd like permissions on the device to be automatically set so that all users can access it and access via a fixed name then you can do so with this udev rule:

    # SiLabs CP2104 USB to UART Bridge Controller.
    SUBSYSTEM=="tty", ATTRS{idVendor}=="10c4", ATTRS{idProduct}=="ea60", \
        SYMLINK+="cp2104", MODE="0666"

If you don't already have a file, containing such rules, under `/etc/udev/rules.d` then just create a file called `50-serial-ports.rules` there and add the rule to this file.

With this rule you can always access the board like so:

    $ PORT=/dev/cp2104
    $ screen $PORT 115200
