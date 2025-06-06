
# This is the Makefile for the hpmicro port of MicroPython
# It is used to build the port and its modules
# The port is built using the hpmicro toolchain and the hpmicro SDK.

include ../../py/verbose.mk
ifdef BOARD_DIR
BOARD ?= $(notdir $(BOARD_DIR:/=))
else
BOARD ?= hpm5301evklite
BOARD_DIR ?= boards/$(BOARD)
endif

ifeq ($(wildcard $(BOARD_DIR)/.),)
$(error Invalid BOARD specified: $(BOARD_DIR))
endif

BUILD ?= build-$(BOARD)

SDK_PATH ?= ../../lib/hpm_sdk

ifeq ($(BUILD_VERBOSE),1)
MAKE_ARGS += VERBOSE=1  # Picked up in Makefile generated by CMake
Q =
else
MAKE_ARGS += VERBOSE=0  # Picked up in Makefile generated by CMake
Q = @
endif

CMAKE_ARGS += -DMICROPY_BOARD=$(BOARD) -DMICROPY_BOARD_DIR="$(abspath $(BOARD_DIR))" -DHPM_SDK_PATH_OVERRIDE="$(abspath $(SDK_PATH))"

ifdef USER_C_MODULES
CMAKE_ARGS += -DUSER_C_MODULES=${USER_C_MODULES}
endif

ifneq ($(FROZEN_MANIFEST),)
CMAKE_ARGS += -DMICROPY_FROZEN_MANIFEST=${FROZEN_MANIFEST}
endif

ifeq ($(DEBUG),1)
CMAKE_ARGS += -DCMAKE_BUILD_TYPE=Debug
endif

# Check if hpm_sdk submodule is initialized

CROSS_COMPILE ?= riscv32-unknown-elf-

HELP_HPMICRO_SDK_SUBMODULE ?= "\033[1;31mError: hpm_sdk submodule is not initialized.\033[0m Run 'make submodules'"
HELP_BUILD_ERROR ?= "See \033[1;31mhttps://github.com/micropython/micropython/wiki/Build-Troubleshooting\033[0m"

.PHONY: all
all:
	$(Q)[ -f $(SDK_PATH)/README.md ] || (echo -e $(HELP_HPMICRO_SDK_SUBMODULE); false)
	$(Q)[ -e $(BUILD)/Makefile ] || cmake -S . -G Ninja .. -B $(BUILD) -DHPMICRO_BUILD_DOCS=0 ${CMAKE_ARGS}
	$(Q)$(MAKE) $(MAKE_ARGS) -C $(BUILD) || (echo -e $(HELP_BUILD_ERROR); false)

#$(BUILD)/Makefile: CMakeLists.txt $(BOARD_DIR)/CMakeLists.txt	
#	@mkdir -p $(BUILD)
#	cd $(BUILD) && cmake $(CMAKE_ARGS) -G Ninja .. || (echo $(HELP_HPMICRO_SDK_SUBMODULE) && exit 1)	
	@echo "Build files generated in $(BUILD)"
.PHONY: clean
clean:
	@rm -rf $(BUILD)
	@echo "Cleaned build directory $(BUILD)"
.PHONY: submodules
submodules:
	@echo "Initializing submodules..."
	$(Q)$(MAKE) -f ../../py/mkrules.mk GIT_SUBMODULES="lib/hpm_sdk" submodules
	$(Q)cmake -S . -B $(BUILD)/submodules -DUPDATE_SUBMODULES=1 ${CMAKE_ARGS}
#	git submodule update --init --recursive || (echo $(HELP_HPMICRO_SDK_SUBMODULE) && exit 1)
	@echo "Submodules initialized."
.PHONY: help
help:
	@echo "Makefile for hpmicro port of MicroPython"
	@echo "Usage:"
	@echo "  make all          - Build the port"
	@echo "  make clean        - Clean the build directory"
	@echo "  make submodules   - Initialize git submodules"
	@echo "  make help         - Show this help message"
	@echo ""
	@echo "Environment variables:"
	@echo "  BOARD_DIR         - Directory containing board-specific files (default: boards/hpmicro)"
	@echo "  BUILD             - Build directory (default: build-hpmicro)"
	@echo "  DEBUG             - Set to 1 to enable debug build"	
	@echo "  FROZEN_MANIFEST   - Path to frozen manifest file"
	@echo "  USER_C_MODULES    - Path to user C modules"
	@echo "  MAKE_ARGS         - Additional arguments to pass to make"
	@echo "  CMAKE_ARGS        - Additional arguments to pass to cmake"
.PHONY: check
check:
	@echo "Checking for required tools..."
	@command -v cmake >/dev/null 2>&1 || (echo "\033[1;31mError: cmake is not installed.\033[0m" && exit 1)
	@command -v ninja >/dev/null 2>&1 || (echo "\033[1;31mError: ninja is not installed.\033[0m" && exit 1)
	@echo "All required tools are available."
