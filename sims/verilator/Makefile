all: emulator

RISCV           ?= /opt/riscv  # prioritize $RISCV env variable
srcDir          := $(abspath ../../generators/riscv-sodor)
fesvrDir        := $(abspath ../../tools/riscv-fesvr)
buildLibDir     := $(RISCV)/lib
buildDir        := $(abspath ./build)

CXX := g++
SBT := sbt $(SBT_FLAGS)

.PHONY: all rtl clean

# common  := $(srcDir)/sim
# install     := $(srcDir)/install

# ELF         := elf2hex

# pipe processor output through the spike-dasm program to turn 
# raw instruction bits into pretty, readable diassembly
disasm_exe := 2>
which_disasm := $(shell which spike-dasm)
ifneq ($(which_disasm),)
	disasm_exe := 3>&1 1>&2 2>&3 | $(which_disasm) $(DISASM_EXTENSION) >
endif

MK_TARGET_PROC ?= rv32_1stage


generated_dir = $(buildDir)/generated-src/${MK_TARGET_PROC}

rtl: $(generated_dir)/Top.v
 
# ifneq ($(shell which pkg-config),)
# 	ifneq ($(shell pkg-config --variable=includedir verilator),)
# 		VERILATOR_INCLUDE=$(shell pkg-config --variable=includedir verilator)
# 	else
# 		ifneq ($(wildcard $(VERILATOR_ROOT)/include/vltstd/vpi_user.h),)
# 			VERILATOR_INCLUDE=$(VERILATOR_ROOT)/include/
# 		else
# 			$(error VERILATOR_ROOT is set to $(VERILATOR_ROOT) but not finding expected_file in there.)
# 		endif
# 	endif
# else
# 	ifneq ($(wildcard $(VERILATOR_ROOT)/include/vltstd/vpi_user.h),)
# 		VERILATOR_INCLUDE=$(VERILATOR_ROOT)/include/
# 	else
# 		$(error VERILATOR_ROOT is set to $(VERILATOR_ROOT) but not finding expected_file in there.)
# 	endif
# endif

ifneq ($(shell which pkg-config 2>/dev/null),)
VERILATOR_CFLAGS := $(shell pkg-config --cflags verilator 2>/dev/null)
endif

ifeq ($(strip $(VERILATOR_CFLAGS)),)
ifeq ($(strip $(VERILATOR_ROOT)),)
VERILATOR_ROOT := $(shell verilator -V 2>/dev/null | sed -n 's/^VERILATOR_ROOT *= *//p')
endif

ifneq ($(wildcard $(VERILATOR_ROOT)/include/vltstd/vpi_user.h),)
VERILATOR_CFLAGS := -I$(VERILATOR_ROOT)/include -I$(VERILATOR_ROOT)/include/vltstd
else
$(error Could not find Verilator headers. Set VERILATOR_ROOT or make pkg-config verilator work.)
endif
endif
 
CXXFLAGS += -O1 -std=c++17 -g

CXXSRCS := SimDTM
CXXFLAGS += -I$(srcDir)/sim -I$(fesvrDir)
CXXFLAGS += $(VERILATOR_CFLAGS)
CXXDEBUGFLAGS := -DDEBUG

LDFLAGS := $(LDFLAGS) -L$(buildLibDir) -Wl,-rpath,$(buildLibDir) -L. -lpthread
# OBJS := $(addsuffix .o,$(CXXSRCS))
OBJS := $(addprefix $(buildDir)/,$(addsuffix .o,$(CXXSRCS)))
# DEBUG_OBJS := $(addsuffix -debug.o,$(CXXSRCS))
DEBUG_OBJS := $(addprefix $(buildDir)/,$(addsuffix -debug.o,$(CXXSRCS)))

CHISEL_ARGS := -td emulator/${MK_TARGET_PROC}/generated-src
CHISEL_ARGS_DEBUG := 

# To find emulator.cpp
vpath %.cpp  $(srcDir)/sim
vpath %.cc 	$(fesvrDir)/fesvr $(srcDir)/sim
vpath %.h 	$(fesvrDir)/fesvr $(srcDir)/sim
vpath %.a    $(buildLibDir) 
vpath %.v 	$(srcDir)/vsrc
vpath %.sv 	$(srcDir)/vsrc

# build/generated-src/Top.v: $(srcDir)/src/${MK_TARGET_PROC}/*.scala $(srcDir)/src/common/*.scala
# 	cd $(buildDir) && $(SBT) "project ${MK_TARGET_PROC}" "run $(CHISEL_ARGS)"

# generated-src/Top.v: \
#   $(wildcard $(srcDir)/src/common/*.scala) \
#   $(wildcard $(srcDir)/src/$(MK_TARGET_PROC)/*.scala)
# 	mkdir -p $(buildDir) && cd $(buildDir) && $(SBT) "project $(MK_TARGET_PROC)" "run $(CHISEL_ARGS)"

$(generated_dir)/Top.v: \
  $(wildcard $(srcDir)/src/common/*.scala) \
  $(wildcard $(srcDir)/src/$(MK_TARGET_PROC)/*.scala)
	mkdir -p $(buildDir)/generated-src/$(MK_TARGET_PROC)
	cd $(srcDir) && $(SBT) "project $(MK_TARGET_PROC)" "run --target-dir $(buildDir)/generated-src/$(MK_TARGET_PROC)"

# $(addsuffix .o,$(CXXSRCS)): %.o: $(srcDir)/sim/%.cc
# 	$(CXX) $(CXXFLAGS) -I${VERILATOR_INCLUDE} -c -o $@ $<

# $(buildDir)/%.o: $(srcDir)/sim/%.cc
# 	mkdir -p $(buildDir)
# 	$(CXX) $(CXXFLAGS) -I$(VERILATOR_INCLUDE) -c -o $@ $<

$(buildDir)/%.o: $(srcDir)/sim/%.cc
	mkdir -p $(buildDir)
	$(CXX) $(CXXFLAGS) -c -o $@ $<

# Run Verilator to produce a fast binary to emulate this circuit.
VERILATOR := verilator --cc --exe --timing
VERILATOR_FLAGS := --top-module Top \
  +define+PRINTF_COND=\$$c\(\"verbose\"\) \
  --assert \
  --output-split 20000 \
	--x-assign unique \
  -I$(srcDir)/vsrc \
  -Mdir $(buildDir)/obj_dir \
  -O3 -CFLAGS "$(CXXFLAGS) $(LDFLAGS) -DVERILATOR -include $(srcDir)/sim/verilator.h"
emulator: $(generated_dir)/Top.v $(srcDir)/sim/emulator.cpp $(buildDir)/SimDTM.o $(srcDir)/vsrc/AsyncReadMem.sv $(srcDir)/vsrc/SyncMem.sv $(srcDir)/vsrc/SimDTM.v
	$(VERILATOR) $(VERILATOR_FLAGS) \
	-o $(buildDir)/$@ $< $(srcDir)/vsrc/SimDTM.v -LDFLAGS "$(LDFLAGS)" $(srcDir)/sim/emulator.cpp $(fesvrDir)/build/libfesvr.so $(buildDir)/SimDTM.o
	$(MAKE) -C $(buildDir)/obj_dir -f VTop.mk

emulator-debug: $(generated_dir)/Top.v $(srcDir)/sim/emulator.cpp $(buildDir)/SimDTM.o $(srcDir)/vsrc/AsyncReadMem.sv $(srcDir)/vsrc/SyncMem.sv $(srcDir)/vsrc/SimDTM.v
	$(VERILATOR) $(VERILATOR_FLAGS) --trace \
	-o $(buildDir)/$@ $< $(srcDir)/vsrc/SimDTM.v -LDFLAGS "$(LDFLAGS)" $(srcDir)/sim/emulator.cpp $(fesvrDir)/build/libfesvr.so $(buildDir)/SimDTM.o
	$(MAKE) -C $(buildDir)/obj_dir -f VTop.mk
 
clean:
	rm -rf $(buildDir)/obj_dir
	rm -rf $(buildDir)/*.o
	rm -rf $(buildDir)/emulator*
	rm -rf output

distclean:
	rm -rf $(buildDir)
	rm -rf output
	rm -rf SimDTM.o

#--------------------------------------------------------------------
# Tests
#--------------------------------------------------------------------

# Globally installed assembly tests
global_tstdir = $(RISCV)/target/share/riscv-tests/isa

global_asm_tests += \
	rv32ui-p-simple \
	rv32ui-p-add \
	rv32ui-p-addi \
	rv32ui-p-auipc \
	rv32ui-p-fence_i \
	rv32ui-p-sb \
	rv32ui-p-sh \
	rv32ui-p-sw \
	rv32ui-p-and \
	rv32ui-p-andi \
	rv32ui-p-beq \
	rv32ui-p-bge \
	rv32ui-p-bgeu \
	rv32ui-p-blt \
	rv32ui-p-bltu \
	rv32ui-p-bne \
	rv32ui-p-jal \
	rv32ui-p-jalr \
	rv32ui-p-lb \
	rv32ui-p-lbu \
	rv32ui-p-lh \
	rv32ui-p-lhu \
	rv32ui-p-lui \
	rv32ui-p-lw \
	rv32ui-p-or \
	rv32ui-p-ori \
	rv32ui-p-sll \
	rv32ui-p-slli \
	rv32ui-p-slt \
	rv32ui-p-slti \
	rv32ui-p-sra \
	rv32ui-p-srai \
	rv32ui-p-sub \
	rv32ui-p-xor \
	rv32ui-p-xori \
	rv32mi-p-sbreak \
	rv32mi-p-scall \
	rv32mi-p-ma_fetch \
	rv32mi-p-ma_addr \
	rv32mi-p-csr \
	rv32mi-p-illegal \

# Globally installed benchmarks
global_bmarkdir = $(RISCV)/target/share/riscv-tests/benchmarks
global_bmarks = \
	median \
	multiply \
	qsort \
	rsort \
	towers \
	vvadd \
	dhrystone

#--------------------------------------------------------------------
# Run assembly tests and benchmarks
#--------------------------------------------------------------------
global_asm_out = $(addsuffix .out, $(global_asm_tests))
global_asm_vcd = $(addsuffix .vcd, $(global_asm_tests))
    
global_bmarks_out = $(addsuffix .riscv.out, $(global_bmarks))
global_bmarks_vcd = $(addsuffix .riscv.vcd, $(global_bmarks))
global_bmarks_outgz = $(addsuffix .riscv.out.gz, $(global_bmarks))

asm_test_timeout ?=30000
bmark_timeout ?=1000000
                           
output:
	mkdir -p $@

$(addprefix output/, $(global_asm_vcd)): output/%.vcd: $(global_tstdir)/% emulator-debug
	mkdir -p output
	$(buildDir)/emulator-debug -v$@ +max-cycles=$(asm_test_timeout) +verbose +loadmem=$< 2> /dev/null

$(addprefix output/, $(global_asm_out)): output/%.out: $(global_tstdir)/% emulator
	mkdir -p output
	$(buildDir)/emulator +max-cycles=$(asm_test_timeout) +verbose +loadmem=$< none $(disasm_exe) $@

$(addprefix output/, $(global_bmarks_out)): output/%.riscv.out: $(global_bmarkdir)/%.riscv emulator
	mkdir -p output
	$(buildDir)/emulator +max-cycles=$(bmark_timeout) +verbose +loadmem=$< none $(disasm_exe) $@
	
$(addprefix output/, $(global_bmarks_vcd)): output/%.riscv.vcd: $(global_bmarkdir)/%.riscv emulator-debug
	mkdir -p output
	$(buildDir)/emulator-debug -v$@ +max-cycles=$(bmark_timeout) +verbose +loadmem=$< 2> /dev/null

run-asm-tests: $(addprefix output/, $(addsuffix .out, $(global_asm_tests)))
	@echo; perl -ne 'print "  [$$1] $$ARGV \t$$2\n" if /\*{3}(.{8})\*{3}(.*)/' $^; echo;

run-bmarks-test: $(addprefix output/, $(addsuffix .riscv.out, $(global_bmarks)))
	-@echo; perl -ne 'print "  [$$1] $$ARGV \t$$2\n" if /\*{3}(.{8})\*{3}(.*)/' $^; echo;

run-asm-tests-debug: $(addprefix output/, $(addsuffix .vcd, $(global_asm_tests)))
	@echo
	@printf '%s\n' $^ | sed 's#^#  [ PASSED ] #' 
	@echo

run-bmarks-test-debug: $(addprefix output/, $(addsuffix .riscv.vcd, $(global_bmarks)))
	@echo
	@printf '%s\n' $^ | sed 's#^#  [ PASSED ] #'
	@echo

run: run-asm-tests run-bmarks-test
run-debug: run-asm-tests-debug run-bmarks-test-debug
run-fast: $(addprefix output/, $(addsuffix .run, $(global_asm_tests) $(global_bmarks)))

