# CPU Requirements

**NDI Lib** is heavily optimized (much of it is written in assembly). While it detects available architecture and uses the best path it can, the minimum required SIMD[^1] level is [SSSE3](https://en.wikipedia.org/wiki/SSSE3) (introduced by Intel in 2005).&#x20;

The NDI library running on ARM platforms requires NEON support. To the degree possible, hardware acceleration of streams uses GPU-based fixed function pipelines for decompression and is supported on Windows, macOS, iOS, and tvOS platforms; all GPUs on these platforms are supported with special case-optimized support for Intel QuickSync and nVidia.&#x20;

However, this is not required, and we will always fall back to software-based compression and decompression.

[^1]: Single instruction, multiple data (SIMD) is a type of [parallel processing](https://en.wikipedia.org/wiki/Parallel\_computer) in [Flynn's taxonomy](https://en.wikipedia.org/wiki/Flynn's\_taxonomy). SIMD can be internal (part of the hardware design) and it can be directly accessible through an [instruction set architecture](https://en.wikipedia.org/wiki/Instruction\_set\_architecture) (ISA), but it should not be confused with an ISA. SIMD describes computers with [multiple processing elements](https://en.wikipedia.org/wiki/Multiple\_processing\_elements) that perform the same operation on multiple data points simultaneously.
