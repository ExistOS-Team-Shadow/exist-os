cmake_minimum_required(VERSION 3.16)

project(ExistOS-OSLoader VERSION 0.1.0.1)


add_link_options(-T ${CMAKE_SOURCE_DIR}/Script/ldr_ld.script -Wl,--gc-sections)

SET(CMAKE_EXE_LINKER_FLAGS "-static")
set(CMAKE_BUILD_TYPE "MinSizeRel") 

include_directories(./Config)
include_directories(./Fonts)
include_directories(./Scheduler/include)
include_directories(./Include)
include_directories(./HAL/Hardware)
include_directories(./HAL)
include_directories(./HAL/Hardware/registers)
include_directories(./VmMgr)
include_directories(./LowLevelAPI)
include_directories(./Scheduler/porting)
include_directories(./Component3rd/tinyusb/src)
include_directories(./Component3rd/tinyusb/src/device)
include_directories(./Component3rd/tinyusb/src/class/hid)
include_directories(./Component3rd/tinyusb/src/class/net)
include_directories(./Component3rd/tinyusb/src/class/vendor)
include_directories(./Component3rd/dhara/dhara)
#include_directories(./Component3rd/FatFs)

aux_source_directory(.                                  LOADER_SRCS)
#aux_source_directory(./Scheduler                        LOADER_SRCS)
aux_source_directory(./VmMgr                            LOADER_SRCS)
aux_source_directory(./LowLevelAPI                      LOADER_SRCS)
#aux_source_directory(./Scheduler/porting                LOADER_SRCS)
#aux_source_directory(./HAL                              LOADER_SRCS)
#aux_source_directory(./HAL/Hardware                     LOADER_SRCS)

#aux_source_directory(./Component3rd                     LOADER_SRCS)
#aux_source_directory(./Component3rd/tinyusb/src             LOADER_SRCS)
#aux_source_directory(./Component3rd/tinyusb/src/device      LOADER_SRCS)
#aux_source_directory(./Component3rd/tinyusb/src/common      LOADER_SRCS)
#aux_source_directory(./Component3rd/tinyusb/src/class/cdc   LOADER_SRCS)
#aux_source_directory(./Component3rd/tinyusb/src/class/msc   LOADER_SRCS)
#aux_source_directory(./Component3rd/tinyusb/src/class/hid   LOADER_SRCS)
#aux_source_directory(./Component3rd/tinyusb/src/class/net   LOADER_SRCS)
#aux_source_directory(./Component3rd/tinyusb/src/class/vendor   LOADER_SRCS)
#aux_source_directory(./Component3rd/dhara               LOADER_SRCS)
#aux_source_directory(./Component3rd/FatFs               LOADER_SRCS)

add_subdirectory(Component3rd)
add_subdirectory(HAL)
add_subdirectory(Scheduler)

add_executable(rom.elf ${LOADER_SRCS})

target_compile_options(kernel PRIVATE -mtune=arm926ej-s -mcpu=arm926ej-s -mlittle-endian -mfloat-abi=soft -marm -Os)
target_compile_options(hal PRIVATE -mtune=arm926ej-s -mcpu=arm926ej-s -mlittle-endian -mfloat-abi=soft -marm  -Os )
target_compile_options(hal_hardware PRIVATE -mtune=arm926ej-s -mcpu=arm926ej-s -mlittle-endian -mfloat-abi=soft -mthumb -Os)
target_compile_options(tusb PRIVATE -mtune=arm926ej-s -mcpu=arm926ej-s -mlittle-endian -mfloat-abi=soft -mthumb -Os)
target_compile_options(dhara PRIVATE -mtune=arm926ej-s -mcpu=arm926ej-s -mlittle-endian -mfloat-abi=soft -marm -O2)

target_link_libraries(
rom.elf
kernel
tusb 
hal hal_hardware
dhara
-lc_nano -nostartfiles)

set_property(TARGET kernel PROPERTY INTERPROCEDURAL_OPTIMIZATION TRUE)
#set_property(TARGET tusb PROPERTY INTERPROCEDURAL_OPTIMIZATION TRUE)
#set_property(TARGET hal PROPERTY INTERPROCEDURAL_OPTIMIZATION TRUE)
#set_property(TARGET hal_hardware PROPERTY INTERPROCEDURAL_OPTIMIZATION TRUE)
#set_property(TARGET dhara PROPERTY INTERPROCEDURAL_OPTIMIZATION TRUE)

target_compile_options(rom.elf PRIVATE -mtune=arm926ej-s -mcpu=arm926ej-s -mlittle-endian -marm -Wall
#-O2
-Os -DNDEBUG
-pipe 
-fdata-sections 
-ffunction-sections
-fdelete-null-pointer-checks 
-fcommon 
-finline-functions 
-finline-small-functions

)
 

add_custom_command(TARGET rom.elf POST_BUILD
    COMMAND arm-none-eabi-size ${CMAKE_CURRENT_BINARY_DIR}/rom.elf 
    )
	
add_custom_target(
  rom.bin ALL
  COMMAND ${CMAKE_OBJCOPY} -I elf32-little -O binary -R .note -R .comment -S ${CMAKE_CURRENT_BINARY_DIR}/rom.elf ${CMAKE_CURRENT_BINARY_DIR}/rom.bin 
  DEPENDS rom.elf 
  )

if (${CMAKE_HOST_SYSTEM_NAME} STREQUAL "Windows")
add_custom_target(
  OSLoader.sb ALL
  COMMAND ${CMAKE_SOURCE_DIR}/tools/sbtools_win/elftosb -z -c ${CMAKE_SOURCE_DIR}/Script/build_fw.bd -o ${CMAKE_CURRENT_BINARY_DIR}/OSLoader.sb ${CMAKE_CURRENT_BINARY_DIR}/rom.bin
  DEPENDS rom.bin
  )
add_custom_target(
  sb_flash 
  COMMAND ${CMAKE_SOURCE_DIR}/tools/sbtools_win/sb_loader -f ${CMAKE_CURRENT_BINARY_DIR}/OSLoader.sb 
  DEPENDS OSLoader.sb
  )
else()
if(EXISTS ${CMAKE_SOURCE_DIR}/tools/sbtools/  )
add_custom_target(
  OSLoader.sb ALL
  COMMAND ${CMAKE_SOURCE_DIR}/tools/sbtools/elftosb -z -c ${CMAKE_SOURCE_DIR}/Script/build_fw.bd -o ${CMAKE_CURRENT_BINARY_DIR}/OSLoader.sb ${CMAKE_CURRENT_BINARY_DIR}/rom.bin
  DEPENDS rom.bin
  )
add_custom_target(
  flash 
  COMMAND ${CMAKE_SOURCE_DIR}/tools/sbtools/sbloader ${CMAKE_CURRENT_BINARY_DIR}/OSLoader.sb -d
  DEPENDS OSLoader.sb
  )
else(EXISTS ${CMAKE_SOURCE_DIR}/tools/sbtools/elftosb)
message(FATAL_ERROR "Please build tools/sbtools first")
endif(EXISTS ${CMAKE_SOURCE_DIR}/tools/sbtools/elftosb)
endif()

add_custom_target(
    edb_flash_loader 
    COMMAND ${CMAKE_SOURCE_DIR}/tools/edb -r -s -f ${CMAKE_CURRENT_BINARY_DIR}/OSLoader.sb 1408 b
    DEPENDS OSLoader.sb
    )

add_custom_target(
      mscmode 
      COMMAND ${CMAKE_SOURCE_DIR}/tools/edb -m
      )

add_custom_target(
    reboot 
    COMMAND ${CMAKE_SOURCE_DIR}/tools/edb -r
    )