# SPDX-FileCopyrightText: 2026 Baptiste Le Duc <baptiste.le-duc@vates.tech>
# SPDX-License-Identifier: GPL-2.0

# ==============================================================================
# Host-side Docker wrapper for Xen RISC-V / xen-riscv64-trixie
#
# Usage: make <target> [INITRD=initrd|initrd-tools] [IMAGE=<tag>]
#
# Before first use: copy config.mk.example to config.mk and set your paths.
# ==============================================================================

-include config.mk

ifndef XEN_SRC
$(error XEN_SRC is not set. Copy config.mk.example to config.mk and fill in your paths.)
endif

IMAGE ?= baptleduc/xen-riscv64-trixie:latest
$(info )
$(info Image: $(IMAGE))
$(info )

BUILD_DIR := /build
XEN_DIR   := $(BUILD_DIR)/xen

DOCKER := docker run --rm -it \
	--platform linux/amd64 \
	-v "$(XEN_SRC)":$(XEN_DIR)

ifdef KERNEL
ifeq ($(wildcard $(KERNEL)),)
$(error KERNEL file not found: $(KERNEL))
endif
DOCKER += -v "$(KERNEL)":/build/kernel/Image.gz \
          -e "KERNEL=/build/kernel/Image.gz"
endif

# Mount vmlinux for GDB kernel symbol loading
ifdef VMLINUX
ifeq ($(wildcard $(VMLINUX)),)
$(error VMLINUX file not found: $(VMLINUX))
endif
DOCKER += -v "$(VMLINUX)":/build/vmlinux \
          -e "VMLINUX=/build/vmlinux"
endif

INITRD ?= initrd
VALID_INITRD := initrd initrd-tools
ifneq ($(filter $(INITRD),$(VALID_INITRD)),$(INITRD))
$(error INITRD must be one of: $(VALID_INITRD) (got '$(INITRD)'))
endif

.PHONY: help setup build-kernel build-image shell run run-rebuild debug debug-rebuild gdb gdb-rebuild clean

help: ## Show this help message
	@echo "Targets:"
	@grep -hE '^[a-zA-Z_-]+:.*##' $(MAKEFILE_LIST) | \
		awk 'BEGIN {FS = ":.*##"}; {printf "  %-16s %s\n", $$1, $$2}'
	@echo ""
	@echo "Options:"
	@echo "  INITRD=initrd        Minimal initrd (default)"
	@echo "  INITRD=initrd-tools  Initrd with Xen tools included"
	@echo "  IMAGE=<tag>          Docker image (default: $(IMAGE))"
	@echo "  KERNEL=<path>        Pre-built kernel image, overrides the bundled one"
	@echo "  VMLINUX=<path>       Path to vmlinux, enables kernel symbols in GDB"
	@echo ""
	@echo "Local config: config.mk (copy from config.mk.example)"

setup: ## Register QEMU binfmt emulators (run once after each reboot)
	docker run --rm --privileged multiarch/qemu-user-static --reset -p yes

build-kernel: ## Build local Linux kernel (requires KERNEL_SRC=<path>)
ifndef KERNEL_SRC
	$(error KERNEL_SRC is not set. Set it in config.mk.)
endif
	docker run --rm -v "$(KERNEL_SRC)":/build/linux tuxmake/riscv_gcc-14 \
		/bin/bash -c "make -C /build/linux ARCH=riscv CROSS_COMPILE=riscv64-linux-gnu- -j$$(nproc) xen_defconfig Image.gz"

build-image: ## Build the Docker image locally from image/
	docker buildx build --platform linux/amd64 -f image/Dockerfile -t $(IMAGE) image/

build-image-debug: ## Build the Docker image with debug tools (gdb) included
	docker buildx build --platform linux/amd64 --build-arg DEBUG_XL=1 -f image/Dockerfile -t $(IMAGE) image/

shell: ## Open an interactive shell in the container
	$(DOCKER) -p 1234:1234 $(IMAGE) /bin/bash

run: ## Boot Xen + dom0 in QEMU
	$(DOCKER) -p 1234:1234 $(IMAGE) \
		/bin/bash -c "make $(INITRD) && make go"

debug: ## Boot Xen in QEMU, wait for GDB on port 1234
	$(DOCKER) -p 1234:1234 $(IMAGE) \
		/bin/bash -c "make $(INITRD) && make go-debug"

# VMLINUX is optional: without it, GDB loads Xen symbols only.
gdb: ## Attach GDB (TUI) to a running Xen instance
ifndef VMLINUX
	$(warning VMLINUX is not set: Linux kernel symbols will not be loaded. Set VMLINUX in config.mk to enable them.)
endif
	$(DOCKER) --network host $(IMAGE) \
		/bin/bash -c "make gdb GDB_FLAGS=-tui"

gdb-rebuild: ## Rebuild Xen, then attach GDB
ifndef VMLINUX
	$(warning VMLINUX is not set: Linux kernel symbols will not be loaded. Set VMLINUX in config.mk to enable them.)
endif
	$(DOCKER) --network host $(IMAGE) \
		/bin/bash -c "make build-xen && make gdb GDB_FLAGS=-tui"

clean: ## Clean all Xen build artifacts (distclean)
	$(DOCKER) $(IMAGE) /bin/bash -c "make clean"
