cmake_minimum_required(VERSION 3.22)

#
# This file is generated only once,
# and is not re-generated if converter is called multiple times.
#
# User is free to modify the file as much as necessary
#

# Setup compiler settings
set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS ON)


# Define the build type
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE "Debug")
endif()

# Set the project name
set(CMAKE_PROJECT_NAME test)

# Include toolchain file
include("cmake/gcc-arm-none-eabi.cmake")

# Core project settings
# 修正：将 enable_language 的功能合并到 project 命令中，并确保 project 在语言启用之前被调用。
project(${CMAKE_PROJECT_NAME} LANGUAGES C ASM)
message("Build type: " ${CMAKE_BUILD_TYPE})

# Enable compile command to ease indexing with e.g. clangd
set(CMAKE_EXPORT_COMPILE_COMMANDS TRUE)

# 移除：enable_language(C ASM) 因为已合并到 project 命令中

# Create an executable object type
add_executable(${CMAKE_PROJECT_NAME})

# Add STM32CubeMX generated sources
add_subdirectory(cmake/stm32cubemx)

# Link directories setup
target_link_directories(${CMAKE_PROJECT_NAME} PRIVATE
    # Add user defined library search paths
)

# Add sources to executable
target_sources(${CMAKE_PROJECT_NAME} PRIVATE
    # Add user sources here
)

# 设置可执行文件输出名称
set(EXECUTABLE_NAME ${CMAKE_PROJECT_NAME})
# 移除多余的 CMAKE_EXECUTABLE_NAME 设置，因为 set_target_properties 会处理
# set(CMAKE_EXECUTABLE_NAME ${EXECUTABLE_NAME})
# 设置全局的可执行文件后缀，对当前目标来说，set_target_properties 已经足够
set(CMAKE_EXECUTABLE_SUFFIX .elf)
set(HEX_FILE ${EXECUTABLE_NAME}.hex)
set(BIN_FILE ${EXECUTABLE_NAME}.bin)

# 设置链接器脚本
if(NOT DEFINED LINKER_SCRIPT)
    set(LINKER_SCRIPT "${CMAKE_SOURCE_DIR}/stm32f103c8tx_flash.ld" CACHE FILEPATH "Linker script")
    message(STATUS "Using linker script: ${LINKER_SCRIPT}")
endif()

# 添加自定义命令来生成HEX和BIN文件
add_custom_command(TARGET ${CMAKE_PROJECT_NAME} POST_BUILD
    COMMAND ${CMAKE_OBJCOPY} -O ihex $<TARGET_FILE:${CMAKE_PROJECT_NAME}> ${HEX_FILE}
    COMMAND ${CMAKE_OBJCOPY} -O binary -S $<TARGET_FILE:${CMAKE_PROJECT_NAME}> ${BIN_FILE}
    COMMENT "Generating ${HEX_FILE} and ${BIN_FILE}"
    VERBATIM
)

# 设置目标属性
set_target_properties(${CMAKE_PROJECT_NAME} PROPERTIES
    OUTPUT_NAME ${EXECUTABLE_NAME}
    SUFFIX ".elf"
)

# Add include paths
# 可以考虑将所有 include 路径合并到一个 target_include_directories 调用中
target_include_directories(${CMAKE_PROJECT_NAME} PRIVATE
    # Add user defined include paths
)

# Add project symbols (macros)
target_compile_definitions(${CMAKE_PROJECT_NAME} PRIVATE
    # Add user defined symbols
)

# Add linked libraries
# 修正：为 target_link_libraries 添加 PRIVATE 作用域关键字
target_link_libraries(${CMAKE_PROJECT_NAME} PRIVATE
    stm32cubemx

    # Add user defined libraries
)
