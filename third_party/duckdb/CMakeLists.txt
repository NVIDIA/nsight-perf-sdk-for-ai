# SPDX-FileCopyrightText: Copyright (c) 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
# SPDX-License-Identifier: Apache-2.0

# DuckDB v1.5.5 — https://github.com/duckdb/duckdb
if(TARGET duckdb_static)
  return()
endif()

if(FETCHCONTENT_SOURCE_DIR_DUCKDB)
  message(STATUS "Using local DuckDB source: ${FETCHCONTENT_SOURCE_DIR_DUCKDB}")
elseif(FETCHCONTENT_FULLY_DISCONNECTED)
  message(STATUS "FetchContent is offline; DuckDB will require a pre-populated source")
else()
  message(STATUS "Downloading DuckDB")
endif()

# SOURCE_SUBDIR names a directory that does not exist so that FetchContent_MakeAvailable only
# populates the source; DuckDB is added below with EXCLUDE_FROM_ALL so that only what the SDK
# links is built.
FetchContent_Declare(duckdb
  URL https://github.com/duckdb/duckdb/archive/refs/tags/v1.5.5.tar.gz
  URL_HASH SHA256=f33155ff962e6e1e08fd1e9caffa487d4325aa60999e2eabc76feff534d6558b
  DOWNLOAD_EXTRACT_TIMESTAMP TRUE
  SOURCE_SUBDIR nvtrace-populate-only
)
FetchContent_MakeAvailable(duckdb)

# The source tarball has no .git for DuckDB to describe.
set(OVERRIDE_GIT_DESCRIBE v1.5.5-0-gd8cdaa33fd)
set(BUILD_SHELL OFF)
set(BUILD_UNITTESTS OFF)
set(BUILD_BENCHMARKS OFF)
set(ENABLE_SANITIZER OFF)
set(ENABLE_UBSAN OFF)
set(ENABLE_JEMALLOC OFF)

  # DuckDB adds -DDEBUG to Debug builds, enabling its assertions and internal verification.
  add_compile_options(-w -g0 -UDEBUG)

# DuckDB force-caches ccache/sccache from PATH as the compiler launcher for the whole build when
# none is defined.
if(NOT DEFINED CMAKE_C_COMPILER_LAUNCHER)
  set(CMAKE_C_COMPILER_LAUNCHER "")
endif()
if(NOT DEFINED CMAKE_CXX_COMPILER_LAUNCHER)
  set(CMAKE_CXX_COMPILER_LAUNCHER "")
endif()

# re2's include(CTest) would otherwise cache BUILD_TESTING=ON and add the CTest dashboard targets.
set(BUILD_TESTING OFF)

# DuckDB overwrites the global JOB_POOLS property, dropping pools defined by other projects.
get_property(nvtrace_duckdb_job_pools GLOBAL PROPERTY JOB_POOLS)
add_subdirectory(${duckdb_SOURCE_DIR} ${duckdb_BINARY_DIR} EXCLUDE_FROM_ALL)
set_property(GLOBAL APPEND PROPERTY JOB_POOLS ${nvtrace_duckdb_job_pools})


# DuckDB adds its include paths with include_directories(), which does not reach consumers.
target_include_directories(duckdb_static SYSTEM INTERFACE $<BUILD_INTERFACE:${duckdb_SOURCE_DIR}/src/include>)
target_compile_definitions(duckdb_static INTERFACE DUCKDB_STATIC_BUILD)
# duckdb_static leaves out the extension loader and the built-in extensions from
# extension/extension_config.cmake; DuckDB's own executables add them with link_extension_libraries().
# The loader references the extensions, so it must precede them on single-pass linker command lines.
target_link_libraries(duckdb_generated_extension_loader INTERFACE
  $<BUILD_INTERFACE:core_functions_extension>
  $<BUILD_INTERFACE:parquet_extension>)
target_link_libraries(duckdb_static INTERFACE $<BUILD_INTERFACE:duckdb_generated_extension_loader>)
