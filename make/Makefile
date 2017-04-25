###############################################################################
## Makefile for compiling apps against the STM32 CMSIS and ST HAL
################################################################################

################################################################################
## Usage
##
## In your application Makefile, set the following variables:
##   - DEVICE             : The STM32 device used (for example stm32l073xx)
##   - DEVICE_FAMILY      : The STM32 family of your device (for example stm32l0xx)
##   - DEVICE_FULL_NAME   : The full name of your STM32 device (for example STM32L073RZ)
##                          This information is used to get the default linker script file
##                          (see ld directory)
##
## Optional for Template:
##   - SRC_PATH           : Local Directory used to copy C template files (default is "../src")
##   - INC_PATH           : Local Directory used to copy H template files (default is as SRC_PATH)
## Note : these two directories are also autmatically added to SOURCE_PATHS and LIBRARY_PATHS.
##   
##
## Optional:
##   - SOURCE_PATHS       : List of directories with .c files
##   - LIBRARY_PATHS      : List of directories with .h files
##   - START_CODE         : .s file to execute first
##   - SYSTEM_FILE        : Base STM32 .c file (copied inside te base template)  
##   - BOARD              : Hardware board being used. Will get set as #define.
##   - USE_ST_HAL         : must be define to "true" tu use ST HAL.
##   - USE_ST_BOARD       : can be define to "true" to include ST Board Definition
##                          usefull to compile example from st as it.
##
## Then at the end of the Makefile do something like:
##    STM_BASE_PATH ?= ../../../stm32_cube_l0
##    include $(STM_BASE_PATH)/make/Makefile


DEVICE ?= stm32l073xx
DEVICE_FAMILY ?= stm32l0xx
DEVICE_FULL_NAME ?= STM32L073RZT

# for example stm32l073xx => STM32L073xx
DEVICE_UP ?= $(shell echo $(subst xx,,$(DEVICE)) | tr a-z A-Z)xx
DEVICE_FAMILY_UP ?= $(shell echo $(subst xx,,$(DEVICE_FAMILY)) | tr a-z A-Z)xx
DEVICE_FULL_NAME_UP ?= $(shell echo $(subst xx,,$(DEVICE_FULL_NAME)) | tr a-z A-Z)

# Define Local User directory
SRC_PATH ?= ../src
INC_PATH ?= $(SRC_PATH)

# Helper for relative paths
BASE_DIR := $(dir $(lastword $(MAKEFILE_LIST)))..

# Setup the compilers to use
TOOLCHAIN_PREFIX ?= arm-none-eabi
AS      = $(TOOLCHAIN_PATH)$(TOOLCHAIN_PREFIX)-as
CC      = $(TOOLCHAIN_PATH)$(TOOLCHAIN_PREFIX)-gcc
LD      = $(TOOLCHAIN_PATH)$(TOOLCHAIN_PREFIX)-gcc
OBJCOPY = $(TOOLCHAIN_PATH)$(TOOLCHAIN_PREFIX)-objcopy
OBJDUMP = $(TOOLCHAIN_PATH)$(TOOLCHAIN_PREFIX)-objdump
SIZE    = $(TOOLCHAIN_PATH)$(TOOLCHAIN_PREFIX)-size
GDB     = $(TOOLCHAIN_PATH)$(TOOLCHAIN_PREFIX)-gdb
MAKE_BUILD_FOLDER = mkdir -p $(OUTPUT_PATH)

# Set default board
BOARD ?= BOARD_NULL

USE_ST_HAL ?= false
USE_ST_BOARD ?= false

# Print some helpful info about what the user is compiling for
space :=
space +=
$(info BUILD OPTIONS:)
$(info $(space) Device      $(DEVICE))
$(info $(space) Board       $(BOARD))
$(info $(space) ST HAL      $(USE_ST_HAL))
$(info $(space) ST BOARD    $(USE_ST_BOARD))
$(info $(space))

# Required files to compile
START_CODE ?= startup_$(DEVICE).s
SYSTEM_FILE ?= system_$(DEVICE_FAMILY).c

# If not set in app makefile, lets optimize for size
all: CFLAGS += -Os #Optimize for size (only in release)
CFLAGS += -fomit-frame-pointer #Don't keep the frame pointer in a register for functions that don't need one

#Define Path Shortcut
CMSIS_PATH ?= $(BASE_DIR)/Drivers/CMSIS

# Add user default directory
LIBRARY_PATHS += $(INC_PATH)
SOURCE_PATHS += $(SRC_PATH)

# Add useful paths from stm32_cube_l0
LIBRARY_PATHS += $(CMSIS_PATH)/Include
LIBRARY_PATHS += $(CMSIS_PATH)//Device/ST/$(DEVICE_FAMILY_UP)/Include

# Add paths for ST CUBE HAL if used
ifeq ($(USE_ST_HAL), true)
	HAL_PATH ?= $(BASE_DIR)/Drivers/$(DEVICE_FAMILY_UP)_HAL_Driver
	
	LIBRARY_PATHS += $(HAL_PATH)/Inc
	
	SOURCE_PATHS += $(HAL_PATH)/Src
	
	# Basic hal source, other must be added by user from its makefile
	APPLICATION_SRCS += $(DEVICE_FAMILY)_hal.c 
	APPLICATION_SRCS += $(DEVICE_FAMILY)_hal_rcc.c
	APPLICATION_SRCS += $(DEVICE_FAMILY)_hal_rcc_ex.c
	APPLICATION_SRCS += $(DEVICE_FAMILY)_hal_cortex.c
	APPLICATION_SRCS += $(DEVICE_FAMILY)_hal_gpio.c
	APPLICATION_SRCS += $(DEVICE_FAMILY)_hal_pwr_ex.c
	
	CFLAGS += -DUSE_HAL_DRIVER
	
endif

# Add all paths for ST Board Definition if used
ifeq ($(USE_ST_BOARD), true)
	BSP_PATH ?= $(BASE_DIR)/Drivers/BSP
	
	LIBRARY_PATHS += $(BSP_PATH)/STM32L0xx_Nucleo
	LIBRARY_PATHS += $(BSP_PATH)/STM32L0xx_Nucleo_32
	LIBRARY_PATHS += $(BSP_PATH)/STM32L073Z_EVAL
	LIBRARY_PATHS += $(BSP_PATH)/STM32L0538-Discovery

endif


# Define Compilation parameters

LINKER_SCRIPT ?= $(BASE_DIR)/make/ld/$(DEVICE_FULL_NAME_UP)x_FLASH.ld
OUTPUT_NAME ?= $(PROJECT_NAME)

LIBRARY_INCLUDES = $(addprefix -I,$(LIBRARY_PATHS))
CMSIS_INCLUDE = $(addprefix -I,$(CMSIS_INCLUDE_PATH))

VPATH = $(SOURCE_PATHS)

OUTPUT_PATH ?= _build/

CPUFLAGS += -mcpu=cortex-m0plus -mthumb -mfloat-abi=soft
ASFLAGS += $(CPUFLAGS) -g
CFLAGS += -std=gnu99 -c $(CPUFLAGS) -Wall
CFLAGS += -D$(DEVICE_UP) -D$(BOARD)
CFLAGS += -s -fdata-sections -ffunction-sections
CFLAGS += $(CMSIS_INCLUDE) $(LIBRARY_INCLUDES) -MD
LDFLAGS += $(CPUFLAGS) -specs=nosys.specs -specs=nano.specs -L $(BASE_DIR)/make/ld -T $(LINKER_SCRIPT) -Wl,--gc-sections -Wl,-Map=$(OUTPUT_PATH)$(OUTPUT_NAME).Map -lm
OBJDUMP_FLAGS += --disassemble-all --source --disassembler-options=force-thumb -C --section-headers

HEX = $(OUTPUT_PATH)$(OUTPUT_NAME).hex
ELF = $(OUTPUT_PATH)$(OUTPUT_NAME).elf
BIN = $(OUTPUT_PATH)$(OUTPUT_NAME).bin
LST = $(OUTPUT_PATH)$(OUTPUT_NAME).lst

SRCS = $(SYSTEM_FILE) $(notdir $(APPLICATION_SRCS))
OBJS = $(addprefix $(OUTPUT_PATH), $(SRCS:.c=.o)) $(addprefix $(OUTPUT_PATH),$(APPLICATION_LIBS))
DEPS = $(addprefix $(OUTPUT_PATH), $(SRCS:.c=.d))
SRCS_AS = $(START_CODE)
OBJS_AS = $(addprefix $(OUTPUT_PATH), $(SRCS_AS:.s=.os))


### Verbosity control. Use  make V=1  to get verbose builds.

ifeq ($(V),1)
  PRINT_CC  =
  PRINT_LD  =
  PRINT_AR  =
  PRINT_AS  =
  PRINT_HEX =
  PRINT_BIN =
  PRINT_SIZ =
  Q=
else
  PRINT_CC  = @echo "  CC       " $<
  PRINT_LD  = @echo "  LD       " $(ELF)
  PRINT_AR  = @echo "  AR       " $@
  PRINT_AS  = @echo "  AS       " $<
  PRINT_HEX = @echo "  HEX      " $(HEX)
  PRINT_BIN = @echo "  BIN      " $(BIN)
  PRINT_LST = @echo "  LST      " $(LST)
  PRINT_SIZ = @echo "  SIZE     " $(ELF)
  Q=@
endif

all: $(OBJS) $(OBJS_AS) $(HEX)

debug: CFLAGS += -g
debug: $(OBJS) $(OBJS_AS) $(HEX)

rebuild: clean all

$(HEX): $(OBJS) $(OBJS_AS)
	$(PRINT_LD)
	$(Q)$(LD) $(LDFLAGS) $(OBJS_AS) $(OBJS) -o $(ELF)
	$(PRINT_HEX)
	$(Q)$(OBJCOPY) -Oihex $(ELF) $(HEX)
	$(PRINT_BIN)
	$(Q)$(OBJCOPY) -Obinary $(ELF) $(BIN)
	$(PRINT_LST)
	$(Q)$(OBJDUMP) $(OBJDUMP_FLAGS) $(ELF) > $(LST)
	$(PRINT_SIZ)
	$(Q)$(SIZE) $(ELF)

size: $(ELF)
	$(SIZE) $(ELF)

$(OUTPUT_PATH)%.o: %.c
	@$(MAKE_BUILD_FOLDER)
	$(PRINT_CC)
	$(Q)$(CC) $(CFLAGS) $< -o $@

$(OUTPUT_PATH)%.os: %.s
	@$(MAKE_BUILD_FOLDER)
	$(PRINT_AS)
	$(Q)$(AS) $(ASFLAGS) $< -o $@

-include $(DEPS)

clean:
	rm -rf $(OUTPUT_PATH)

.PHONY: all clean rebuild size

#---- TEMPLATE

dep obj $(SRC_PATH):
	@echo "[MKDIR]   $@"
	$Qmkdir -p $@

# Minimal Project is composed by
#   - create: main.c
#   - create from template: system_<device_family>.c
#   - copy startup code: startup_<device>.s
#   - use linker_script: <DEVICE>CE_FLASH.ld
template_minimal: $(SRC_PATH)
	cp $(BASE_DIR)/Template/main.c $(SRC_PATH)
	cp $(CMSIS_PATH)/Device/ST/$(DEVICE_FAMILY_UP)/Source/Templates/gcc/startup_$(DEVICE).s $(SRC_PATH)
	cp $(CMSIS_PATH)/Device/ST/$(DEVICE_FAMILY_UP)/Source/Templates/system_$(DEVICE_FAMILY).c $(SRC_PATH)

# Helper File : <device_family>_it.c & <device_family>_it.h
template_minimal_irq: template_minimal
	cp $(BASE_DIR)/Template/$(DEVICE_FAMILY)_it.c $(SRC_PATH)
	cp $(BASE_DIR)/Template/$(DEVICE_FAMILY)_it.h $(SRC_PATH)

# StCube configuration: <device_family>_hal_conf.h
template_cube: template_minimal_irq
	cp $(BASE_DIR)/Drivers/STM32L0xx_HAL_Driver/Inc/stm32l0xx_hal_conf_template.h $(SRC_PATH)/stm32l0xx_hal_conf.h	
	
