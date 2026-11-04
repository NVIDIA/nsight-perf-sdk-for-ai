# SPDX-FileCopyrightText: Copyright (c) 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
# SPDX-License-Identifier: Apache-2.0

# Perfetto trace schema — the NvTrace subset of https://github.com/google/perfetto
if(TARGET perfetto_trace_proto)
  return()
endif()

set(PerfettoTraceProtoFile "${CMAKE_CURRENT_SOURCE_DIR}/perfetto_trace.proto")
if(NOT EXISTS "${PerfettoTraceProtoFile}")
  message(FATAL_ERROR "Missing ${PerfettoTraceProtoFile}")
endif()

if(NOT TARGET protobuf::libprotobuf)
  if(FETCHCONTENT_SOURCE_DIR_PROTOBUF)
    message(STATUS "Using local Protobuf source: ${FETCHCONTENT_SOURCE_DIR_PROTOBUF}")
  elseif(FETCHCONTENT_FULLY_DISCONNECTED)
    message(STATUS "FetchContent is offline; Protobuf will require a pre-populated source")
  else()
    message(STATUS "Downloading protobuf")
  endif()

  # 3.21 is the last release line that does not pull in Abseil.
  FetchContent_Declare(protobuf
    URL https://github.com/protocolbuffers/protobuf/releases/download/v21.12/protobuf-cpp-3.21.12.tar.gz
    URL_HASH SHA256=4eab9b524aa5913c6fffb20b2a8abf5ef7f95a80bc0701f3a6dbb4c607f73460
    DOWNLOAD_EXTRACT_TIMESTAMP TRUE
  )

  set(protobuf_BUILD_TESTS OFF CACHE BOOL "" FORCE)
  set(protobuf_BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)
    set(protobuf_BUILD_PROTOC_BINARIES ON CACHE BOOL "" FORCE)
  set(protobuf_INSTALL OFF CACHE BOOL "" FORCE)
  set(protobuf_WITH_ZLIB OFF CACHE BOOL "" FORCE)
  # protobuf 3.21 declares cmake_minimum_required(VERSION 3.5). CMake 4.x
  # deprecates compatibility below 3.10; use policy behavior through 3.10
  # without modifying the vendored protobuf sources.
  set(CMAKE_POLICY_VERSION_MINIMUM 3.10)
  set(CMAKE_WARN_DEPRECATED OFF CACHE BOOL "" FORCE)
  # libprotobuf ends up inside the cpp_data_model Python module, a shared object.
  set(CMAKE_POSITION_INDEPENDENT_CODE ON)

  FetchContent_MakeAvailable(protobuf)

  # protobuf is third-party code the SDK does not maintain
  foreach(ProtobufTarget libprotobuf libprotobuf-lite libprotoc protoc)
    if(NOT TARGET ${ProtobufTarget})
      continue()
    endif()

      target_compile_options(${ProtobufTarget} PRIVATE -w)

    get_target_property(ProtobufIncludeDirs ${ProtobufTarget} INTERFACE_INCLUDE_DIRECTORIES)
    if(ProtobufIncludeDirs)
      set_property(TARGET ${ProtobufTarget} APPEND PROPERTY
        INTERFACE_SYSTEM_INCLUDE_DIRECTORIES "${ProtobufIncludeDirs}")
    endif()
  endforeach()
endif()

# Build a host-native Release protoc for codegen when the in-tree one cannot run.
if((CMAKE_CROSSCOMPILING
    ) AND protobuf_SOURCE_DIR)
  include(ExternalProject)

  set(HostProtocDir "${CMAKE_CURRENT_BINARY_DIR}/protoc_host/bin")
    set(HostProtocPath "${HostProtocDir}/protoc-3.21.12.0")
    set(HostProtocCFlags "-w")
    set(HostProtocCxxFlags "-w")

  ExternalProject_Add(protoc_host
    SOURCE_DIR "${protobuf_SOURCE_DIR}"
    PREFIX "${CMAKE_CURRENT_BINARY_DIR}/protoc_host"
    BINARY_DIR "${CMAKE_CURRENT_BINARY_DIR}/protoc_host/build"
    CMAKE_ARGS
      -DCMAKE_BUILD_TYPE=Release
      -DCMAKE_RUNTIME_OUTPUT_DIRECTORY=${HostProtocDir}
      -DCMAKE_RUNTIME_OUTPUT_DIRECTORY_RELEASE=${HostProtocDir}
      -DCMAKE_POLICY_VERSION_MINIMUM=3.10
      -DCMAKE_WARN_DEPRECATED=OFF
      "-DCMAKE_CXX_FLAGS=${HostProtocCxxFlags}"
      "-DCMAKE_C_FLAGS=${HostProtocCFlags}"
      -Dprotobuf_BUILD_TESTS=OFF
      -Dprotobuf_BUILD_EXAMPLES=OFF
      -Dprotobuf_BUILD_PROTOC_BINARIES=ON
      -Dprotobuf_INSTALL=OFF
      -Dprotobuf_WITH_ZLIB=OFF
    BUILD_COMMAND ${CMAKE_COMMAND} --build <BINARY_DIR> --config Release --target protoc
    INSTALL_COMMAND ""
    BUILD_BYPRODUCTS "${HostProtocPath}"
  )

  set(ProtocCommand "${HostProtocPath}")
  set(ProtocDependency protoc_host "${HostProtocPath}")
else()
  set(ProtocCommand "$<TARGET_FILE:protobuf::protoc>")
  set(ProtocDependency protobuf::protoc)
endif()

set(PerfettoTraceProtoGenDir "${CMAKE_CURRENT_BINARY_DIR}/generated")

add_custom_command(
  OUTPUT
    "${PerfettoTraceProtoGenDir}/perfetto_trace.pb.cc"
    "${PerfettoTraceProtoGenDir}/perfetto_trace.pb.h"
  COMMAND ${CMAKE_COMMAND} -E make_directory "${PerfettoTraceProtoGenDir}"
  COMMAND ${ProtocCommand}
    --cpp_out "${PerfettoTraceProtoGenDir}"
    -I "${CMAKE_CURRENT_SOURCE_DIR}"
    "${PerfettoTraceProtoFile}"
  DEPENDS "${PerfettoTraceProtoFile}" ${ProtocDependency}
  COMMENT "Generating perfetto_trace.pb.cc"
  VERBATIM
)

add_library(perfetto_trace_proto STATIC
  "${PerfettoTraceProtoGenDir}/perfetto_trace.pb.cc"
)

set_target_properties(perfetto_trace_proto PROPERTIES POSITION_INDEPENDENT_CODE ON)

# protoc output, not NvTrace-authored code.
  target_compile_options(perfetto_trace_proto PRIVATE -w)

target_include_directories(perfetto_trace_proto SYSTEM PUBLIC "${PerfettoTraceProtoGenDir}")

target_link_libraries(perfetto_trace_proto PUBLIC protobuf::libprotobuf)
