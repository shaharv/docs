# RISC-V Software Tools

This page summarizes the steps required for building and running RISC-V programs, using the GNU toolchain, LLVM, QEMU and Spike.  

## Table of contents

1. [Building and running C on RISC-V64D](#1-building-and-running-c-on-risc-v64d)  
1.1 [Installing GNU RISC-V toolchain](#installing-gnu-risc-v-toolchain)  
1.2 [Installing QEMU](#installing-qemu)  
1.3. [Building a C program](#building-a-c-program)  
1.4. [Running the program with QEMU](#running-the-program-with-qemu)  
1.5. [Compiling for RISC-V32IM](#compiling-for-risc-v32im)  
2. [Assembling RISC-V-V programs with llvm-mc](#2-assembling-risc-v-v-programs-with-llvm-mc)  
2.1 [llvm-mc](#llvm-mc)  
2.2 [llvm-objdump](#llvm-objdump)  
3. [RISC-V Simulation with Spike](#3-risc-v-simulation-with-spike)  
3.1 [Installing GNU RISC-V64 Toolchain](#installing-gnu-risc-v64-toolchain)  
3.2 [Build pk from source](#build-pk-from-source)  
3.3 [Build Spike from source](#build-spike-from-source)  
3.4 [Running Spike](#running-spike)  
4. [Useful links](#4-useful-links)

## 1. Building and running C on RISC-V64D

### Installing GNU RISC-V toolchain

The installation instructions were tested on Ubuntu 18.04 (WSL).

- Install GNU's cross compiler, bintools and libraries for `riscv64` cross compilation:  
  `sudo apt-get install gcc-8-riscv64-linux-gnu libc6-dev-riscv64-cross`

  Then, create required symlinks:

  ```
  sudo ln -s /usr/riscv64-linux-gnu/lib/ld-linux-riscv64-lp64d.so.1 /lib
  sudo ln -s /usr/bin/riscv64-linux-gnu-gcc-8 /usr/bin/riscv64-linux-gnu-gcc
  ```

  The first is for picking the dynamic linker from `/lib`, required for `qemu`.  
  The second is for having `gcc` point to `gcc-8`, needed for `pk`.  

- Install latest clang (version 10 as of May 2020):  
  `sudo bash -c "$(wget -O - https://apt.llvm.org/llvm.sh)"`

  On Ubuntu 20.04, this will do instead: `sudo apt-get install clang`
  
- **Note:** The GNU toolchain binaries are installed in the following locations:  
  - Compiler driver: `/usr/bin/riscv64-linux-gnu-gcc-8`
  - Assembler: `/usr/bin/riscv64-linux-gnu-as`
  - Linker: `/usr/bin/riscv64-linux-gnu-ld`
  - Libraries: `/usr/lib/riscv64-linux-gnu/`

### Installing QEMU

For building latest QEMU from source (5.0.0 as of May 2020) for `riscv64`:

- Install dependencies: https://wiki.qemu.org/Hosts/Linux
- Install Flex and Bison: `sudo apt-get install flex bison`
- Follow these steps:

```
mkdir ~/qemu && cd ~/qemu
wget https://download.qemu.org/qemu-5.0.0.tar.xz
tar -xf qemu-5.0.0.tar.xz
mkdir build && cd build
../qemu-5.0.0/configure --target-list=riscv64-softmmu,riscv64-linux-user
sudo make install
```

**Tip:** For a debug QEMU build, add `--enable-debug --disable-pie` to the configure line.

### Building a C program

- Using GCC:  
  `riscv64-linux-gnu-gcc-8 -static foo.c`

- Using Clang:  
  `clang-10 --target=riscv64-linux-gnu -static -isystem /usr/riscv64-linux-gnu/include foo.c`

- Using Clang and the experimental `V` extension:  
  `$LLVM_BIN/clang --target=riscv64-linux-gnu -menable-experimental-extensions -march=rv64ifdv0p9 -static -isystem /usr/riscv64-linux-gnu/include foo.c`

- Run `file a.out` to verify the resulting executable:  
  `./a.out: ELF 64-bit LSB executable, UCB RISC-V, version 1 (SYSV), statically linked, for GNU/Linux 4.15.0, not stripped`

- **Note:** The installed RISC-V toolchain is 64-bit, with the `D` extension (double precision floating point). This is "hard float", as opposed to "soft float" which means floats are emulated in software. The relevant ABI flag is `-mabi=lp64d`. It is implied, but could be specified in the command line for extra clarity.  

- Clang options explained:
  - `--target=riscv64-linux-gnu`: specify the target tuple. As opposed to gcc, clang is a generic cross compiler, so we use the host clang compiler and specify the target to compile to.  
  - `-static`: link libraries statically into the final executable.  
  - `-isystem /usr/riscv64-linux-gnu/include`: point clang towards GNU riscv64 headers.  

### Running the program with QEMU

For running the `riscv64` program:

`qemu-riscv64 ./a.out`

### Compiling for RISC-V32IM

With the `riscv64` toolchain, we can't build and run 32-bit RISC-V, but we can compile simple functions into 32-bit RISC-V assembly.  
The influential compiler flags are `-march=rv32im` and `-mabi=ilp32`.  
For clang, we should also specify `--target=riscv32`.  

GCC command line:  
- `riscv64-linux-gnu-gcc-8 -march=rv32im -mabi=ilp32 -O3 -S foo.c`

Clang command line:  
- `clang-10 --target=riscv32 -march=rv32im -mabi=ilp32 -O3 -S foo.c`


## 2. Assembling RISC-V-V programs with llvm-mc

Assembling RISC-V programs of ISA `RV32IV`:  
- `RV32I` - the basic RISC-V instruction set with 32-bit integer operations  
- `V` - the experimental vector extension.  

### llvm-mc

For assembling RV32IV programs and displaying encodings:

```
$LLVM_BIN/llvm-mc -triple=riscv32 --mattr=+experimental-v -show-encoding foo.asm
```

For assembling into an object:

```
$LLVM_BIN/llvm-mc -triple=riscv32 --mattr=+experimental-v --filetype=obj foo.asm > foo.o
```

### llvm-objdump

For disassembling the generated object with `llvm-objdump`:  

```
$LLVM_BIN/llvm-objdump --arch=riscv64 --mattr=+experimental-v -d foo.o
```


## 3. RISC-V Simulation with Spike

[Spike](https://github.com/riscv/riscv-isa-sim) is a the original RISC-V ISA simulator implementation.  
It is an instruction accurate implementation (ISS), as opposed to cycle accurate (CAS).  
Currently, it is the only free RISC-V simulator implementation supporting the V extension (vector instructions).

### Installing GNU RISC-V64 Toolchain

The installation instructions were tested on Ubuntu 18.04 (WSL).

First, install a RISC-V toolchain, as described [here](#installing-gnu-risc-v-toolchain).  
At minimum, install the GNU toolchain (`gcc-8-riscv64-linux-gnu`). Clang is optional.

It is recommended to add the toolchain binaries location to `PATH`, in `.bashrc` for example:

```
export PATH=$PATH:/usr/riscv64-linux-gnu/bin
```

### Build pk from source

From [`pk` homepage](https://github.com/riscv/riscv-pk):

>pk is a lightweight application execution environment that can host statically-linked RISC-V ELF binaries. It is designed to support tethered RISC-V implementations with limited I/O capability and thus handles I/O-related system calls by proxying them to a host computer.

Suggested steps for installing `pk`:

```
mkdir ~/src/riscv-pk && cd ~/src/riscv-pk
git clone https://github.com/riscv/riscv-pk
mkdir build && cd build
../riscv-pk/configure --prefix=/usr --host=riscv64-linux-gnu
make -j
sudo make install
```

**Note:** The `pk` code is cross compiled to RISC-V, using the installed `riscv64-linux-gnu` toolchain.  

### Build Spike from source

Suggested steps for installing Spike:

```
sudo apt-get install device-tree-compiler
mkdir ~/src/riscv-isa-sim && cd ~/src/riscv-isa-sim
git clone https://github.com/riscv/riscv-isa-sim.git
mkdir build && cd build
../riscv-isa-sim/configure --prefix=/usr/riscv64-linux-gnu
make -j
sudo make install
```

### Running Spike

- For building a C program, see [here](#building-a-c-program).

- Run Spike: `spike /usr/riscv64-linux-gnu/bin/pk ./a.out`

- Spike command line options:  
  - `--help` - list all available options.
  - `-d` - run in interactive debug mode.

- Spike interactive mode options:  
  - `help` - list all options.  
  - `reg 0` - print all registers of core `0`.
  - `pc 0` - print the PC of core `0`.
  - `r 1000` - run 1000 instructions while printing them.  
  - `rs 1000` - run 1000 instructions, silently.  

## 4. Useful links

- Debian Wiki - RISC-V  
  https://wiki.debian.org/RISC-V

- GCC RISC-V options  
  https://gcc.gnu.org/onlinedocs/gcc/RISC-V-Options.html

- RISC-V software tools presentation (2015)  
  https://riscv.org/wp-content/uploads/2015/02/riscv-software-stack-tutorial-hpca2015.pdf

- Spike homepage  
  https://github.com/riscv/riscv-isa-sim

- Tutorial on Spike internals  
  https://github.com/poweihuang17/Documentation_Spike

- List of RISC-V Simulators  
  https://riscv.org/software-status/#simulators

- RISC-V Vector Extensions spec drafts  
  https://github.com/riscv/riscv-v-spec/releases
