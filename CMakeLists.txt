# SPDX-FileCopyrightText: Copyright (c) 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
# SPDX-License-Identifier: Apache-2.0

cmake_minimum_required(VERSION 3.24)

project(nsight_perf_sdk_for_ai)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

set(CMAKE_CUDA_STANDARD 17)
set(CMAKE_CUDA_STANDARD_REQUIRED ON)
set(CMAKE_CUDA_EXTENSIONS OFF)
set(CMAKE_CUDA_ARCHITECTURES 100)


if(PROJECT_IS_TOP_LEVEL)
    enable_language(CUDA)

    if(NOT CMAKE_CONFIGURATION_TYPES AND NOT CMAKE_BUILD_TYPE)
        set(CMAKE_BUILD_TYPE Release CACHE STRING "Build type" FORCE)
        set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS
            Debug Release RelWithDebInfo MinSizeRel)
    endif()
endif()

if(NOT TARGET CUDA::toolkit)
    find_package(CUDAToolkit REQUIRED)
endif()
if(NOT TARGET Threads::Threads)
    find_package(Threads REQUIRED)
endif()

function(nvtrace_link_cuda_runtime target)
    set_property(TARGET ${target} PROPERTY CUDA_RUNTIME_LIBRARY None)
    target_link_libraries(${target} PRIVATE CUDA::cudart_static)
endfunction()

add_subdirectory(third_party)

if(PROJECT_IS_TOP_LEVEL)
    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")
    set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")
    set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")
endif()

if(NOT TARGET nvtx3)
    if(NOT NVTX3_INCLUDE_DIR)
        if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/NVTX/c/include")
            set(NVTX3_INCLUDE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/NVTX/c/include")
        else()
            message(FATAL_ERROR
                "NVTX headers not found. Initialize the NVTX submodule or set NVTX3_INCLUDE_DIR to its include directory.")
        endif()
    endif()
    add_library(nvtx3 INTERFACE)
    target_include_directories(nvtx3 INTERFACE ${NVTX3_INCLUDE_DIR})
    target_link_libraries(nvtx3 INTERFACE ${CMAKE_DL_LIBS})
endif()

if(NOT NVPERF_CORE_INCLUDE_DIR)
    if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/nvperf-core/include")
        set(NVPERF_CORE_INCLUDE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/nvperf-core/include")
    else()
        message(FATAL_ERROR
            "nvperf-core headers not found. Initialize the nvperf-core submodule or set NVPERF_CORE_INCLUDE_DIR to its include directory.")
    endif()
endif()

if(CMAKE_SYSTEM_NAME STREQUAL "Linux" AND CMAKE_SYSTEM_PROCESSOR MATCHES "x86_64|AMD64")
    set(NVPERF_CORE_PLATFORM linux-desktop-x64)
elseif(CMAKE_SYSTEM_NAME STREQUAL "Linux" AND CMAKE_SYSTEM_PROCESSOR MATCHES "aarch64|arm64")
    set(NVPERF_CORE_PLATFORM linux-desktop-a64)
else()
    message(FATAL_ERROR "Unsupported platform for nvperf-core headers")
endif()

add_library(nvperf_core INTERFACE)
target_include_directories(nvperf_core INTERFACE
    ${NVPERF_CORE_INCLUDE_DIR}
    ${NVPERF_CORE_INCLUDE_DIR}/${NVPERF_CORE_PLATFORM})
unset(NVPERF_CORE_PLATFORM)

add_library(nvtrace_client INTERFACE)
target_include_directories(nvtrace_client INTERFACE "${CMAKE_CURRENT_SOURCE_DIR}/nvtrace-client/include")
target_link_libraries(nvtrace_client INTERFACE flatbuffers)

add_library(nvtrace_client_offline INTERFACE)
if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
    target_compile_definitions(nvtrace_client_offline INTERFACE NVTRACE_ENABLE_CXA_DEMANGLE=1)
else()
    target_compile_definitions(nvtrace_client_offline INTERFACE NVTRACE_ENABLE_CXA_DEMANGLE=0)
endif()
target_link_libraries(nvtrace_client_offline INTERFACE ${CMAKE_DL_LIBS} nvperf_core nvtrace_client perfetto_trace_proto)

add_library(NvTraceConsumerDuckDb INTERFACE)
target_link_libraries(NvTraceConsumerDuckDb INTERFACE nvtrace_client_offline duckdb_static)

add_subdirectory(nvtrace-samples-cuda/benchmarks)
add_subdirectory(nvtrace-samples-cuda/tests/NvTraceTestUtilities)
add_subdirectory(nvtrace-samples-cuda/tests/DataModelValidator)
add_subdirectory(nvtrace-samples-cuda/tests/NvTraceContinuousProfiling)

add_subdirectory(nvtrace-tools/NvTraceToolUtilities)
add_subdirectory(nvtrace-tools/UptimeTester)
add_subdirectory(nvtrace-tools/NvtxCudaDriverInjectionProxy)
add_subdirectory(nvtrace-tools/PerfettoTraceExporter)
add_subdirectory(nvtrace-tools/PerfettoTraceMerger)
add_subdirectory(nvtrace-tools/PollUntilServiceReady)
add_subdirectory(nvtrace-tools/SnapshotConsumer)
add_subdirectory(nvtrace-tools/ContinuousExport)
add_subdirectory(nvtrace-tools/StreamingConsumer)

function(nvtrace_find_sha256  SUMS  ARCHIVE  OUT_VAR)
    set(Sums "${SUMS}")
    set_property(DIRECTORY APPEND PROPERTY CMAKE_CONFIGURE_DEPENDS "${Sums}")
    file(STRINGS "${Sums}" Lines REGEX "^[0-9a-fA-F]+ [ *]")
    set(Hash "")
    foreach(Line IN LISTS Lines)
        if(Line MATCHES "^([0-9a-fA-F]+) [ *](.+)$" AND CMAKE_MATCH_2 STREQUAL ARCHIVE)
            set(Hash "${CMAKE_MATCH_1}")
        endif()
    endforeach()
    string(LENGTH "${Hash}" HashLength)
    if(NOT HashLength EQUAL 64)
        message(FATAL_ERROR "No valid SHA256 for ${ARCHIVE} in ${Sums}")
    endif()
    set(${OUT_VAR} "${Hash}" PARENT_SCOPE)
endfunction()

if(PROJECT_IS_TOP_LEVEL)
    include(FetchContent)

    set(NSIGHT_PERF_DAEMON_VERSION beta-preview)

    if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
        if(CMAKE_SYSTEM_PROCESSOR MATCHES "x86_64|AMD64")
            set(NSIGHT_PERF_DAEMON_PLATFORM linux-x86_64)
        elseif(CMAKE_SYSTEM_PROCESSOR MATCHES "aarch64|arm64")
            set(NSIGHT_PERF_DAEMON_PLATFORM linux-aarch64)
        else()
            message(FATAL_ERROR "Unsupported Linux architecture for the Nsight Perf daemon")
        endif()
        set(NSIGHT_PERF_DAEMON_EXTENSION tar.gz)
        set(NSIGHT_PERF_DAEMON_BINARIES nvidia-nsight-perf libnvperf_grfx_host.so)
    else()
        message(FATAL_ERROR "Unsupported platform for the Nsight Perf daemon")
    endif()

    set(NSIGHT_PERF_DAEMON_ARCHIVE
        nsight-perf-daemon-${NSIGHT_PERF_DAEMON_VERSION}-${NSIGHT_PERF_DAEMON_PLATFORM}.${NSIGHT_PERF_DAEMON_EXTENSION})

    nvtrace_find_sha256(${CMAKE_CURRENT_SOURCE_DIR}/nsight-perf-daemon/SHA256SUMS
        ${NSIGHT_PERF_DAEMON_ARCHIVE} NSIGHT_PERF_DAEMON_SHA256)

    FetchContent_Declare(nsight_perf_daemon
        URL https://github.com/NVIDIA/nsight-perf-daemon/releases/download/${NSIGHT_PERF_DAEMON_VERSION}/${NSIGHT_PERF_DAEMON_ARCHIVE}
        URL_HASH SHA256=${NSIGHT_PERF_DAEMON_SHA256}
        DOWNLOAD_EXTRACT_TIMESTAMP TRUE
    )
    FetchContent_MakeAvailable(nsight_perf_daemon)

    foreach(binary IN LISTS NSIGHT_PERF_DAEMON_BINARIES)
        file(COPY "${nsight_perf_daemon_SOURCE_DIR}/bin/${binary}" DESTINATION "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}")
    endforeach()

    # Place all shipped metric configs beside the service binary.
    file(GLOB NSIGHT_PERF_DAEMON_CONFIGS
        "${nsight_perf_daemon_SOURCE_DIR}/bin/*.config")
    file(COPY ${NSIGHT_PERF_DAEMON_CONFIGS}
        DESTINATION "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}")
    unset(NSIGHT_PERF_DAEMON_CONFIGS)

    unset(NSIGHT_PERF_DAEMON_PLATFORM)
    unset(NSIGHT_PERF_DAEMON_EXTENSION)
    unset(NSIGHT_PERF_DAEMON_ARCHIVE)
    unset(NSIGHT_PERF_DAEMON_SHA256)
    unset(NSIGHT_PERF_DAEMON_BINARIES)
endif()
