Before finalizing this fork, this commit needs to include moving to git.beagleboard.org CI.


CC1352 flasher for BeaglePlay and BeagleConnect Freedom [![Build Status](https://openbeagle.org/beagleconnect/cc1352-flasher/badges/main/pipeline.svg)](https://openbeagle.org/beagleconnect/cc1352-flasher)
==========================================

This folder contains a python script that communicates with the boot loader of the Texas Instruments CC13xx SoCs (System on Chips) on BeaglePlay or BeagleConnect Freedom.
It can be used to erase, program, verify and read the flash of those SoCs with a simple USB to serial converter.

### Requirements

To run this script you need a Python interpreter, Linux and Mac users should be fine, Windows users have a look here: [Python Download][python].

Alternatively, Docker can be used to run this script as a one-liner without the need to install dependencies, see [git-developer/ti-cc-tool](https://github.com/git-developer/ti-cc-tool) for details.

BeaglePlay provides the required GPIO and UART connections between the AM62 and CC1352 to enable programming using the CC1352 serial BSL.

BeagleConnect Freedom provides a USB to serial bridge to the CC1352 to enable programming using the CC1352 serial BSL.

### Dependencies

This script uses the pyserial package to communicate with the serial port and chip (https://pypi.org/project/pyserial/). You can install it by running `pip install pyserial`.

If you want to be able to program your device from an Intel Hex file, you will need to install the IntelHex package: https://pypi.python.org/pypi/IntelHex (e.g. by running `pip install intelhex`).

The script will try to auto-detect whether your firmware is a raw binary or an Intel Hex by using python-magic:
(https://pypi.python.org/pypi/python-magic). You can install it by running `pip install python-magic`. Please bear in mind that installation of python-magic may have additional dependencies, depending on your OS: (https://github.com/ahupp/python-magic#dependencies).

If python-magic is _not_ installed, the script will try to auto-detect the firmware type by looking at the filename extension, but this is sub-optimal. If the extension is `.hex`, `.ihx` or `.ihex`, the script will assume that the firmware is an Intel Hex file. In all other cases, the firmware will be treated as raw binary.

### BeagleConnect Freedom

The MSP430 USB-to-UART bridge in [BeagleConnect Freedom] uses a serial [BREAK](https://en.wikipedia.org/wiki/Universal_asynchronous_receiver-transmitter#Break_condition) to know when to invoke the CC1352P7 BSL. Use `--bootloader-send-break` to activate the bootloader.

### BeaglePlay

The GPIOs on [BeaglePlay] are toggled using the `gpiod` library and found via the device tree provided labels. Use `--play` to select these gpios and the proper UART.

### Other notes

For all the CC13xx and CC26xx families, the ROM bootloader is configured through the `BL_CONFIG` 'register' in CCFG. `BOOTLOADER_ENABLE` should be set to `0xC5` to enable the bootloader in the first place.

This is enough if the chip has not been programmed with a valid image. If a valid image is present, then the remaining fields of `BL_CONFIG` and the `ERASE_CONF` register must also be configured correctly:

* Select a DIO by setting `BL_PIN_NUMBER`
* Select an active level (low/high) for the DIO by setting `BL_LEVEL`
* Enable 'failure analysis' by setting `BL_ENABLE` to `0xC5`
* Make sure the `BANK_ERASE` command is enabled: The `BANK_ERASE_DIS_N` bit in the `ERASE_CONF` register in CCFG must be set. `BANK_ERASE` is enabled by default.

If you are using CC13xx/CC26xxware, the relevant settings are under `startup_files/ccfg.c`. This is the case if you are using Contiki.

Similar to the CC2538, the bootloader will be activated if, at the time of reset, failure analysis is enabled and the selected DIO is found to be at the active level.

As an example, to bind the bootloader backdoor to KEY_SELECT on the SmartRF06EB, you need to set the following:

* `BOOTLOADER_ENABLE = 0xC5` (Bootloader enable. `SET_CCFG_BL_CONFIG_BOOTLOADER_ENABLE` in CC13xx/CC26xxware)
* `BL_LEVEL = 0x00` (Active low. `SET_CCFG_BL_CONFIG_BL_LEVEL` in CC13xx/CC26xxware)
* `BL_PIN_NUMBER = 0x0B` (DIO 11. `SET_CCFG_BL_CONFIG_BL_PIN_NUMBER` in CC13xx/CC26xxware)
* `BL_ENABLE = 0xC5` (Enable "failure analysis". `SET_CCFG_BL_CONFIG_BL_ENABLE` in CC13xx/CC26xxware)

These settings are very useful for development, but enabling failure analysis in a deployed firmware may allow a malicious user to read out the contents of your device's flash or to erase it. Do not enable this in a deployment unless you understand the security implications.

### Usage

Install from PyPi using `pip install cc1352-flasher` or using the local repo using `pip install .`.

You can find info on the various options by executing `cc1352-flasher -h`.

### Remarks

If you found a bug or improved some part of the code, please submit an [issue] or pull request.

##### Authors
Jason Kridner, BeagleBoard.org (c) 2023, <jkridner@beagleboard.org>
Jelmer Tiete (c) 2014, <jelmer@tiete.be>   
Loosly based on [stm32loader] by Ivan A-R <ivan@tuxotronic.org>   

[BeagleConnect Freedom]: https://beagleconnect.org "BeagleConnect Freedom"
[BeaglePlay]: https://beagleplay.org "BeaglePlay"
[python]: http://www.python.org/download/ "Python Download"
[contiki cc2538dk]: https://github.com/contiki-os/contiki/tree/master/platform/cc2538dk "Contiki CC2538DK readme"
[stm32loader]: https://github.com/jsnyder/stm32loader "stm32loader"
[issue]: https://openbeagle.org/beagleconnect/cc1352-flasher/issues "issue"
