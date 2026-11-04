# SPDX-FileCopyrightText: Copyright (c) 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
# SPDX-License-Identifier: Apache-2.0

# GoogleTest v1.14.0 — https://github.com/google/googletest
if(TARGET googletest)
  return()
endif()

if (NOT TARGET gtest)
  if(FETCHCONTENT_SOURCE_DIR_GTEST)
    message(STATUS "Using local GoogleTest source: ${FETCHCONTENT_SOURCE_DIR_GTEST}")
  elseif(FETCHCONTENT_FULLY_DISCONNECTED)
    message(STATUS "FetchContent is offline; GoogleTest will require a pre-populated source")
  else()
    message(STATUS "Downloading gtest")
  endif()

  FetchContent_Declare(gtest
    URL https://github.com/google/googletest/archive/refs/tags/v1.14.0.tar.gz
    URL_HASH SHA256=8ad598c73ad796e0d8280b082cebd82a630d73e73cd3c70057938a6501bba5d7
    DOWNLOAD_EXTRACT_TIMESTAMP TRUE
  )

  set(BUILD_GMOCK OFF CACHE BOOL "" FORCE)
  set(INSTALL_GTEST OFF CACHE BOOL "" FORCE)
  set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)

  FetchContent_MakeAvailable(gtest)
endif()

# Client expects target googletest
add_library(googletest ALIAS gtest)
