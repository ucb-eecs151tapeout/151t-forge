# 151t forge

https://pixi.prefix.dev/latest/installation/

$151t-forge/ pixi install

https://pixi.prefix.dev/latest/advanced/pixi_shell/#shell-completions




```
repo/
├─ src/
│  ├─ common/
│  ├─ rv32_1stage/
│  ├─ rv32_2stage/
│  ├─ rv32_3stage/
│  ├─ rv32_5stage/
│  └─ rv32_ucode/
│
├─ emulator/
│  └─ common/
│
├─ vsrc/
│
├─ tools/
│  └─ riscv-isa-sim/
│
└─ sims/
   └─ verilator/
      ├─ Makefile
      ├─ generated-src/
      └─ output/

```



cd riscv-tests
autoconf
./configure
make -C isa XLEN=32
./configure --prefix="$(pwd)/build"


make run MK_TARGET_PROC=rv32_3stage BINARY=/scratch/jfx/151t-forge/tests/riscv-tests/build/share/riscv-tests/isa/rv32ui-v-or

