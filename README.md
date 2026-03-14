# Lab 2: "151T Forge" Sodor Repository Setup

## Introduction

In lecture sessions, we talked about Chipyard, Rocket Chip, various RISC-V cores, etc.. And in Lab 1, you learned Chisel. We are now going to use Chisel and System Verilog to setup an SoC repository for the 151T tapeout called "151T Forge". 

Forge is going to be a teaching version of the integrated SoC development platform [Chipyard](https://github.com/ucb-bar/chipyard). This is a Spring 2026 experiment driven by feedback to integrate System Verilog usage, remove dependency on Chipyard (which some find quite complex), and integrate IP from scratch. 

Since this is an experiment, you are encouraged to explore your own ideas, provide feedback, submit Github pull requests, and document errors on the Discord. Everybody will work together on the same repository, creating branches and pull requests as described in Lab 0. Do NOT force push to the main branch even if settings do not prevent you from it. 

The [Sodor project](https://github.com/ucb-bar/riscv-sodor?tab=readme-ov-file) provides multiple pipelined implementations (1-, 2-, 3-, 5-stage, and microcoded) of a RISC-V core written in Chisel. You can choose any of the 1-5 stage cores to start.

As the Lab 2 deliverable, you will run through the repository setup. Then you will run existing assembly (RISC-V) tests on the Sodor core. Your submisison is the core passing all tests. Staff will work in parallel to provide guidance for the follow-up tasks, currently listed as "next steps," but you're welcome to jump ahead. 

Later you will add peripherals, run verification, and take the Sodor RTL through physical-design (PD) flows in Sky130 to tapeout.


## Pixi


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


i've attached a image of what setup successfully should look like (asm tests pass on any of the 1-5 stage sodor cores)

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