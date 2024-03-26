# CPU Requirements

**NDI Lib** is heavily optimized (much of it is written in assembly). While it detects available architecture and uses the best path it can, the minimum required SIMD level is [SSSE3](https://en.wikipedia.org/wiki/SSSE3) (introduced by Intel in 2005).&#x20;

The NDI library running on ARM platforms requires NEON support. To the degree possible, hardware acceleration of streams uses GPU-based fixed function pipelines for decompression and is supported on Windows, macOS, iOS, and tvOS platforms; all GPUs on these platforms are supported with special case-optimized support for Intel QuickSync and nVidia.&#x20;

However, this is not required, and we will always fall back to software-based compression and decompression.
