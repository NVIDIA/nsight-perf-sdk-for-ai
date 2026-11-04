# SPDX-FileCopyrightText: Copyright (c) 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
# SPDX-License-Identifier: Apache-2.0

# Header-only cxxopts v3.3.1 — https://github.com/jarro2783/cxxopts
if(TARGET cxxopts)
  return()
endif()

if(FETCHCONTENT_SOURCE_DIR_CXXOPTS)
  message(STATUS "Using local cxxopts source: ${FETCHCONTENT_SOURCE_DIR_CXXOPTS}")
elseif(FETCHCONTENT_FULLY_DISCONNECTED)
  message(STATUS "FetchContent is offline; cxxopts will require a pre-populated source")
else()
  message(STATUS "Downloading cxxopts")
endif()

FetchContent_Declare(cxxopts
  URL https://github.com/jarro2783/cxxopts/archive/refs/tags/v3.3.1.tar.gz
  URL_HASH SHA256=3bfc70542c521d4b55a46429d808178916a579b28d048bd8c727ee76c39e2072
  DOWNLOAD_EXTRACT_TIMESTAMP TRUE
)

FetchContent_MakeAvailable(cxxopts)
