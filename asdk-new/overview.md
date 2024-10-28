# Overview

This section of the NDI Advanced SDK is designed for use by device manufacturers who wish to provide hardware assisted encoding or decoding. It follows the same API as the NDI SDK so that any experience, sample code, documentation from one can be applied to the other.

Importantly, this part of the NDI Advanced SDK provides direct access to the video data in compressed form so that it can be sent and received directly. Thus, it is also likely that this SDK can be used for other tasks that require or benefit from interacting directly with the compressed video data for sending and receiving.

There are currently two primary uses of the NDI Advanced SDK:

1.  You can use NDI compression (sometimes called "<mark style="color:red;">SpeedHQ</mark>"), a high performance and high quality I-Frame video codec. An FPGA compressor for native NDI is provided within this SDK.

    If you are using SpeedHQ compression, you are likely also using a (SoC) device with an ARM core and an FPGA that can be used for real-time compression. The NDI Advanced SDK provides NDI Encode and Decode IP cores for Xilinx and Altera FPGAs, along with example FPGA projects and reference C++ applications including full source code.

    To help get you started, prebuilt bootable uSD drive images for several standard Development kits are provided. The FPGA core supplied will easily encode 4K video in real-time on relatively modest FPGA designs, assuming the device has sufficient memory bandwidth (multiple banks of RAM are recommended); on latest generation SoC systems it can easily encode one or more streams of 8K if sufficient network bandwidth is provided.
2. You may use H.264 or H.265 for video and AAC audio with this SDK if you are developing using devices that already have hardware compressors for these available.

{% hint style="info" %}
Because many Advanced SDK systems require custom tool chains, NDI can provide a compiled version of the SDK expressly built for your specific system. Please email [sdk@ndi.video](mailto:sdk@ndi.video) if you have such requirements.
{% endhint %}
