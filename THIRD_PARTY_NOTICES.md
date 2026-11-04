# Third-Party Notices

The SDK uses the dependencies below. Upstream references are listed below;
full license texts and source attributions accompany this repository in
[THIRD_PARTY_LICENSES.md](THIRD_PARTY_LICENSES.md).

| Dependency | Version/provenance | Use | License | License text |
| --- | --- | --- | --- | --- |
| cxxopts | 3.3.1 | Command-line parsing | MIT | [license](https://github.com/jarro2783/cxxopts/blob/v3.3.1/LICENSE) |
| DuckDB | 1.5.5; v1.5.5 source tree with the built-in `core_functions` and `parquet` extensions | DuckDB table export and Parquet export | MIT; bundled dependencies and extensions under their own terms | [license](https://github.com/duckdb/duckdb/blob/v1.5.5/LICENSE), [bundled dependencies](https://github.com/duckdb/duckdb/tree/v1.5.5/third_party), [extensions](https://github.com/duckdb/duckdb/tree/v1.5.5/extension) |
| FlatBuffers | 23.5.26 | Runtime headers, schemas, and generated NvTrace headers | Apache-2.0 | [license](https://github.com/google/flatbuffers/blob/v23.5.26/LICENSE) |
| GCC runtime libraries | GCC 15.2.0 Linux release toolchain | Runtime code incorporated into SDK-built binaries; daemon attribution is supplied separately | GPL-3.0-or-later WITH GCC-exception-3.1 | [runtime license and exception](THIRD_PARTY_LICENSES.md#gcc-runtime-libraries) |
| zstd (Zstandard) | 1.5.6 bundled in the DuckDB 1.5.5 source tree | Compression used by DuckDB-backed table/Parquet export in `nvtrace_continuous_export` | BSD-3-Clause elected; MIT for bundled divsufsort code | [copyright and terms](THIRD_PARTY_LICENSES.md#duckdb--zstd) |
| Google Benchmark | 1.9.5 | Benchmarks | Apache-2.0 | [license](https://github.com/google/benchmark/blob/v1.9.5/LICENSE) |
| GoogleTest | 1.14.0 | Validation tests | BSD-3-Clause | [license](https://github.com/google/googletest/blob/v1.14.0/LICENSE) |
| NVTX C/C++ | 3.7.0 | Performance annotations and injection proxy | Apache-2.0 WITH LLVM-exception | [license](https://github.com/NVIDIA/NVTX/blob/v3.7.0/LICENSE.txt) |
| Protocol Buffers | 3.21.12 (upstream tag v21.12) | Perfetto schema compiler and C++ runtime | BSD-3-Clause | [license](https://github.com/protocolbuffers/protobuf/blob/v21.12/LICENSE) |
| Perfetto | Minimal trace-schema subset; documented provenance baseline v58.3, commit `11fbaed836361258f23604daf67c837f3034e663` | NVIDIA-modified `third_party/perfetto/perfetto_trace.proto` | Apache-2.0 | [license](https://github.com/google/perfetto/blob/11fbaed836361258f23604daf67c837f3034e663/LICENSE) |

The SDK includes an NVIDIA-modified minimal Perfetto trace protobuf schema.
Preserve upstream notices in fetched source and generated/bundled distributions.

The samples component contains imported/adapted NVIDIA CUDA Samples source under
BSD-style terms. See [cuda-samples/LICENSE](https://github.com/NVIDIA/nvtrace-samples-cuda/blob/main/cuda-samples/LICENSE)
and [the samples notices](https://github.com/NVIDIA/nvtrace-samples-cuda/blob/main/THIRD_PARTY_NOTICES.md).
The independently supplied NvPerf Core headers/runtime retain their own terms.

## Binary component notices

Daemon dependencies and license terms are listed in
[the daemon notices](nsight-perf-daemon/THIRD_PARTY_NOTICES.md).
PerfWorks runtime dependencies and license terms are listed in
[the runtime notices](nsight-perf-daemon/THIRD_PARTY_NOTICES.md#perfworks-runtime).
Redistribute these notices and the applicable EULA along with any
redistribution of the corresponding components.

When redistributing SDK-built binaries, include the applicable dependency
license and notice texts, including those for DuckDB's bundled dependencies
and selected extensions.

## Apache notices for DuckDB-bundled components

When distributing code or binaries incorporating these DuckDB-bundled
components, preserve the following upstream notices. These components are
licensed under Apache-2.0; see [LICENSE](LICENSE).

```text
Apache Arrow ADBC
Copyright 2022 The Apache Software Foundation

Apache Arrow nanoarrow
Copyright 2023 The Apache Software Foundation

Apache Parquet Format
Copyright 2014 The Apache Software Foundation

Apache Thrift
Copyright (C) 2006 - 2019, The Apache Software Foundation

This product includes software developed at
The Apache Software Foundation (http://www.apache.org/).
```

## Repository policy documents

`CODE_OF_CONDUCT.md` is based on Contributor Covenant version 1.4 and retains its
upstream attribution and license link. `CONTRIBUTING.md` reproduces Developer
Certificate of Origin version 1.1, copyright 2004 and 2006 The Linux Foundation
and its contributors, under the permission stated in that text.
