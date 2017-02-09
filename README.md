STM32 CUBE FILE
==========================

This repository is a starting point and shared code for STM32 Cube Files. 
Currently only L0 familly is supported tested, but all is in place to can 
extand this point.
This repo is small copy of the ST Cube Delivery (only usefull file are keept) to
which a Generic Makefile is added to can build application from console or
any other IDE.

The files are currently based on :
 - STM32CubeL0 V1.7.0 delivery


Usage
-----

First, add this project as a submodule inside of your repo with your
STM32 code.

    git submodule add https://github.com/tech4race/stm32-base

Then write an application for the STM32 uc you are using and include
a Makefile that looks like this:

```make
PROJECT_NAME = $(shell basename "$(realpath ./)")

# Define your CPU
DEVICE = stm32l073xx
DEVICE_FAMILY = stm32l0xx
DEVICE_FULL_NAME = STM32L073RZ

APPLICATION_SRCS = $(notdir $(wildcard ../src/*.c))
# Various C libraries that need to be included
#APPLICATION_SRCS += ...

# Add other libraries here!

# If you are using ST HAL
#USE_ST_HAL = true
#APPLICATION_SRCS += stm32l0xx_hal_uart.c

# platform-level headers and source files
LIBRARY_PATHS += ../../include
SOURCE_PATHS += ../../src

# If you are using ST BOARD
USE_ST_BOARD = true


# Include the main Makefile
STM_BASE_PATH ?= ../../../stm32_cube_l0
include $(STM_BASE_PATH)/make/Makefile
```
An example Makefile is included in this repo as Makefile.example. Copy to your
own application directory and modify as desired.

Generally, the expected directory structure for your project is:
```
    /apps
        /<application 1>
          /<make>
             Your Makefile
          /<src>
             Your .c files
        /<application 2>
        ...
    /src
        various platform-level code (e.g. functions shared between applications)
    /include
        various platform-level headers (e.g. platform pin mappings)
    /stm32-base (submodule)
```



Program your Board
------------------

TODO : add this feature


Git Submodules
--------------

If you're using submodules in your project, you may want to use this to make
git automatically update them:
https://gist.github.com/brghena/fc4483a2df83c47660a5


Thanks to
-------

This project organisation is mainly based on the workd done on [nrf5x-base project]
(https://github.com/lab11/nrf5x-base)

