# Building and running ARM Cortex-M0

## Installation

The installation instructions were tested on Ubuntu 18.04 (WSL).

### Toolchain

- Install [`gcc-arm-none-eabi`](https://packages.ubuntu.com/bionic/gcc-arm-none-eabi): `sudo apt-get install gcc-arm-none-eabi`.

- This handily pulls in the bintools and standard library packages:  
  - [`binutils-arm-none-eabi`](https://packages.ubuntu.com/bionic/libs/binutils-arm-none-eabi)
  - [`libnewlib-arm-none-eabi`](https://packages.ubuntu.com/bionic/libs/libnewlib-arm-none-eabi)
  - [`libstdc++-arm-none-eabi-newlib`](https://packages.ubuntu.com/bionic/devel/libstdc++-arm-none-eabi-newlib)

- The toolchain is installed in the following locations:  
  - Compiler driver: `/usr/bin/arm-none-eabi-gcc`
  - Assembler: `/usr/lib/arm-none-eabi/bin/as`
  - Linker: `/usr/bin/arm-none-eabi-ld`
  - Libraries: `/usr/lib/arm-none-eabi/`

### QEMU

For building latest QEMU from source (5.0.0 as of May 2020):

- Install dependencies: https://wiki.qemu.org/Hosts/Linux
- Install Flex and Bison: `sudo apt-get install flex bison`
- Follow these steps:

```
mkdir ~/qemu && cd ~/qemu
wget https://download.qemu.org/qemu-5.0.0.tar.xz
tar -xf qemu-5.0.0.tar.xz
mkdir build && cd build
../qemu-5.0.0/configure --target-list=arm-softmmu,arm-linux-user
sudo make install
```

## Building a C program

- For cross compiling to ARM Cortex, use the following command line:  
  `arm-none-eabi-gcc foo.c -mcpu=cortex-m0 --specs=rdimon.specs`

  `-mcpu=cortex-m0` specifies the target processor.  
  `--specs=rdimon.specs` allows for semihosting - interacting with the host machine, like printing to the console.  
  For baremetal, we should add `--specs=nosys.specs`.

- Run `file a.out` to verify the resulting executable:  
  `a.out: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), statically linked, with debug_info, not stripped`

## Running the program with QEMU

For running a ARM Cortex-M0 program:

`qemu-arm ./a.out -cpu cortex-m0`

## Useful links

- GCC ARM options
  https://gcc.gnu.org/onlinedocs/gcc/ARM-Options.html
