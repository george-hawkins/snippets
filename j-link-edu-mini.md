Connecting up the J-Link Edu Mini
=================================

TLDR; ...

If there's a good getting started guide to simply connecting up the J-Link Edu Mini to a dev board for the first time, I haven't found it.

When you buy a J-Link Edu Mini, you get the J-Link Mini debugger, a USB cable, 10-pin cable (often called an SWD cable) and a 20-pin cable (often called a JTAG cable).

Note: even if one cable is _commonly_ called an SWD cable and the other a JTAG cable, both can actually be used for either SWD or JTAG.

Most ARM Cortex dev boards use SWD debugging so let's focus of the 10-pin cable. The first obvious question is which way to connect the cable to the J-Link.

You've got four options, only x of which are valid:

red-top cable pointing left
red-top cable pointing right
red-bottom cable pointing left
red-bottom cable pointing right

The valid option/s is/are y.

Then connecting the other end to your dev board is easy _if_ it has an SWD header like this:

https://media.digikey.com/Photos/CnC%20Tech/3220-10-0100-00.JPG

The notch on the cable's connector means that there's only one way round you can plug it into this.

However, many boards don't have an SWD header like this. Instead, you have to search for the relevant SWD pins, among all the other pins available around the edge of the board, or for a group of 5x2 pins somewhere else on the board.

So what pins are you looking for? You'll make things much easier for yourself if you buy a breakout like this for the end of the SWD cable that is not plugged into the J-Link itself:

https://www.adafruit.com/product/2743
https://cdn-shop.adafruit.com/1200x900/2743-00.jpg
https://cdn-shop.adafruit.com/1200x900/2743-04.jpg

If we look at the names of the pins on this breakout, we see Vref, GND, GND, KEY, GNDd on one side and SWIO, CLK, SWO, NC (not connected) and RST on the other.

Note: SWIO is often also written as SWDIO or DIO and CLK as SWCLK.

We can line that up with the pinout shown on the J-Link Edu Mini [product page](https://www.segger.com/products/debug-probes/j-link/models/j-link-edu-mini/):

https://www.segger.com/fileadmin/images/products/J-Link/Accessory/Adapters/J-Link_9-pin_Cortex-M_Adapter.png

In this diagram, we can see that two of the pins aren't connected at all, i.e. the ones corresponding to KEY and GNDd on the breakout above.

KEY is expected to not be present - on some SWD headers that pin is actually removed and this helps you with orienting things (i.e. you know the missing pin should be the second one up from the bottom left). And GNDd is just a pin that the J-Link Edu Mini doesn't bother with - GNDd stands for xxx and it's a rather mysterious pin that seems to be rarely if ever used even on more expensive debuggers that support this pin. About the only documentation for it is [here] ... on xyz where ARM describe it as "xxx".

On the Segger pinout above we see a pin called TDI and three pins with the names TMS, TCK and TDO. These are the names for the pins when working with [JTAG](https://en.wikipedia.org/wiki/JTAG#Electrical_characteristics). But as we're working with SWD, we're not interested in TDI (that's a pure JTAG pin) and we're only interested in the SWD names of the other three pins, i.e. SWDIO, SWCLK and SWO.

So looking at our breakout about again, we're basically left with GND and five other relevant pins - Vref, SWIO, CLK, SWO and RST. Things now get even easier - it turns out that only GND, SWIO and CLK really have to be connected up for an [SWD](https://wiki.segger.com/SWD) connection.

So what are the other pins and why _might_ you connect them:

* Vref can be connected to the regulated voltage output of your dev board (often labelled 3V3). Connecting this pin allows the debugger to detect when the board is powered up and gives it a reference voltage against which it can determing if the SWIO or CLK pins are high (usually, such a reference isn't required but apparently with some boards it can be an issue).
* SWO is an optional output pin that isn't a core part of SWD but which you can use to output your own debug specific messages, i.e. you can pepper your code with printf-like statements which are configured to write to the SWO pin and this data can be read and output e.g. by your IDE during a debug session. For more on SWO, see e.g. this [tutorial from AdAstra](https://adastra-soft.com/poor-man-arm-cortex-m-swo/) or this [one from @McuOnEclipse](https://mcuoneclipse.com/2016/10/17/tutorial-using-single-wire-output-swo-with-arm-cortex-m-and-eclipse/).
* RST allows the debugger to trigger hard resets of the MCU on the dev board. Depending on how the board is set up this may be necessary (e.g. typical Chinese black-pill boards) but often it is not.

So generally you end up with a setup like this:

https://cdn-learn.adafruit.com/assets/assets/000/069/853/original/micropython___circuitpython_itsy_fritzing_bb.png

Don't be confused - the "RESET" on the board is the label for the board's push button and not for the pin that the green wire is connected to.

So here SWIO, CLK are connected along with GND and Vref (an noted, connecting Vref may not strictly be necessary).

The board above is an Adafruit [ItsyBitsy M0 Express](https://www.adafruit.com/product/3727), you'll need to find the location of the relevant pins on your particular board.

The ItsyBitsy also has a RST pin (bottom left) but SWO is not made available as a pin so on this particular board you can't connect to SWO even it you want to.

Here's an example (from [here](https://wiki.segger.com/Black_Pill) on the Segger wiki) of a board where RST has to be connected up (in addition to GND, Vref, SWIO and CLK):

https://wiki.segger.com/images/4/49/black-pill-setup.PNG

3V3, SWIO, CLK and GND are connected to on the left-hand side and RST is the pin bottom-right connected to with a yellow wire.

Upgrading the J-Link Mini
-------------------------

Fill out these sections with how to use J-Link commander (first three tabs in browser).

Connecting to your dev board
----------------------------

Notes
-----

For more on the JTAG and SWD pins on a 10-pin connector, see this [page from Keil](https://www.keil.com/support/man/docs/ulinkplus/ulinkplus_jtagswd_interface.htm) and for more on the JTAG and SWD pins on a 20-pin connector see the [page from Segger](https://www.segger.com/products/debug-probes/j-link/technology/interface-description/) (as you can see for the 20-pin connector one side is all GND pins so you've actually only got a few real additonal pins over those found on the 10-pin connector).
