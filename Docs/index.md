#Debugging with CMSIS-DAP

mbed uses CMSIS-DAP as the protocol for debugging, but you need something on your host computer to talk to CMSIS-DAP, for example, [Keil uVision IDE](http://www.keil.com/uvision/) or [pyOCD](https://github.com/mbedmicro/pyOCD) with GDB. These documents will guide you through:

1. Debugging with [printf() calls](Debugging/printf.md).
1. Debugging with [pyOCD and GDB](Debugging/pyOCD.md).
1. Debugging with [Keil uVision](Debugging/Keil.md).
1. Debugging the [micro:bit with pyOCD and GDB](Debugging/debugging_microbit.md)

## Links to other sources

Our blog has [an article about using CMSIS-DAP to debug a device after it crashes](http://blog.mbed.com/post/139539984822/debugging-a-crashed-device-with-cmsis-dap).
