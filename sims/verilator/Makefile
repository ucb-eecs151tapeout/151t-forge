#
# RISCV-SODOR Verilator Makefile
#

# --- Boilerplate ---
SHELL := /bin/bash
.SHELLFLAGS += -o pipefail -c
.SUFFIXES:
.PHONY: default all verilog fesvr sim run clean
.DEFAULT_GOAL := all

# --- Path Setup ---
sim_dir  := $(patsubst %/,%,$(dir $(abspath $(lastword $(MAKEFILE_LIST)))))
base_dir := $(abspath $(sim_dir)/../..)

sodor_dir         := $(base_dir)/generators/riscv-sodor
vsrc_dir          := $(sodor_dir)/vsrc
riscv_isa_sim_dir := $(base_dir)/tools/riscv-isa-sim
common_sim_dir    := $(sodor_dir)/emulator/common

build_root_dir    := $(sim_dir)/build
output_root_dir   := $(sim_dir)/output

# --- User-configurable Variables ---
MK_TARGET_PROC ?= rv32_1stage
BINARY         ?=
MAX_CYCLES     ?= 1000000
VERILATOR_THREADS ?= 1
RANDOM_SEED    ?=
MODEL          ?= Top

# --- Toolchain ---
SBT       ?= sbt
CXX       ?= g++
VERILATOR ?= verilator
LDFLAGS   ?=

# --- Derived Variables ---
core_build_dir    := $(build_root_dir)/$(MK_TARGET_PROC)
output_dir        := $(output_root_dir)/$(MK_TARGET_PROC)
sim_exe           := $(sim_dir)/simulator-$(MK_TARGET_PROC)

TOP_V             := $(core_build_dir)/generated-src/Top.v
verilog_stamp     := $(core_build_dir)/generated-src/.stamp

fesvr_build_dir   := $(core_build_dir)/fesvr
libfesvr          := $(fesvr_build_dir)/lib/libfesvr.a

verilator_obj_dir := $(core_build_dir)/verilator_obj_dir

# --- Build Targets ---
all: $(sim_exe)
sim: $(sim_exe)

# 1. Chisel: Elaborate Verilog
verilog: $(TOP_V)
$(TOP_V): $(verilog_stamp)
	@# Phony dependency to ensure Verilog is generated.
$(verilog_stamp): $(wildcard $(sodor_dir)/src/common/*.scala) $(wildcard $(sodor_dir)/src/$(MK_TARGET_PROC)/*.scala) $(sodor_dir)/build.sbt
	@echo "Elaborating $(MK_TARGET_PROC) Verilog..."
	mkdir -p $(dir $@)
	cd $(sodor_dir) && $(SBT) "project $(MK_TARGET_PROC)" "run -td $(abspath $(dir $@))"
	touch $@

# 2. FESVR library
fesvr: $(libfesvr)
$(fesvr_build_dir)/Makefile: $(riscv_isa_sim_dir)/configure
	mkdir -p $(fesvr_build_dir)
	cd $(fesvr_build_dir) && $(riscv_isa_sim_dir)/configure --prefix=$(fesvr_build_dir)
$(libfesvr): $(fesvr_build_dir)/Makefile
	@echo "Building and installing FESVR..."
	$(MAKE) -C $(fesvr_build_dir)
	$(MAKE) -C $(fesvr_build_dir) install

# 3. Verilator Build
VERILATOR_INCLUDE := $(shell pkg-config --variable=includedir verilator 2>/dev/null)
ifeq ($(VERILATOR_INCLUDE),)
ifeq ($(VERILATOR_ROOT),)
$(error VERILATOR_ROOT is not set and pkg-config could not find verilator)
endif
VERILATOR_INCLUDE := $(VERILATOR_ROOT)/include
endif

VLT_CFLAGS := -I$(common_sim_dir) -I$(riscv_isa_sim_dir) -I$(fesvr_build_dir)/include -I$(VERILATOR_INCLUDE) -DVERILATOR -include $(common_sim_dir)/verilator.h
CXXFLAGS += -O1 -std=c++17 -g

VLT_LDFLAGS := "$(LDFLAGS) -L$(fesvr_build_dir)/lib -lfesvr -lpthread -ldl -lfdt"
VLT_OPTS = \
	--cc --exe --main \
	-O3 --x-assign 0 --x-initial 0 \
	-Wall -Wno-fatal --assert \
	--threads $(VERILATOR_THREADS) --threads-dpi all \
	+define+PRINTF_COND=1 +define+VERILATOR \
	--top-module $(MODEL) \
	-I$(vsrc_dir) \
	-CFLAGS '$(VLT_CFLAGS)' \
	-LDFLAGS '$(VLT_LDFLAGS)' \
	-o $(sim_exe)

VLT_VSRCS = $(TOP_V) $(vsrc_dir)/SimDTM.v $(vsrc_dir)/AsyncReadMem.sv $(vsrc_dir)/SyncMem.sv
VLT_CPPSRCS = $(common_sim_dir)/emulator.cpp $(common_sim_dir)/SimDTM.cc

verilator_model_mk := $(verilator_obj_dir)/V$(MODEL).mk
$(verilator_model_mk): $(TOP_V) $(libfesvr)
	@echo "Verilating $(MK_TARGET_PROC)..."
	$(VERILATOR) $(VLT_OPTS) $(VLT_VSRCS) $(VLT_CPPSRCS) --Mdir $(verilator_obj_dir)

$(sim_exe): $(verilator_model_mk)
	@echo "Building simulator executable..."
	$(MAKE) -C $(verilator_obj_dir) -f V$(MODEL).mk

# --- Run Targets ---
run: $(sim_exe)
	@if [ -z "$(BINARY)" ]; then 
		echo "ERROR: Must specify a binary to run with BINARY="; exit 1; 
	fi
	mkdir -p $(output_dir)
	@echo "Running $(BINARY) on $(MK_TARGET_PROC) simulator..."
	$(sim_exe) +max-cycles=$(MAX_CYCLES) $(BINARY) 2>&1 | tee $(output_dir)/$(notdir $(BINARY)).log

# --- Clean Targets ---
clean:
	rm -rf $(build_root_dir) $(output_root_dir) $(sim_dir)/simulator-*
