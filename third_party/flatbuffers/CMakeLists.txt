# SPDX-FileCopyrightText: Copyright (c) 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
# SPDX-License-Identifier: Apache-2.0

# FlatBuffers v23.5.26 — https://github.com/google/flatbuffers
if(TARGET flatbuffers)
  return()
endif()

if(FETCHCONTENT_SOURCE_DIR_FLATBUFFERS)
  message(STATUS "Using local FlatBuffers source: ${FETCHCONTENT_SOURCE_DIR_FLATBUFFERS}")
elseif(FETCHCONTENT_FULLY_DISCONNECTED)
  message(STATUS "FetchContent is offline; FlatBuffers will require a pre-populated source")
else()
  message(STATUS "Downloading flatbuffers")
endif()

FetchContent_Declare(flatbuffers
  URL https://github.com/google/flatbuffers/archive/refs/tags/v23.5.26.tar.gz
  URL_HASH SHA256=1cce06b17cddd896b6d73cc047e36a254fb8df4d7ea18a46acf16c4c0cd3f3f3
  DOWNLOAD_EXTRACT_TIMESTAMP TRUE
)

set(FLATBUFFERS_BUILD_FLATC OFF CACHE BOOL "" FORCE)
set(FLATBUFFERS_BUILD_TESTS OFF CACHE BOOL "" FORCE)
set(FLATBUFFERS_INSTALL OFF CACHE BOOL "" FORCE)

FetchContent_MakeAvailable(flatbuffers)
