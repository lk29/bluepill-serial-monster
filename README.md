# bluepill-serial-monster

_bluepill-serial-monster_ is a firmware for _STM32 Blue Pill_ that turns it
into a _3 Port USB-to-Serial_ adapter. The firmware implements a USB 2.0
full-speed composite device that consists of 3 USB CDC devices.

_STM32 Blue Pill_ is a ridiculously cheap _STM32_ development board which
is available in many stores around the globe. The board contains decent
hardware that supports _USB 2.0 Full-Speed_, has 3 independent _USARTs_
and enough processing power to handle high-speed _UART_ communications.

**Note**: some _Blue Pill_ clones have an incorrect pull-up resistor soldered
to the _USB D+_ line (_PA12_) which prevents them from being successfully
detected by the host. Please refer to
[Fixing USB on Blue Pill Boards](#fixing-usb-on-blue-pill-boards)
for more information.

Some USB controllers work fine even with faulty _Blue Pill_ boards. If your
board works with your computer, don't bother fixing it.

## Features

* 3 independent _UART_ ports;
* Hardware flow control (**RTS**/**CTS**) support<sup>1</sup>;
* **DSR**/**DTR**/**DCD**/**RI** signals support;
* 7 or 8 bit word length;
* None, even, odd parity;
* 1, 1.5, and 2 stop bits;
* Works with _CDC Class_ drives on _Linux_, _OS X_, and _Windows_;
* Supports all baud rates up to 2 MBaud;
* **TXA** signal for controlling RS-485 transceivers (**DE**, **/RE**);
* _DMA_ _RX_/_TX_ for high-speed communications;
* _IDLE line_ detection for short response time;
* Signed _INF_ driver for _Windows XP, 7, and 8_;
* Built-in command shell for device parameters configuration;
* No external dependencies other than _CMSIS_;

(1) _UART1_ does not support hardware flow control because _RTS_/_CTS_ pins
(_PA12_, _PA11_) are used for _USB_ communication and cannot be remapped.
If you need hardware flow control, use _UART2_ or _UART3_.

## Donations

If this project helped you with whatever you use it for
or you just want to say thanks, you can buy me a coffee :)

[!["Buy Me A Coffee"](https://www.buymeacoffee.com/assets/img/custom_images/orange_img.png)](https://www.buymeacoffee.com/r2axz)

## UART Signal Levels

Although _STM32F103C8T6_ installed on the Blue Pill board is a **3.3 V**
device, a number of its inputs are actually **5 V** tolerant.
This means you can safely use the selected inputs with **3.3** and **5 V**
TTL devices.

**Do not use non 5 V tolerant inputs with 5 V devices as doing that will
result in permanent damage of MCU inputs or MCU itself.**

**When configured as an output, none of the _STM32F103C8T6_ pins is 5 V
tolerant. Make sure you don't accidentally get more than 4.0 V on such pin
or damage may occur.**

**5 V** tolerant pins are shown **in bold** in the next section.

## UART Pinout

| Signal |   Direction   |     UART1     |     UART2     |     UART3     |
|:-------|:-------------:|:--------------|:--------------|:--------------|
|   RX   |      IN       |    **PA10**   |      PA3      |    **PB11**   |
|   TX   |      OUT      |      PA9      |      PA2      |      PB10     |
|   RTS  |      OUT      |      N/A      |      PA1      |      PB14     |
|   CTS  |      IN       |      N/A      |      PA0      |    **PB13**   |
|   DSR  |      IN       |    **PB7**    |    **PB4**    |    **PB6**    |
|   DTR  |      OUT      |      PA4      |      PA5      |      PA6      |
|   DCD  |      IN       |    **PB15**   |    **PB8**    |    **PB9**    |
|   RI   |      IN       |    **PB3**    |    **PB12**   |    **PA8**    |
|   TXA  |      OUT      |    **PB0**    |    **PB1**    |    **PA7**    |

Note: **5 V** tolerant input pins are shown **in bold**.

## Control Signals (Default Configuration)

**RTS**, **CTS**, **DSR**, **DTR**, **DCD**, **RI** are **active-low** signals,
**TXA** is an active-high signal.

**TXA** (**TX** **A**ctive) is active when UART is transmitting data and
can be used to control **DE** and **/RE** pins of RS-485 transceivers.

**TXA** goes inactive within 0.6 us after the transmission is complete,
which meets RS-485 and IO-link timing requirements at speeds up to 920 kBaud
with almost double safety margin.

**DSR**, **DTR**, and **DCD**, **RI** are connected to the internal _weak pull-up_
resistors, so they remain inactive at rest.

**CTS** is **pulled down** internally, which enables _UART TX_ when nothing is
connected to **CTS**. Hardware flow control is always on, but it does not get
in the way of communications as long as nothing is connected to the flow control lines.

**RTS** can be controlled by the host, but as soon as the _UART RX_ buffer is
**half-full**, **RTS** is forced to the **inactive** state. As long as more than
one half of the buffer space is available, **RTS** remains in the state set
by the host. Please take this behaviour into account if you rely on the
**RTS** signal to control non-standard periphery.

**DSR**, **DCD**, and **RI** are polled 50 times per second.

_UART DMA RX/TX_ buffer size is **1024** bytes.

## Advanced Configuration

_bluepill-serial-monster_ provides a configuration shell that allows
controlling various parameters of the UART signal lines.

To access the configuration shell, open _UART1_ with any terminal emulator
application (such as _screen_, _Tera Term_, etc.) and connect **PB5** to ground. Serial port settings do not matter.

You should see the configuration shell prompt:

```text
*******************************
* Configuration Shell Started *
*******************************

>
```

The configuration shell has minimal support for ANSI escape sequences. You can
use the arrow keys to move the cursor when editing a command, erase text with _Backspace_, and insert text anywhere in the command. You can also recall the
last command by pressing _UP_.

Command and parameter names are case-sensitive.

To get the list of available commands, type:

```text
>help
```

To get command-specific help, type:

```text
>help command-name
```

### UART Port Parameters

UART port parameters can be viewed and set with the _uart_ command:

```text
>help uart
uart: set and view UART parameters
Usage: uart port-number|all show|signal-name-1 param-1 value-1 ... [param-n value-n] [signal-name-2 ...]
Use "uart port-number|all show" to view current UART configuration.
Use "uart port-number|all signal-name-1 param-1 value-1 ... [param-n value-n] [signal-name-2 ...]"
to set UART parameters, where signal names are rx, tx, rts, cts, dsr, dtr, dcd,
and params are:
  output        [pp|od]
  active        [low|high]
  pull          [floating|up|down]
Example: "uart 1 tx output od" sets UART1 TX output type to open-drain
Example: "uart 3 rts active high dcd active high pull down" allows to set multiple parameters at once.
```

Changes to the UART parameters are applied instantly; however, the configuration
is not stored in the flash memory until you explicitly save it with:

```text
>config save
```

To view current configuration of all UART ports, type:

```text
>uart all show
```

To view current configuration of a particular UART port, type:

```text
>uart port-number show
```

where _port-number_ is in range of 1 to 3.

Output type can be set for any output signal. Available output types are:

* **pp** for push-pull output;
* **od** for open-drain output;

Example:

```text
uart 1 tx output od
```

Pull type can be set for any input signal. Available pull types are:

* **floating** for floating input;
* **up** for weak pull up;
* **down** for weak pull down;

Example:

```text
uart 1 dcd pull up
```

Signal polarity can be set for all input and output signals except for **RX**,
**TX**, and **CTS**. Available signal polarities are:

* **low** for active-low signal polarity;
* **high** for active-high signal polarity;

Example:

```text
uart 1 rts active high
```

It is possible to set multiple signal parameters for multiple signals in one
command:

```text
uart 1 tx output od rts output od active high
```

If the uart command encounters a syntax error or an invalid parameter in
the middle of a multiple parameters command line, it stops execution
immediately. However, it does not roll back valid parameters set before
the point where the error occured.

It is also possible to set signal parameters for multiple ports in one command:

```text
uart all tx output od
```

### Saving and Resetting Configuration

To permanently save current device configuration, type:

```text
config save
```

To reset the device to the default settings, type:

```text
config reset
```

The default configuration is automatically stored in the flash memory after reset.

## Flashing Firmware

Download binary firmware from the
[Releases](https://github.com/r2axz/bluepill-serial-monster/releases) page.

Flash with [ST-LINK](https://www.st.com/en/development-tools/st-link-v2.html)
or similar programmer.

```bash
st-flash --format ihex write bluepill-serial-monster.hex
```

You can also flash _STM32F103C8T6_ via a built-in serial bootloader. Visit
[https://www.st.com/en/development-tools/flasher-stm32.html](https://www.st.com/en/development-tools/flasher-stm32.html)
for instructions and software.

## Windows Driver (WinXP, 7, 8)

_Windows_ versions prior to _Windows 10_ require an _INF_ file that maps
the device Vendor ID / Product ID to the Microsoft _usbser.sys_ CDC ACM driver.
_Windows 10_ does not require this and loads the standard driver automatically.

A signed _INF_ file for _Windows XP, 7, and 8_ is included in the distribution.
To install the _INF_ file, plug in _bluepill-serial-monster_ and point _Windows_
to a directory containing both **bluepill-serial-monster.inf** and **bluepill-serial-monster.cat** files during a new device installation.

Alternatively, you can open _Windows Device Manager_, right-click on any of the
_Bluepill Serial Monster_ devices, choose _Update driver_ and point Windows to
the _INF_ file directory from there.

## Fixing USB on Blue Pill Boards

_STM32 Blue Pill_ boards come in slightly different variations. Nevertheless,
their schematic is very similar. Below you will find the instructions on how to
identify and replace the incorrect USB pull-up resistor on any _Blue Pill_ board.

With a digital multimeter, measure resistance between **PA12** and **3.3 V**
pads on the board. If the resistance reads close to **1.5k** (**1500 ohms**),
then your board is either non-faulty or faulty for some other reason,
and this section does not apply.

If the resistance if far away from **1.5k** (such as **4.7k** or **10k**),
you will have to locate the incorrect resistor on the board and replace it
with a **1.5k** or **1.8k** resistor.

If your board has component names on it, locate **R10**. Otherwise, trace the
board to find the incorrect resistor.

Once you identified the incorrect resistor, replace it with a **1.5k** or
**1.8k** resistor.

## Building Firmware

### Prerequisites

Install the following software:

* [GNU make](https://www.gnu.org/software/make/)
* [arm-none-eabi toolchain](
    https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm/downloads)
* [open source st-link](<https://github.com/texane/stlink>)
* [STM32CubeF1](
    <https://www.st.com/en/embedded-software/stm32cubef1.html>)

Here is an example assuming everything is installed in ~/stm32/,
and we use _bash_:

ARM toolchain and st-link must be added to PATH.

```bash
# add ARM toolchain path
export PATH=~/stm32/gcc-arm-none-eabi/bin:$PATH
# add stlink path
export PATH=~/stm32/stlink-install/bin:$PATH
```

Path to STM32CubeF1 should be also exported (use ~/.bash_profile):

```bash
# export STM32Cube
export STM32CUBE_PATH=~/stm32/stm32cube
```

## Building

To build the firmware, **cd** to the project directory and run

```bash
make
```

To flash the MCU using st-link, run

```bash
make flash
```

To remove object and dependency files, run

```bash
make clean
```

To remove object, dependency, and firmware files, run

```bash
make distclean
```
