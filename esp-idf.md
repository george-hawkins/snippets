ESP-IDF
=======

This page covers installing the Espressif IoT Development Framework (ESP-IDF) and getting started with an initial project.

It's assumed that you already have Python 3 installed, if not see my notes on [installing Python](install-python.md).

Basic setup
-----------

Install the [prerequisites](https://docs.espressif.com/projects/esp-idf/en/latest/get-started/#step-1-install-prerequisites). For Mac this involved:

    $ pip install --user pyserial
    $ brew install cmake ninja

Then just work through the Espressif "[Get ESP-IDF](https://docs.espressif.com/projects/esp-idf/en/latest/get-started/#step-2-get-esp-idf)" steps:

    $ mkdir ~/esp
    $ cd ~/esp
    $ git clone --recurse-submodules -j8 git@github.com:espressif/esp-idf.git

`esp-idf` pulls in many submodules, e.g. [esptool](https://github.com/espressif/esptool), hence the use of `--recurse-submodules`.

Note: Espressif use `--recursive` while I used `--recurse-submodules`, introduced in Git 2.13, along with `-j8` which fetches submodules in parallel.

Then after cloning:

    $ cd esp-idf
    $ ./install.sh

This step installs various things under `~/.espressif`.

And finally update the `PATH` etc.:

    $ source ~/esp/esp-idf/export.sh

This last step needs to be added to your `~/.bashrc` or repeated each time you open a new terminal.

First project
-------------

Copy the included Hello World project:

    $ echo $IDF_PATH 
    /Users/georgehawkins/esp/esp-idf
    $ cd ~/esp
    $ cp -r $IDF_PATH/examples/get-started/hello_world .
    $ cd hello_world

You can now configure the build setup like so:

    $ idf.py menuconfig

This displays a terminal based menu system. However this step can be skipped as the defaults are fine.

Now to build everything:

    $ idf.py build

This takes some time (and involves building 910 object files).

Now we're almost ready to plug in the board but before we can do that it may be necessary to install a driver for the board's USB to UART bridge - see [here](esp-usb-to-uart.md) for more details.

Once that's done and the board is plugged in, the serial port, that corresponds to the board, needs to be determined. On Mac the port is usually `/dev/cu.SLAB_USBtoUART` and on Linux it's usually `/dev/ttyUSB0`.

Now to flash the result of the build process to the board:

    $ PORT=/dev/cu.SLAB_USBtoUART
    $ idf.py -p $PORT flash

And then interact with the board via the serial port monitor:

    $ idf.py -p $PORT monitor

Various bits of interesting boot information flash past:

    I (29) boot: ESP-IDF v4.1-dev-2263-g605da33c3 2nd stage bootloader
    I (29) boot: chip revision: 1
    I (40) boot.esp32: SPI Speed      : 40MHz
    I (45) boot.esp32: SPI Mode       : DIO
    I (49) boot.esp32: SPI Flash Size : 2MB
    ...
    I (195) cpu_start: Application information:
    I (200) cpu_start: Project name:     hello-world
    ...
    I (210) cpu_start: Compile time:     Feb 15 2020 22:28:28
    ...
    I (239) heap_init: Initializing. RAM available for dynamic allocation:
    I (246) heap_init: At 3FFAE6E0 len 00001920 (6 KiB): DRAM
    I (252) heap_init: At 3FFB2968 len 0002D698 (181 KiB): DRAM
    I (258) heap_init: At 3FFE0440 len 00003AE0 (14 KiB): D/IRAM
    I (264) heap_init: At 3FFE4350 len 0001BCB0 (111 KiB): D/IRAM
    I (271) heap_init: At 40089994 len 0001666C (89 KiB): IRAM
    ...
    W (296) spi_flash: Detected size(4096k) larger than the size in the binary image header(2048k). Using the size in the binary image header.

And then the output from the program:

    Hello world!
    This is esp32 chip with 2 CPU cores, WiFi/BT/BLE, silicon revision 1, 2MB external flash
    Restarting in 10 seconds...
    ...
    Restarting now.

So all it does is print 'Hello world!' and some basic information about the chip and any external flash - and then restarts the board.

You can exit the monitor with `ctrl-[`.

Let's look at that `W` line above, i.e the warning that it detected more SPI flash that was specified in the binary that was uploaded.

Start the menu:

    $ idf.py menuconfig

And go to `Serial flasher config` / `Flash size` and select `4 MB` rather than the `2 MB` default. Then `Q` (exit) and save. This updates the `sdkconfig` file:

    $ diff sdkconfig sdkconfig.old 
    82,83c82,83
    < CONFIG_ESPTOOLPY_FLASHSIZE_4MB=y
    ---
    > CONFIG_ESPTOOLPY_FLASHSIZE_2MB=y
    86c86
    < CONFIG_ESPTOOLPY_FLASHSIZE="4MB"
    ---
    > CONFIG_ESPTOOLPY_FLASHSIZE="2MB"
    ...

Now rebuild, reupload and check the monitor output:

    $ idf.py build
    $ idf.py -p $PORT flash
    $ idf.py -p $PORT monitor

Now the `W` no longer appears and the Hello World program outputs the correct amount of flash:

    Hello world!
    This is esp32 chip with 2 CPU cores, WiFi/BT/BLE, silicon revision 1, 4MB external flash

Updating ESP-IDF
----------------

As noted in the Espressif instructions for [updating your setup](https://docs.espressif.com/projects/esp-idf/en/latest/get-started/#updating-esp-idf), there are no special steps - you simply throw away your existing `esp-idf` directory and go through the above steps again.

