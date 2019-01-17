# Linux kernel FL2000DX/IT66121FN dongle DRM driver

Clean re-implementation of FrescoLogic FL2000DX DRM driver and ITE Tech IT66121F driver, allowing to enable full display controller capabilities for [USB-to-HDMI dongle based on such chips](https://www.aliexpress.com/item/HD-1080P-USB-3-0-To-HDMI-External-Video-Graphic-Card-Multi-Display-Cable-Adapter-Converter/32808836824.html?spm=a2g0s.9042311.0.0.4a9f4c4dow19O6) in Linux

## FL2000DX
The FL2000DX is Fresco Logic’s USB 3.0 Display device controller. It integrates Fresco Logic’s display transfer engine, USB 3.0 device controller, USB 3.0 transceiver, and a VGA (D-Sub) DAC. Fresco Logic’s display transfer engine is designed with Fresco Logic’s proprietary architecture and processes the video stream optimally for USB 3.0 bandwidth. The high performance video DAC provides clear and crisp display quality, and supports full HD (1920×1080) resolution. The integrated USB 3.0 device controller and transceiver were developed in conjunction with the de-facto standard Fresco USB 3.0 host controllers, which ensures the best performance and compatibility.

## IT66121FN
The IT66121 is a high-performance and low-power single channel HDMI transmitter, fully compliant with HDMI 1.3a, HDCP 1.2 and backward compatible to DVI 1.0 specifications. IT66121 also provide the HDMI1.4 3D feature, which enables direct 3D displays through an HDMI link. The IT66121 serves to provide the most cost-effective HDMI solution for DTV-ready consumer electronics such as set-top boxes, DVD players and A/V receivers, as well as DTV-enriched PC products such as notebooks and desktops, without compromising the performance. Its backward compatibility to DVI standard allows connectivity to myriad video displays such as LCD and CRT monitors, in addition to the ever-so-flourishing Flat Panel TVs.

## Implementation
![Diagram](fl2000.svg)

### Debug scenario
Most probably, in order to implement flexibility of register programming we will use scripts (Python or shell) during development phase when there is lots of unknowns. Later – when programming flow will be clear and development is done – register programming flow will have to be implemented in the driver’s modules. Same is applied for I2C communication with respect to EDID parsing or IT66121 client programming or whatever else.
NOTE: there are 2 exceptions: interrupts status register is still processed in the driver, IT66121 client detection will still happen on I2C automatically

### Debugging implementation
In order to implement debugging, following must be implemented in the driver:
1. No register programming for fl2000 or it66121
1. Storing of interrupt status values in a ringbufffer for further reading through debugfs
1. Implementing debugfs entries
#### Debugfs entries
*Register programming*
- reg_address: specify register address to work with
- reg_data: read causes read from register, write causes write to register

*I2C access*
- i2c_address: i2c device address to operate with
- i2c_offset: specify device register address to work with
- i2c_value: read causes i2c read, write causes i2c write

*Interrupt handling*
- status: array of values of interrupt statuses since last reading

*Entries structure*<br>
:open_file_folder: fl2000_drm<br>
&nbsp;&nbsp;&nbsp;&nbsp;:page_facing_up:reg_address<br>
&nbsp;&nbsp;&nbsp;&nbsp;:page_facing_up:reg_data<br>
&nbsp;&nbsp;&nbsp;&nbsp;:page_facing_up:i2c_address<br>
&nbsp;&nbsp;&nbsp;&nbsp;:page_facing_up:i2c_offset<br>
&nbsp;&nbsp;&nbsp;&nbsp;:page_facing_up:i2c_data<br>
&nbsp;&nbsp;&nbsp;&nbsp;:page_facing_up:irq_status<br>

## Limitations
 * HDMI CEC is not supported
 * HDMI Audio is not supported
 * HDCP is not supported
 * USB2.0 is not supported
 * Dongle onboard SPI EEPROM access via USB Mass Storage not implemented
 * Connecting more than one dongle to the same USB bus may not work
 * Non big-endian hosts (e.g. little endian) may not work
 * 32-bit hosts may not work

## Upstreaming
Considering, no firm decision yet

## Sources
 * Original driver by FrescoLogic: https://github.com/FrescoLogic/FL2000
 * Major clean-up of original FL driver by Hans Ulli Kroll: https://github.com/ulli-kroll/FL2000
 * Reference IT66121FN driver ftom RK3188 Android KK kernel repositpry: https://github.com/phjanderson/Kernel-3188
 * Reference USB DRM implementation of DisplayLink driver (see drivers/gpu/drm/udl)
 * Reference simple DRM implementation of PL111 driver (see drivers/gpu/drm/pl111)

## Notes
 * We can use simple DRM display pipeline (only prime plane -> crtc -> only one encoder)
 * FL2000DX outputs DPI interface (kind of "crtc" output, not "encoder")
 * VGA (D-Sub) DAC output of FL2000DX can be implemented as a DRM bridge (dumb_vga_dac)
 * Register access for both FL2000DX and IT66121 can be done via regmap (see <linux/regmap.h>)
 * For registration of sibling I2C devices of IT66121 (CEC, DDC, ...) i2c\_new\_dummy() function may be used
 * USB Bulk Streams are not supported by FL2000DX

## Open questions
 * How to simultaneously support HDMI bridge and D-Sub bridge? Config option?
 * I2C autodetection require non-empty class. Is it safe to introduce custom one? Need to ask community
