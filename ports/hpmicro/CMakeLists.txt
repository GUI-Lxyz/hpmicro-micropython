cmake_minimum_required(VERSION 3.13)

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE MinSizeRel)
endif()

# Set the project name and version
set(MICROPY_TARGET firmware)
set(CMAKE_C_STANDARD 17) 
set(CMAKE_CXX_STANDARD 17)

get_filename_component(MICROPY_DIR "../.." ABSOLUTE)

if(HPM_SDK_PATH_OVERRIDE)
    set(HPM_SDK_PATH "${HPM_SDK_PATH_OVERRIDE}")
else()
    set(HPM_SDK_PATH "${MICROPY_DIR}/lib/hpm_sdk")
endif()

set(HPM_TINYUSB_PATH ${HPM_SDK_PATH}/middleware/tinyusb)
set(HPM_LWIP_PATH ${HPM_SDK_PATH}/middleware/lwip)

# Set the location of this port's directory.
set(MICROPY_PORT_DIR ${CMAKE_CURRENT_LIST_DIR})

if(NOT MICROPY_BOARD)
    set(MICROPY_BOARD hpm5301evklite)
endif()
# Set the location of the board directory.
if(NOT MICROPY_BOARD_DIR)
    set(MICROPY_BOARD_DIR ${MICROPY_PORT_DIR}/boards/${MICROPY_BOARD})
endif()

get_filename_component(MICROPY_BOARD_DIR ${MICROPY_BOARD_DIR} ABSOLUTE)

if(NOT EXISTS ${MICROPY_BOARD_DIR}/mpconfigboard.cmake)
    message(FATAL_ERROR "Invalid MICROPY_BOARD specified: ${MICROPY_BOARD}")
endif()

set(MICROPY_USER_FROZEN_MANIFEST ${MICROPY_FROZEN_MANIFEST})

include(${MICROPY_BOARD_DIR}/mpconfigboard.cmake)
include(${MICROPY_BOARD_DIR}/mpconfigvariant.cmake OPTIONAL)

# Include the common port configuration.
if(NOT HPM_BOARD)
    string(TOLOWER ${MICROPY_BOARD} HPM_BOARD)
endif()

if(NOT MICROPY_C_HEAP_SIZE)
    set(MICROPY_C_HEAP_SIZE 0)
endif()

# Enable extmod components that will be configured by extmod.cmake.
# A board may also have enabled additional components.
set(MICROPY_SSL_MBEDTLS ON)

list(APPEND GIT_SUBMODULES lib/tinyusb)

include(${MICROPY_DIR}/py/py.cmake)
include(${MICROPY_DIR}/extmod/extmod.cmake)
include(${HPM_SDK_PATH}/CMakeLists.txt)

project(${MICROPY_TARGET})

include(${MICROPY_DIR}/py/usermod.cmake)

add_executable(${MICROPY_TARGET})

set(MICROPY_QSTRDEFS_PORT
    ${MICROPY_PORT_DIR}/qstrdefsport.h
)
set(MICROPY_SOURCE_LIB
    ${MICROPY_DIR}/lib/littlefs/lfs1.c
    ${MICROPY_DIR}/lib/littlefs/lfs1_util.c
    ${MICROPY_DIR}/lib/littlefs/lfs2.c
    ${MICROPY_DIR}/lib/littlefs/lfs2_util.c
    ${MICROPY_DIR}/lib/oofatfs/ff.c
    ${MICROPY_DIR}/lib/oofatfs/ffunicode.c
    ${MICROPY_DIR}/shared/netutils/dhcpserver.c
    ${MICROPY_DIR}/shared/netutils/netutils.c
    ${MICROPY_DIR}/shared/netutils/trace.c
    ${MICROPY_DIR}/shared/readline/readline.c
    ${MICROPY_DIR}/shared/runtime/gchelper_native.c
    ${MICROPY_DIR}/shared/runtime/interrupt_char.c
    ${MICROPY_DIR}/shared/runtime/mpirq.c
    ${MICROPY_DIR}/shared/runtime/pyexec.c
    ${MICROPY_DIR}/shared/runtime/stdout_helpers.c
    ${MICROPY_DIR}/shared/runtime/softtimer.c
    ${MICROPY_DIR}/shared/runtime/sys_stdio_mphal.c
    ${MICROPY_DIR}/shared/timeutils/timeutils.c
    ${MICROPY_DIR}/shared/tinyusb/mp_usbd.c
    ${MICROPY_DIR}/shared/tinyusb/mp_usbd_cdc.c
    ${MICROPY_DIR}/shared/tinyusb/mp_usbd_descriptor.c
    ${MICROPY_DIR}/shared/tinyusb/mp_usbd_runtime.c

    ${MICROPY_DIR}/shared/runtime/gchelper_rv32i.s

)

set(MICROPY_SOURCE_DRIVERS
    ${MICROPY_DIR}/drivers/bus/softspi.c
    ${MICROPY_DIR}/drivers/dht/dht.c
)
