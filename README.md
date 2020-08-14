# Emb-Rust
Embedded Rust example for STM32F4-Discovery board.

The [Rust Discovery book](https://docs.rust-embedded.org/discovery/index.html) is an excellent introduction to embedded Rust.
The book uses the slighty different STM32**F3**-Discovery board, whereas this project uses the STM32**F4**-Discovery board.

The differences are as follows:
* MCU changes from Cortex-M3 to Cortex-M4
* Version of the debugging port (ST-Link)
* Hardware Abstraction Layer (HAL)

The memory map remains the same between the two boards.

## Setup

Using Ubuntu 20.04 LTS 

1. Install Rust using [Rustup](https://www.rust-lang.org/tools/install), update if required
1. Add the Cortex-M4 target to the Rust compiler 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$ rustup target add thumbv7em-none-eabi` 

3. Install the multi-architecture debugger gdb and open On-Chip debugger packages using:

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$ sudo apt-get install gdb-multiarch openocd`

4. Set udev rules for USB to enable access without root privilege
* Create the file `/etc/udev/rules.d/99-openocd.rules`

* Add the following lines to the file 
``` 
# STM32F4 - ST-Link/V2 
ATTRS{idVendor}=="0483", ATTRS{idProduct}=="3748", MODE:="0666" 
```

* Reload the rules with 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; `$ sudo udevadm control --reload-rules`

## Verify the board
Connect the discovery board ST-Link (USB mini B) port to the computer USB port and type:

```
$ lsusb | grep -i stm
Bus 001 Device 006: ID 0483:3748 STMicroelectronics ST-LINK/V2
```

Note that the Bus and Device numbers may differ, but the ID should be the same if you have a STM32F4-Discovery board.


### Verify the permissions
```
$ ls -l /dev/bus/usb/001/006
crw-rw-rw-+ 1 root plugdev 189, 5 Aug 13 14:48 /dev/bus/usb/001/006
```
As above the permissions should be `crw-rw-rw-`


# Build the application
Run the command:

`$ cargo build`

Note that this uses the `.cargo\config` file to set the target to `thumbv7em-none-eabi` for the Cortex-M4 processor on the STM32F4-discovery board.

Alternatively run the command:

`cargo build --target thumbv7em-none-eabi`

This build uses the memory map for the STM32F4-Discovery board recorded in the file `memory.x`.

# Flash the application

First open the OCD connection.

## Open OCD connection
Run the command:

`$ openocd -f interface/stlink-v2.cfg -f target/stm32f4x.cfg`


This should produce the following output:

```
Open On-Chip Debugger 0.10.0
Licensed under GNU GPL v2
For bug reports, read
        http://openocd.org/doc/doxygen/bugs.html
Info : auto-selecting first available session transport "hla_swd". To override use 'transport select <transport>'.
Info : The selected transport took over low-level target control. The results might differ compared to plain JTAG/SWD
adapter speed: 2000 kHz
adapter_nsrst_delay: 100
none separate
Info : Unable to match requested speed 2000 kHz, using 1800 kHz
Info : Unable to match requested speed 2000 kHz, using 1800 kHz
Info : clock speed 1800 kHz
Info : STLINK v2 JTAG v21 API v2 SWIM v0 VID 0x0483 PID 0x3748
Info : using stlink api v2
Info : Target voltage: 2.896991
Info : stm32f4x.cpu: hardware has 6 breakpoints, 4 watchpoints
```

Also, one of the red LEDs, the one closest to the USB port, should start oscillating between red light and green light.

Use control-c to quit the program.

### Connect GDB
With OCD running in another terminal run the following command:

`$ gdb-multiarch -q target/thumbv7em-none-eabi/debug/emb-rust`

You should see:
```
Registered pretty printers for UE4 classes
Reading symbols from target/thumbv7em-none-eabi/debug/emb-rust...
(gdb) 
```
Connect to OCD
```
(gdb) target remote :3333
Remote debugging using :3333
0x00000000 in ?? ()
(gdb)
```

Load the application
```
(gdb) load
Loading section .vector_table, size 0x1a8 lma 0x8000000
Loading section .text, size 0x4230 lma 0x80001a8
Loading section .rodata, size 0xa4c lma 0x80043e0
Start address 0x080041fe, load size 20004
Transfer rate: 14 KB/sec, 5001 bytes/write.
(gdb) 
```

## Debug the application

```
(gdb) break main
Breakpoint 1 at 0x80002d8: file src/main.rs, line 18.
(gdb) continue
Continuing.
Note: automatically using hardware breakpoints for read-only addresses.

Breakpoint 1, main () at src/main.rs:18
18      #[entry]
(gdb) 
```

You can use standard GDB commands at this point.

## Running the application
Disconnect the board from the computer, and attach the board to a power source.
The application will be automatically copied from flash and start running.