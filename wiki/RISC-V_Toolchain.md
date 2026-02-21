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

The installation instructions were tested on Ubuntu 24.04 (WSL).

- Install GNU's cross compiler, bintools and libraries for `riscv64` cross compilation:
  `sudo apt-get install gcc-13-riscv64-linux-gnu libc6-dev-riscv64-cross`

  Then, create required symlinks:

  ```
  sudo ln -s /usr/riscv64-linux-gnu/lib/ld-linux-riscv64-lp64d.so.1 /lib
  sudo ln -s /usr/bin/riscv64-linux-gnu-gcc-13 /usr/bin/riscv64-linux-gnu-gcc
  ```

  The first is for picking the dynamic linker from `/lib`, required for QEMU.
  The second is for having `gcc` point to `gcc-13`, needed for `pk`.

- Install clang (version 18 on Ubuntu 24.04): `sudo apt-get install clang-18`.  
  For the latest LLVM version, use the official script: `sudo bash -c "$(wget -O - https://apt.llvm.org/llvm.sh)"`

- **Note:** The GNU toolchain binaries are installed in the following locations:
  - Compiler driver: `/usr/bin/riscv64-linux-gnu-gcc-13`
  - Assembler: `/usr/bin/riscv64-linux-gnu-as`
  - Linker: `/usr/bin/riscv64-linux-gnu-ld`
  - Libraries: `/usr/lib/riscv64-linux-gnu/`

### Installing QEMU

For building latest QEMU from source (10.2.0 as of December 2025) for `riscv64`:

- Install dependencies: https://wiki.qemu.org/Hosts/Linux
- Install required packages:
```
sudo apt-get install -y bison bzip2 flex git libglib2.0-dev libpixman-1-dev make ninja-build python3-venv wget xz-utils
```
- Follow these steps:

```
mkdir ~/qemu && cd ~/qemu
wget https://download.qemu.org/qemu-10.2.0.tar.xz
tar -xf qemu-10.2.0.tar.xz
mkdir build && cd build
../qemu-10.2.0/configure --target-list=riscv64-softmmu,riscv64-linux-user
make -j
sudo make install
```

**Tip:** For a debug QEMU build, add `--enable-debug --disable-pie` to the configure line.

### Building a C program

- Using GCC:
  `riscv64-linux-gnu-gcc-13 -static foo.c`

- Using Clang:
  `clang-18 --target=riscv64-linux-gnu -static -isystem /usr/riscv64-linux-gnu/include foo.c`

- Using Clang with the `V` extension (ratified in LLVM 17+):
  `clang-18 --target=riscv64-linux-gnu -march=rv64gcv -static -isystem /usr/riscv64-linux-gnu/include foo.c`

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

With the `riscv64` toolchain, we can't build and run 32-bit RISC-V, but we can compile simple freestanding functions into 32-bit RISC-V assembly.
The influential compiler flags are `-march=rv32im` and `-mabi=ilp32`.
For clang, we should also specify `--target=riscv32`.

**Note:** Ubuntu's `libc6-dev-riscv64-cross` package only includes 64-bit (LP64) headers. The 32-bit ILP32 headers are not available, so code must be freestanding (no libc includes like `<stdio.h>`).

Example freestanding function:
```c
// foo.c
int add(int a, int b) {
    return a + b;
}
```

GCC command line:
- `riscv64-linux-gnu-gcc-13 -march=rv32im -mabi=ilp32 -ffreestanding -O3 -S foo.c`

Clang command line:
- `clang-18 --target=riscv32 -march=rv32im -mabi=ilp32 -ffreestanding -O3 -S foo.c`


## 2. Assembling RISC-V-V programs with llvm-mc

Assembling RISC-V programs of ISA `RV32IV`:  
- `RV32I` - the basic RISC-V instruction set with 32-bit integer operations  
- `V` - the experimental vector extension.  

### llvm-mc

For assembling RV32IV programs and displaying encodings:

```
llvm-mc-18 -triple=riscv32 --mattr=+experimental-v -show-encoding foo.s
```

For assembling into an object:

```
llvm-mc-18 -triple=riscv32 --filetype=obj foo.s > foo.o
```

### llvm-objdump

For disassembling the generated object with `llvm-objdump`:  

```
llvm-objdump-18 --arch=riscv64 -d foo.o
```


## 3. RISC-V Simulation with Spike

[Spike](https://github.com/riscv/riscv-isa-sim) is the original RISC-V ISA simulator implementation.
It is an instruction accurate implementation (ISS), as opposed to cycle accurate (CAS).

### Installing GNU RISC-V64 Toolchain

The installation instructions were tested on Ubuntu 24.04 (WSL).

First, install a RISC-V toolchain, as described [here](#installing-gnu-risc-v-toolchain).
At minimum, install the GNU toolchain (`gcc-13-riscv64-linux-gnu`). Clang is optional.

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

- Spike homepage  
  https://github.com/riscv/riscv-isa-sim

- Tutorial on Spike internals  
  https://github.com/poweihuang17/Documentation_Spike

- RISC-V Vector Extensions spec drafts  
  https://github.com/riscv/riscv-v-spec/releases

- RVV Intrinsics API specs  
  https://github.com/riscv-non-isa/rvv-intrinsic-doc/releases

- rvv-benchmark: testing of the vector assembly code examples come from riscv/riscv-v-spec/example  
  https://github.com/plctlab/rvv-benchmark
