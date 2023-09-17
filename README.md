# CJMCU-232H-based JTAG, SWD, and AVR Programmer

This is a very popular module based on the FT232H chip made by CJMCU.
You can use this module as a high-speed programmer for your embedded projects.
I discovered that this module can be used as a JTAG and SWD programmer for ESP32 and STM32 microcontrollers.
It can also be used as an ISP programmer for AVR microcontrollers.

This module might be used to program other microcontrollers, but no information about them has been published yet. 

![CJMCU-232H](https://user-images.githubusercontent.com/1549028/219136649-b83fe1e2-26ae-417c-9f38-46d55629f4f6.jpg)

Any similar module/board based on FT233H/FT2232H which exposes `AD0`, `AD1`, `AD1`, and `AD3` pins can be used for this purpose.

**Note that voltage level of all I/Os is 3.3V but they are 5V-tolerant**

![](CJMCU-FT232HQ-PROGRAMMER.png)

### OpenOCD (for JTAG & SWD)

Install `WinUSB` driver for FT232H using [Zadig](https://zadig.akeo.ie/):

![image](https://github.com/m3y54m/cjmcu-ft232hq-programmer/assets/1549028/8ebb5a65-fc5d-45e3-a089-9c635e0c701a)

![image](https://github.com/m3y54m/cjmcu-ft232hq-programmer/assets/1549028/88c1b160-3ca1-4c9e-86ff-ac882d59347a)


```bash
openocd -f interface/ftdi/ft232h-module-swd.cfg -f board/stm32f103c8_blue_pill.cfg
```

**Read flash:**

For STM32f103C8 with 64KB of flash. Target is `board/stm32f103c8_blue_pill.cfg`. Size to read is `0x10000`. Read firmware as `firmware.bin`.

```bash
openocd -f interface/ftdi/ft232h-module-swd.cfg -f board/stm32f103c8_blue_pill.cfg -c init -c "reset halt" -c "flash read_bank 0 firmware.bin 0 0x10000" -c "reset" -c shutdown
```

Write flash:

For STM32f103C8 with 64KB of flash. Target is `board/stm32f103c8_blue_pill.cfg`. Write firmware `firmware.bin` to flash.

```bash
openocd -f interface/ftdi/ft232h-module-swd.cfg -f board/stm32f103c8_blue_pill.cfg  -c init -c "reset halt" -c "flash write_image erase firmware.bin 0x08000000" -c "reset" -c shutdown
```

Using with PlatformIO:

```
[env:bluepill_f103c8_128k]
platform = ststm32
board = bluepill_f103c8_128k
framework = arduino
; Upload options for FT232H USB-JTAG adapter in SWD Mode
upload_protocol = custom
upload_command = ${platformio.packages_dir}/tool-openocd/bin/openocd -s ${platformio.packages_dir}/tool-openocd/scripts -f interface/ftdi/ft232h-module-swd.cfg -f board/stm32f103c8_blue_pill.cfg -c "init" -c "reset halt" -c "flash write_image erase {$SOURCE} 0x08000000" -c "reset" -c "shutdown"
; Debug options for FT232H USB-JTAG adapter in SWD Mode
debug_tool = custom
debug_server =
  ${platformio.packages_dir}/tool-openocd/bin/openocd
  -s
  ${platformio.packages_dir}/tool-openocd/scripts
  -f
  interface/ftdi/ft232h-module-swd.cfg
  -f
  board/stm32f103c8_blue_pill.cfg
```

Using JLink + SWD:

```bash
openocd -f interface/jlink.cfg -c "transport select swd" -f board/stm32f103c8_blue_pill.cfg
```

### AVRDUDE (for AVR ISP)

Install default driver from FTDI. If you have installed `WinUSB` or `libusb` driver for FT232H, you should revert it to defalut driver:

![image](https://github.com/m3y54m/cjmcu-ft232hq-programmer/assets/1549028/769b44e2-633e-455d-8b71-699d58a02b60)

![image](https://github.com/m3y54m/cjmcu-ft232hq-programmer/assets/1549028/1b76b404-82dc-4795-baf4-731dbf7b121c)

![image](https://github.com/m3y54m/cjmcu-ft232hq-programmer/assets/1549028/4eaf5fbf-3a84-43d3-bc39-9f7bd1b20329)

![image](https://github.com/m3y54m/cjmcu-ft232hq-programmer/assets/1549028/9d0e970d-a585-4761-bab1-f289c5497622)

![image](https://github.com/m3y54m/cjmcu-ft232hq-programmer/assets/1549028/ef8a3db8-d489-4a2e-a0ee-c84775525ba1)

![image](https://github.com/m3y54m/cjmcu-ft232hq-programmer/assets/1549028/39646de2-7604-4e26-a589-cbb9ac1bc5b6)

## Resources

- [Low-cost ESP32 In-circuit Debugging](https://medium.com/@manuel.bl/low-cost-esp32-in-circuit-debugging-dbbee39e508b)
- [Configuring JTAG debugging in Linux](https://nodemcu.readthedocs.io/en/dev-esp32/debug/)
- [Getting Started with OPENOCD Using FT2232H Adapter for SWD Debugging](https://www.allaboutcircuits.com/technical-articles/getting-started-with-openocd-using-ft2232h-adapter-for-swd-debugging/)
- [How to use ft2232h adapter and openocd to debug the SWD interface of arm cortex M Series MCU](https://archive.ph/y9BLG)
- [Read and write stm32 firmware binary with openOCD](https://gist.github.com/vanbwodonk/6b425a093786e7eaddf535de747118dc)
- [AVRDUDE and FTDI *232H](http://www.jdunman.com/ww/AmateurRadio/SDR/helix_air_net_au%20%20AVRDUDE%20and%20FTDI%20232H.htm)
- [TinyAVR-0/1 Programming](https://semjonov.de/docs/tips/avr/)
