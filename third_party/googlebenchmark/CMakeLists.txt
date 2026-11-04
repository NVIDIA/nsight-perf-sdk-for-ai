# SPDX-FileCopyrightText: Copyright (c) 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
# SPDX-License-Identifier: Apache-2.0

# Google Benchmark v1.9.5 — https://github.com/google/benchmark
if(TARGET googlebenchmark)
  return()
endif()

if(NOT TARGET benchmark)
  if(FETCHCONTENT_SOURCE_DIR_GOOGLEBENCHMARK)
    message(STATUS "Using local Google Benchmark source: ${FETCHCONTENT_SOURCE_DIR_GOOGLEBENCHMARK}")
  elseif(FETCHCONTENT_FULLY_DISCONNECTED)
    message(STATUS "FetchContent is offline; Google Benchmark will require a pre-populated source")
  else()
    message(STATUS "Downloading googlebenchmark")
  endif()

  FetchContent_Declare(googlebenchmark
    URL https://github.com/google/benchmark/archive/refs/tags/v1.9.5.tar.gz
    URL_HASH SHA256=9631341c82bac4a288bef951f8b26b41f69021794184ece969f8473977eaa340
    DOWNLOAD_EXTRACT_TIMESTAMP TRUE
  )

  set(BENCHMARK_ENABLE_TESTING OFF CACHE BOOL "" FORCE)
  set(BENCHMARK_ENABLE_INSTALL OFF CACHE BOOL "" FORCE)
  set(BENCHMARK_ENABLE_WERROR OFF CACHE BOOL "" FORCE)

  # Pre-define HAVE_* to bypass upstream googlebenchmark's try_run / try_compile feature
  # detection, which can fail (or warn) in environments where the compiled probe binary
  # can't execute (sandboxed builders, cross-compile). See google/benchmark
  # cmake/CXXFeatureCheck.cmake.
  set(HAVE_STD_REGEX 1 CACHE INTERNAL "")
  set(HAVE_STEADY_CLOCK 1 CACHE INTERNAL "")
    set(HAVE_PTHREAD_AFFINITY 1 CACHE INTERNAL "")

  FetchContent_MakeAvailable(googlebenchmark)
endif()

# Client expects target googlebenchmark; upstream creates benchmark + benchmark_main.
add_library(googlebenchmark ALIAS benchmark_main)
