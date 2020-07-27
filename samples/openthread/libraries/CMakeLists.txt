#
# Copyright (c) 2020 Nordic Semiconductor ASA
#
# SPDX-License-Identifier: LicenseRef-BSD-5-Clause-Nordic
#
cmake_minimum_required(VERSION 3.13.1)

include($ENV{ZEPHYR_BASE}/cmake/app/boilerplate.cmake NO_POLICY_SCOPE)
project(NONE)

FILE(GLOB app_sources src/*.c)

target_sources(app PRIVATE ${app_sources})

execute_process(
  COMMAND           git rev-parse HEAD
  WORKING_DIRECTORY ${openthread_SOURCE_DIR}
  RESULT_VARIABLE   git_return
  OUTPUT_VARIABLE   openthread_git_hash)

list(APPEND ot_settings "OpenThread_commit=${openthread_git_hash}\n\n")

# Store all OT related variables
get_cmake_property(_variableNames VARIABLES)
list (SORT _variableNames)
foreach (_variableName ${_variableNames})
    if("${_variableName}" MATCHES "^CONFIG_OPENTHREAD_.*|^OT_.*") 
	    list(APPEND ot_settings "${_variableName}=${${_variableName}}\n")
    endif()
endforeach()

FILE(WRITE "${NRFXLIB_DIR}/openthread/openthread_lib_configuration.txt" ${ot_settings})

#TODO: replace with a proper glob
set(OPENTHREAD_LIBS
	"${PROJECT_BINARY_DIR}/modules/openthread/build/src/core/libopenthread-ftd.a"
	"${PROJECT_BINARY_DIR}/modules/openthread/build/src/core/libopenthread-mtd.a"
	"${PROJECT_BINARY_DIR}/modules/openthread/build/src/core/libopenthread-radio.a"
	"${PROJECT_BINARY_DIR}/modules/openthread/build/src/cli/libopenthread-cli-ftd.a"
	"${PROJECT_BINARY_DIR}/modules/openthread/build/src/cli/libopenthread-cli-mtd.a"
	"${PROJECT_BINARY_DIR}/modules/openthread/build/src/ncp/libopenthread-ncp-ftd.a"
	"${PROJECT_BINARY_DIR}/modules/openthread/build/src/ncp/libopenthread-ncp-mtd.a"
	"${PROJECT_BINARY_DIR}/modules/openthread/build/src/ncp/libopenthread-rcp.a"
)

# file(GLOB OPENTHREAD_LIBS FILES_MATCHING PATTERN
# 	"${PROJECT_BINARY_DIR}/modules/openthread/build/src/core/*.a"
# 	"${PROJECT_BINARY_DIR}/modules/openthread/build/src/cli/*.a"
# 	"${PROJECT_BINARY_DIR}/modules/openthread/build/src/ncp/*.a")

file (GLOB files "path/to/files/*" "path/to/files2*")

message(STATUS "OPENTHREAD_LIBS=${OPENTHREAD_LIBS}")

foreach(lib IN LISTS OPENTHREAD_LIBS)
    add_custom_command(
		TARGET app POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E copy "${lib}" "${NRFXLIB_DIR}/openthread/lib/${GCC_M_CPU}/${FLOAT_VARIANT}/v${OT_THREAD_VERSION}"
		COMMAND echo "Copying library: ${lib}"
		DEPENDS ${OPENTHREAD_LIBS})
endforeach()

#TODO: is it true?
if(CONFIG_CPU_HAS_FPU)
	set(FLOAT_VARIANT "hard-float")
else()
	set(FLOAT_VARIANT "soft-float")
endif()

file(COPY "${openthread_SOURCE_DIR}/include" 
	DESTINATION "${NRFXLIB_DIR}/openthread/lib/${GCC_M_CPU}/${FLOAT_VARIANT}/v${OT_THREAD_VERSION}"
	FILES_MATCHING PATTERN "*[!android].h")

file(COPY "${openthread_SOURCE_DIR}/examples/platforms/openthread-system.h" 
	DESTINATION "${NRFXLIB_DIR}/openthread/lib/${GCC_M_CPU}/${FLOAT_VARIANT}/v${OT_THREAD_VERSION}/include/"
	FILES_MATCHING PATTERN "*[!android].h")

message(STATUS "OT source dir=${openthread_SOURCE_DIR}")
message(STATUS "OT_LIB destination dir=${NRFXLIB_DIR}/openthread/lib/${GCC_M_CPU}/${FLOAT_VARIANT}/v${OT_THREAD_VERSION}")