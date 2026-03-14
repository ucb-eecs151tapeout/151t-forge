# 151t forge

https://pixi.prefix.dev/latest/installation/

`$151t-forge/ pixi install`

https://pixi.prefix.dev/latest/advanced/pixi_shell/#shell-completions

# build


## spike
```bash
cd ${forge}/tools/riscv-isa-sim

mkdir build

cd build

../configure --prefix=$RISCV

make -j32

make -j32 install

which spike # should resolve to ${forge}/.pixi/envs/default/riscv-tools/bin/spike

```

## fesvr
* we are on an older version of fesvr... install here: https://github.com/ucb-eecs151tapeout/riscv-fesvr-sodor
   * it'll install on top of the fesvr installed by the riscv-isa-sim (spike) dir


## riscv tests
```bash

cd ${forge}/tests/riscv-tests

git submodule update --init --recursive

autoconf

./configure --prefix=$RISCV/target

make -j32 XLEN=32 # XLEN=32 for rv32 isa tests only

make -j32 install
```

## running asm tests
```bash
cd ${forge}/sims/verilator

make rtl

make emulator

make run-asm-tests

# output logs in ${forge}/sims/verilator/output

```

## waveforms
```bash
cd ${forge}/sims/verilator

make rtl

make emulator-debug

make run-asm-tests-debug

# waveforms in ${forge}/sims/verilator/output

```

# problems
* sodor uses a custom reset -- by writing to 0x44
   * https://github.com/riscv-software-src/riscv-isa-sim/blob/90a04da3f9235e17ab456b9263080f2dc32e4f1e/fesvr/dtm.cc 
   * this should get updated so that sodor supports resuming execution from `0x7b1` (dpc) [spec compliant] after coming out of debug mode.

* cannot build riscv bmark test in rv32 due to ucb-bar riscv toolchain (feedstock) not supporting multi-lib - https://github.com/ucb-bar/riscv-tools-feedstock/pull/8 and the lack of a rv32 toolchain thats been compiled from source (cs152 has one, but ideally we don't depend on that, and we build from scratch)

* when run it creates a bunch of "rv32_2stage", "rv32_1stage" etc folders in `${forge}/generators/riscv-sodor/rv32_1stage`.. these shouldnt be made, its done somewhere in the makefile

# things that need to be done before tapeout
* jtag TAP to DMI unit - https://chipyard.readthedocs.io/en/stable/Advanced-Concepts/Chip-Communication.html
* replace 3 stage's 1 cycle mem w/ srams
* IO
* bump to chisel 3, java 17 or smth, not java 8 (1.8.0), bump scala version from 2.12 to latest CY version
* whats the modern version of chisel-iotesters? move to that
* figure out the custom reset/custom fesvr (to write to 0x44 to start exec) situation, and whether its even viable to keep that around?? (will we successfully bringup if its like that)




# random
https://fires.im/micro19-slides-pdf/02_chipyard_basics.pdf