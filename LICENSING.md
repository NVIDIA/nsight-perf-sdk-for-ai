# Licensing

NVIDIA-authored source, documentation, and build orchestration in this repository
are licensed under the Apache License, Version 2.0. See [LICENSE](LICENSE).
The component repositories and their release assets retain their own terms:

| Component | License boundary |
| --- | --- |
| [NvTrace Client](https://github.com/NVIDIA/nvtrace-client/blob/main/README.md) | NVIDIA-authored source: Apache-2.0; see its [LICENSE](https://github.com/NVIDIA/nvtrace-client/blob/main/LICENSE) and [third-party notices](https://github.com/NVIDIA/nvtrace-client/blob/main/THIRD_PARTY_NOTICES.md). |
| [NvTrace Tools](https://github.com/NVIDIA/nvtrace-tools/blob/main/README.md) | NVIDIA-authored source: Apache-2.0; see its [LICENSE](https://github.com/NVIDIA/nvtrace-tools/blob/main/LICENSE) and [third-party notices](https://github.com/NVIDIA/nvtrace-tools/blob/main/THIRD_PARTY_NOTICES.md). |
| [NvTrace CUDA Samples](https://github.com/NVIDIA/nvtrace-samples-cuda/blob/main/README.md) | NVIDIA-authored SDK code: Apache-2.0; imported/adapted `cuda-samples/` files retain their BSD-style terms in [cuda-samples/LICENSE](https://github.com/NVIDIA/nvtrace-samples-cuda/blob/main/cuda-samples/LICENSE). |
| [Nsight Perf Daemon](https://github.com/NVIDIA/nsight-perf-daemon/blob/main/README.md) | Repository-authored documentation and packaging material: Apache-2.0. Separately distributed daemon executables: proprietary Nsight Perf SDK EULA; see [LICENSING.md](https://github.com/NVIDIA/nsight-perf-daemon/blob/main/LICENSING.md). |
| [NvPerf Core](https://github.com/NVIDIA/nvperf-core) | Headers and binary assets retain the terms in its own [licensing document](https://github.com/NVIDIA/nvperf-core/blob/main/LICENSING.md). |
| [NVTX](https://github.com/NVIDIA/NVTX) | Apache-2.0 WITH LLVM-exception; see the pinned upstream license and [third-party notices](THIRD_PARTY_NOTICES.md). |

Third-party dependencies are identified in
[THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md), and their license texts and
source attributions are reproduced in
[THIRD_PARTY_LICENSES.md](THIRD_PARTY_LICENSES.md). Each proprietary binary bundle
must carry its applicable EULA and dependency notices. See
[daemon notices](nsight-perf-daemon/THIRD_PARTY_NOTICES.md) and
[PerfWorks runtime notices](nsight-perf-daemon/THIRD_PARTY_NOTICES.md#perfworks-runtime).
The aggregate Apache license does not relicense component or binary assets.
