# NDI FPGA Reference Design

The NDI Advanced SDK contains working example designs intended to assist in using the Advanced NDI software SDK and the FPGA NDI IP cores.

Building a working embedded NDI design requires wide-ranging expertise and these example designs are provided in an attempt to make this process more accessible. Prebuilt uSD card images are available for the supported platforms, and details are provided regarding how to rebuild each required piece from scratch.

## Quick Start

1. Obtain one of the supported development boards:
   * [Altera Arria-10 SoC Devkit](https://www.intel.com/content/www/us/en/products/details/fpga/development-kits/arria/10-sx.html)
   * [Digilent Zybo-Z7-20](https://store.digilentinc.com/zybo-z7-zynq-7000-arm-fpga-soc-development-board/)
   * [Digilent Arty-Z7-20](https://digilent.com/shop/arty-z7-zynq-7000-soc-development-board/) (partial support, the Zybo-Z7-20 is preferred)
   * [Xilinx ZCU104](https://www.xilinx.com/products/boards-and-kits/zcu104.html)
2.  Boot your board using a prebuilt image

    See the README.uSD.md file for details on obtaining the uSD image, writing it to a uSD card, and using it with one of the supported platforms.

## Directory structure:

| Filename        | Contents                                                    |
| --------------- | ----------------------------------------------------------- |
| `README.md`     | High level overview of the provided example projects        |
| `README.uSD.md` | Details on using the prebuilt uSD images                    |
| `CHANGELOG.md`  | List of notable changes                                     |
| `src/cpp`       | Software source code and example projects                   |
| `src/fpga`      | Hardware design files and example projects                  |
| `linux_kernel`  | Projects to build kernel and boot loader                    |
| `os_uSD`        | Scripts to generate root filesystem and bootable uSD images |

## Design Overview

There are a lot of different pieces required to create a working example system:

* FPGA hardware design
* Boot loader
* Linux kernel
* Device-tree
* Linux root file system
* Application software
* Bootable image including all of the above

While each of these components is important and necessary, not all of them have to be customized in order to create a custom project. The three components that need to be customized for almost any project are the actual FPGA hardware, the application software, and the device tree. The Linux kernel and boot loader can typically be used without customizations and there are several options for creating a Linux root file system.

This example design uses vendor specific processes to generate the boot loader, the Linux kernel, and most of the device tree. The root file system is a standard Debian install. The FPGA logic and application software are custom, and some device-tree customizations are required to support the custom FPGA hardware. Each of these components is discussed in more detail below.

## Theory of Operation

Embedded systems require interaction between hardware and software. This is a complex process even on small micro-controllers, and is even more so on systems running a high-level OS such as Linux. This example design was created with the intent to make the interaction between hardware and software as simple as possible to implement and modify, with no need to write kernel-space code.

### Common Encode and Decode Base Functionality

#### Low Level Details and Device-Tree

In embedded systems, it is necessary to communicate details of the system across several fundamentally different environments (e.g., hardware, the Linux kernel, and application software). Rather than using lots of "magic" numbers stashed in cryptic header files, this is now mostly done at the system level with device-tree. Details provided via device-tree can control which kernel modules get loaded (or do not), as well as modify the behavior of those modules.

**Hardware IP**

*   Vendor IP

    Standard vendor IP blocks for GPIO and I2C are implemented along with the Hard IP ARM cores in the example design files. These vendor IP blocks are added to the device-tree and are controlled by standard Linux kernel drivers. This allows the pushbutton switches and LEDs connected to the FPGA fabric to be made available to the Linux kernel as part of the standard gpio subsystem.
*   Xilinx HDMI IP

    The ZCU104 uses the Xilinx HDMI IP core, however the Linux drivers for this core are disabled since the NDI encoding data flow operates outside of the Linux kernel's standard video input and output systems. The control software for the HDMI core instead operates on one of the Cortex-R5 cores, which means all the hardware used by the HDMI core (uart1, i2c0, i2c1) must be disabled in Linux to avoid contention.
*   Altera HDMI IP

    The Arria 10 examples use the Altera HDMI IP cores. Management of these IP cores is done outside of Linux using the NIOS CPU core instantiated as part of the Altera HDMI example design. This code is used as-is for HDMI Rx, and modified to allow configuration of the HDMI Tx logic to support several different resolutions for the NDI Decode example design.
*   Custom IP

    The NDI hardware implements a 64K block of register space which is manually added to the device-tree along with IRQ settings and other hardware details. The generic-uio driver is used to map the hardware register space and IRQs so they are available to user-space code.

**Reserved Memory Regions**

This design requires two large blocks of buffer memory, one for compressed video and raw audio data that must be visible to Linux, and one for raw video data which can be visible to Linux but may also be implemented as a separate memory bank for improved system performance.

*   NewTek\_Reserved

    This memory region is used to store compressed video data and raw audio data. This region is marked cacheable for performance. Cache coherency is maintained since the hardware accesses this memory region using a cache coherent bus interface.
*   NewTek\_Video

    This memory region is used to store raw (uncompressed) video data and thus requires high bandwidth. The application does not access this memory in normal operation so this region may be implemented as a dedicated FPGA-side memory bank to improve performance. This region will only appear in the device-tree if implemented using shared memory visible to Linux. If a live video input is not available and this memory region is visible to Linux, a video pattern can be written to allow testing of the compression core and software application.

**Software**

*   Standard Vendor IP Blocks

    The standard Vendor IP blocks (for I2C and GPIO) are added to the device tree and stock Linux kernel drivers are used to communicate with this hardware.
*   Custom IP blocks

    The device-tree entries for the custom FPGA IP use the generic-uio kernel driver. This driver makes the memory regions and IRQs specified by the device-tree entries accessible to user-space code. The libuio library is used to facilitate accessing the hardware.

    The application also reads details regarding the reserved memory regions directly from the device-tree, as this functionality is not supported by libuio.

#### Initial Startup

| File(s)                           | Description                                    |
| --------------------------------- | ---------------------------------------------- |
| `<device-tree>`                   | Contains memory addresses and IRQ details      |
| `src/cpp/ndi_common/hardware.*`   | Low-level access to register space and IRQs    |
| `src/cpp/ndi_common/device.*`     | Machine specific details                       |
| `src/fpga/ip/common/Version.vhd`  | Version information compiled into the hardware |
| `src/cpp/ndi_encode/ndi_encode.*` | Top level application file (Encode)            |
| `src/cpp/ndi_decode/ndi_decode.*` | Top level application file (Decode)            |

When the application is launched, quite a bit of setup is performed before continuing operation is passed to a number of created threads. Most of this setup happens when the hardware class is initialized. After the software processes any command-line options, an instance of the hardware class is created and initialized.

The hardware class uses libuio to access the memory regions and interrupts specified in the device-tree for the custom hardware IP. In addition, details for the reserved memory regions are read from the device-tree and the memory is mmap()'d to make it available for access. At this point, version information is read from the hardware and a device class specific to the hardware platform is created. This allows some behaviors to change between different physical boards (currently used to control any required ACP address translation). Finally, details regarding the UIO devices and reserved memory are printed to the console, and the reserved memory region for compressed video is written with the value 0xdeaddead, making it easier to identify uninitialized memory locations when debugging.

### Supported Video Formats

The NDI Encode and Decode FPGA cores support a variety of standard and custom in-memory video formats for 4:2:2 and 4:2:0 YUV video. Planar alpha is also supported for 4:2:2 formats. For further details, see the generate\_video() routine in src/cpp/ndi\_encode/video\_pattern.cpp.

#### UYVY

4:2:2 Packed 8-bit

```
Addr : Component
  00 : U0
  01 : Y0
  02 : V0
  03 : Y1
  ...
```

#### YUYV

4:2:2 Packed 8-bit, Y first

```
Addr : Component
  00 : Y0
  01 : U0
  02 : Y1
  03 : V0
  ...
```

#### NV16

4:2:2 Semi-planar 8-bit - luma then packed chroma

```
Plane 0 (Luma)
Addr : Component
  00 : Y0
  01 : Y1
  02 : Y2
  03 : Y3
  ...

Plane 1 (Chroma)
Addr : Component
  00 : U0
  01 : V0
  02 : U1
  03 : V1
  ...
```

#### UYVW

4:2:2 Packed 16-bit (custom format) - 16-bit version of UYVY

```
Addr : Component
  00 : Y0
  02 : U0
  04 : Y1
  06 : V0
  ...
```

#### Y216

4:2:2 Packed 16-bit, Y first - 16-bit version of YUYV

```
Addr : Component
  00 : Y0
  02 : U0
  04 : Y1
  06 : V0
  ...
```

#### P216

4:2:2 Semi-planar 16-bit - luma then packed chroma

```
Plane 0 (Luma)
Addr : Component
  00 : Y0
  02 : Y1
  04 : Y2
  06 : Y3
  ...

Plane 1 (Chroma)
Addr : Component
  00 : U0
  02 : V0
  04 : U1
  06 : V1
  ...
```

#### 420

4:2:0 Packed 8-bit (custom format) - Macroblock interleaved, 16 pixels of Y followed by 8 pixels of U (even lines) or V (odd lines)

```
Addr : Component
0x00 : Y0
0x01 : Y1
...
0x0E : Y14
0x0F : Y15
0x10 : U0
0x11 : U1
...
0x16 : U6
0x17 : U7

0x18 : Y16
0x19 : Y17
...
```

#### NV12

4:2:0 Semi-planar 8-bit - Luma then packed chroma

```
Plane 0 (Luma)
Addr : Component
  00 : Y0
  02 : Y1
  04 : Y2
  06 : Y3
  ...

Plane 1 (Chroma)
Addr : Component
  00 : U0
  02 : V0
  04 : U1
  06 : V1
  ...
```

#### 420W

4:2:0 Packed 16-bit (custom format) - Macroblock interleaved, 16 pixels of Y followed by 8 pixels of U (even lines) or V (odd lines)

```
Addr : Component
0x00 : Y0
0x02 : Y1
...
0x1C : Y14
0x1E : Y15
0x20 : U0
0x22 : U1
...
0x2C : U6
0x2E : U7

0x30 : Y16
0x32 : Y17
...
```

#### P016

4:2:0 Semi-planar 16-bit - Luma then packed chroma

```
Plane 0 (Luma)
Addr : Component
  00 : Y0
  02 : Y1
  04 : Y2
  06 : Y3
  ...

Plane 1 (Chroma)
Addr : Component
  00 : U0
  02 : V0
  04 : U1
  06 : V1
  ...
```

### NDI Encode Operation

#### Video Input

| File(s)                              | Description                           |
| ------------------------------------ | ------------------------------------- |
| `src/cpp/ndi_encode/video_capture.*` | `video_capture::` class               |
| `src/cpp/ndi_encode/track.*`         | `track::` class                       |
| `src/fpga/ip/common/Vid_Track.vhd`   | video format tracking hardware        |
| `src/fpga/ip/common/Vid_In.vhd`      | Video input DMA logic                 |
| `src/fpga/ip/common/Preview.vhd`     | Creates low-resolution preview stream |
| Filename varies                      | Low level video input logic           |

Video input starts with the low level video input logic. This logic is platform specific:

| Platform   | Dir | Implementation                  |
| ---------- | --- | ------------------------------- |
| a10socdk   | Rx  | Altera HDMI RxTx example design |
| ZCU104     | Rx  | Xilinx HDMI Rx example design   |
| Zybo-Z7-20 | Rx  | Open-source HDMI Rx logic       |

The software for controlling the low-level video input logic is currently outside the scope of this example. The Zybo-Z7 HDMI input requires no control software, the Xilinx HDMI control software is running bare-metal on one of the Cortex-R5 cores, and the Altera HDMI control software runs on a dedicated NIOS core (see the README file accompanying the hardware project files for details). There is, however, software support for low-level input monitoring code to send details (locked/unlocked, resolution, etc) to the `video_capture::` class. See `video_capture::set_signal()` and the `do_commands()` function in `ndi_send.cpp` for details.

Once the low-level video signal is received by the FPGA hardware, the signal is synchronized to the main clock and converted to a standard interface (`HDMI_T`). This signal is sent to the format tracking logic (`Vid_Track.vhd`) and is filtered (`Preview.vhd`) to create a low resolution preview stream. The resulting full and preview resolution streams are each sent to a bus mastering DMA engine (`Vid_In.vhd`) which assembles pixels into words and writes the raw video data into memory.

The `track::` class monitors the video format tracking hardware to detect the acquisition or loss of signal lock. When either event happens, appropriate details are communicated to the `video_capture::` class to start or stop video streaming.

The `video_capture::` class manages the recording of video data into the reserved memory buffer as well as hardware settings controlling things like the data format and preview decimation ratio. Since there are two input streams, the `video_capture::` class operates on frame pairs. Each pair always contains a full resolution frame and may or may not contain a preview resolution frame. The maximum preview framerate is 30 fps, so if the full framerate is greater than 30 fps, some full frames will not have a corresponding preview frame. For simplicity, this is represented as a frame pair with an invalid preview pointer (`NULL`).

The `video_capture::capture_frames` thread loops through the following process to capture frame pairs:

* Create a new frame pair
  * Allocate memory from the reserved buffer
  * Fill in frame details
* Queue the frame (send details to the hardware)
* Post the frame (tell hardware the new frame data is valid)
* Wait for an interrupt indicating the frame has been captured
* Send the frame to the compression logic

Once a frame pair is captured, it is placed on a work queue for the video compression engine. If this queue is too full, the oldest pending frame pair is dropped and a warning is printed.

NOTE: The video input logic assumes a "clean" video signal and is an example design intended to be simple and easy to understand. This logic does not attempt to deal with real world problems like random noise on an unterminated SDI input or when users plug/unplug the cable while the system is running. An actual production device should be more tolerant of signal errors.

#### Video Pattern

| File(s)                              | Description             |
| ------------------------------------ | ----------------------- |
| `src/cpp/ndi_encode/video_pattern.*` | `video_pattern::` class |

The `video_pattern::` class is a sub-class of the `video_capture::` class that does not rely on hardware to generate a video stream. Rather than capturing frames from hardware, this class generates a video pattern any time the signal format changes (it is possible to manually change the resolution at run-time from the console). The video pattern generated is twice as tall as the video format (see the m\_scroll\_dist member variable), and a preview resolution version of the pattern is also created (since both full and preview resolution streams are required). In normal operation, a frame-sized window of the over-sized video pattern is used to provide source video for the `video_compress::` class, with the start point of the window moving down one line with each new frame, providing the illusion of moving video.

#### Video Compression

| File(s)                               | Description                               |
| ------------------------------------- | ----------------------------------------- |
| `src/cpp/ndi_encode/video_compress.*` | `video_compress::` class                  |
| `src/fpga/NDI_Enc/Encode_x4.vhd`      | 4-core NDI Hardware Encoder               |
| `src/fpga/NDI_Enc/Encode_Xilinx.vhdp` | Single NDI Hardware Encoder core (Xilinx) |
| `src/fpga/NDI_Enc/Encode_Altera.vhd`  | Single NDI Hardware Encoder core (Altera) |

One, two, or four copies of the single NDI Encoder core are instantiated in hardware (`Encode_x4.vhd`). The cores operate in parallel, with each core operating on one or more quarters of the video frame known as a "slice". This increases the effective throughput of the compression core and lowers system latency.

The `video_compress::` class monitors a work queue filled with raw video frames by the `video_capture::` class (above). When a new frame pair is received the full resolution frame is sent to hardware for processing. Once the hardware generates an interrupt indicating processing is finished, the software performs minimal post-processing on the compression thread. Slice lengths are read from the hardware and written to memory by `fixup_frame()`, the resulting frame length is used to update the quality setting, then the frame is passed to the NDI send threads via `add_frame_ndi()`.

The send threads (`send_full`, `send_prvw`) monitor independent work queues for the full and preview resolution video streams and pass new frames to the NDI stack. Sending a frame to the NDI stack is a synchronizing event which will block until the frame is fully processed, which is why there are independent threads for the two video streams. This gives each stream the maximum amount of time to send data to the listeners.

#### NDI Transmission

| File(s)                                     | Description                   |
| ------------------------------------------- | ----------------------------- |
| `src/cpp/ndi_encode/video_compress.*`       | `video_compress::` class      |
| `src/cpp/ndi_encode/network_send_video.cpp` | network\_send:: video support |

The `send_full()` and `send_prvw()` threads in the `video_compress::` class monitor independent work queues for the full and preview resolution video streams and pass new frames to `network_send::add_frame()`. This function is basically a shim which converts the `video_compress::frame_t` type into an `NDIlib_video_frame_v2_t` type and sends it to the NDI stack. Sending a frame to the NDI stack is a synchronizing event which will block until the frame is fully processed, which is why there are independent threads for the two video streams. This gives each stream the maximum amount of time to send data to the listeners.

NOTE: The various frame types exist primarily because much of the example code predates the availability of the official NDI Advanced SDK. Expect future releases of this code to migrate to using native NDI frame types.

#### Audio Input

| File(s)                              | Description             |
| ------------------------------------ | ----------------------- |
| `src/cpp/ndi_encode/audio_capture.*` | `audio_capture::` class |
| `src/fpga/ip/common/Aud_In.vhd`      | Audio input DMA logic   |

Audio input starts with the low level audio input logic. This logic supports standard iis serial audio streams. Support for parallel audio data extracted from the HDMI streams will be supported soon. Once assembled into complete audio samples, the data is queued in a FIFO allowing burst writes to system memory.

The `audio_capture::` class manages the recording of audio data into the reserved memory buffer as well as hardware settings controlling things like the data format (typically left-justified).

The `audio_capture::capture_frames` thread loops through a process very similar to the video\_capture thread. One major difference is the audio logic currently supports "overlapped" commands, meaning the next command is written to hardware while the current command is still in progress:

* Create a new frame
  * Allocate memory from the reserved buffer
  * Fill in frame details
* Queue the frame (send details to the hardware)
* Post the frame (tell hardware the new frame data is valid)
* Loop until signaled to exit
  * Create a new frame
    * Allocate memory from the reserved buffer
    * Fill in frame details
  * Queue the frame (send details to the hardware)
  * Post the frame (tell hardware the new frame data is valid)
  * Wait for an interrupt indicating the frame has been captured
  * Send the frame to the compression logic

#### Audio Compression

| File(s)                               | Description              |
| ------------------------------------- | ------------------------ |
| `src/cpp/ndi_encode/audio_compress.*` | `audio_compress::` class |

This class simply passes captured audio frames from the `audio_capture::` class to the NDI stack. It exists primarily to match the video path data flow, to allow any processing of the audio samples that might be required (eg: masking lower-order bits that might contain AES packet data), and because when this code was initially written the NDI API did not support 32-bit signed audio samples.

In future versions, expect the native NDI types to be used and the audio\_compress class to be removed.

#### Audio Transmission

| File(s)                                     | Description                    |
| ------------------------------------------- | ------------------------------ |
| `src/cpp/ndi_encode/network_send_audio.cpp` | `network_send::` audio support |

This function is basically a shim which converts the `audio_capture::frame_t` type into an NDIlib\_audio\_frame\_interleaved\_32s\_t type and sends it to the NDI stack.

Expect this code to be deprecated in future versions and the sending logic to be migrated to the `audio_capture::` class.

#### Tally Operation

| File(s)                      | Description     |
| ---------------------------- | --------------- |
| `src/cpp/ndi_encode/tally.*` | `tally::` class |

Tally outputs are implemented using the Linux kernel LED class. Tally LED entries are created in the device-tree, and the application uses the resulting sysfs entries to control the LED behavior.

The tally class initializes all LEDs to a known state on construction, then the tally process simply loops, checking for updates from the NDI stack. If new tally data is available, the tally LEDs are updated

### NDI Decode Operation

#### Video Output

| File(s)                                      | Description                      |
| -------------------------------------------- | -------------------------------- |
| `src/cpp/ndi_decode/video_playback.*`        | `video_playback::` class         |
| `src/vpga/ip/common/Test_Pattern_Gen_YUV444` | Video timing & pattern generator |
| `src/fpga/ip/common/Vid_Out.vhd`             | Video output DMA logic           |
| Filename varies                              | Low level video outputlogic      |

Video output timing is driven by the low level video logic. This logic is platform specific:

| Platform   | Dir | Implementation                                       |
| ---------- | --- | ---------------------------------------------------- |
| a10socdk   | Tx  | Altera HDMI RxTx example design modified for Tx only |
| ZCU104     | Tx  | Xilinx HDMI Tx example design                        |
| Zybo-Z7-20 | Tx  | Open-source DVI Tx logic                             |

The software for controlling the low-level video input logic is currently outside the scope of this example. The Zybo-Z7 HDMI output requires no control software, the Xilinx HDMI control software is running bare-metal on one of the Cortex-R5 cores, and the Altera HDMI control software runs on a dedicated NIOS core (see the README file accompanying the hardware project files for details).

The video output logic generates a pixel stream with programmable timings based on the video clock which is programmable on most platforms to support multiple resolutions. The video pixel data can be a hardware generated test pattern or the video data can be filled in with data from memory for video playback. The vertical sync interrupt from `Vid_Out.vhd` is used to drive all output operations, and an NDI FrameSync instance is used to synchronize the received NDI video stream with the hardware output timing.

**Video output process flow:**

**video\_playback::pull\_frames thread**

* Wait for vertical sync interrupt: `video_playback::pull_frames`
* Pull a frame from the NDI FrameSync Instance: `network_recv::get_video()`
* Copy video frame to reserved memory region visible by hardware: `video_decode::add_frame()`
* Push copied frame onto NDI Decode queue: `video_decode::add_frame()`
* Free video frame acquired from the NDI FrameSync Instance

**video\_decode::decode\_frames thread**

* Wait for new frame to decode
* Setup hardware to decode NDI frame into raw video data: `video_decode::decode_frame()`
* Start hardware decoding: `video_decode::post_frame()`
* Wait for interrupt
* Queue the raw frame for display: `video_playback::send_frame()`
* The frame will display after the next vertical sync

#### Video Decompression

| File(s)                               | Description                               |
| ------------------------------------- | ----------------------------------------- |
| `src/cpp/ndi_decode/video_decode.*`   | `video_decode::` class                    |
| `src/fpga/NDI_Dec/Decode_x4.vhd`      | 4-core NDI Hardware Encoder               |
| `src/fpga/NDI_Dec/Decode_Xilinx.vhdp` | Single NDI Hardware Encoder core (Xilinx) |
| `src/fpga/NDI_Dec/Decode_Altera.vhd`  | Single NDI Hardware Encoder core (Altera) |

One, two, or four copies of the NDI Decoder core are instantiated in hardware (`Decode_x4.vhd`). The cores operate in parallel, with each core operating on one or more quarters of the video frame known as a "slice". This increases the effective throughput of the compression core and lowers system latency.

The `video_decode::` class monitors a work queue filled with NDI video frames by the `video_playback::` class (above). When a new frame is received it is sent to hardware for processing. Once the hardware generates an interrupt indicating processing is finished, the software queus the decoded frame for display via

NDI send threads via `video_playback::send_frame()`.

#### Audio Output

| File(s)                               | Description              |
| ------------------------------------- | ------------------------ |
| `src/cpp/ndi_decode/audio_playback.*` | `audio_playback::` class |
| `src/fpga/ip/common/Aud_Out.vhd`      | Audio output DMA logic   |

Audio output starts with the low level output input logic. This logic supports standard iis serial audio streams. Support for parallel audio data extracted from the HDMI streams will be supported soon. Once assembled into complete audio samples, the data is queued in a FIFO allowing burst writes to system memory.

The `audio_playback::` class manages the playback of audio data from the reserved memory buffer as well as hardware settings controlling things like the data format (typically left-justified).

The `audio_playback::pull_frames` thread loops through a process very similar to the video\_playback thread. An interrupt is generated when a programmed audio DMA completes which drives the timing of the rest of the audio output operations.

**Audio output process flow**

**audio\_playback::pull\_frames thread**

* Wait for audio interrupt: `audio_playback::pull_frames`
* Pull a frame from the NDI FrameSync Instance: `network_recv::get_audio()`
* Convert audio frame to 32-bit data and copy to reserved memory region visible by hardware: `audio_playback::queue_frame()`
* Free video frame acquired from the NDI FrameSync Instance
* Enable playabck of audio data: `audio_playback::post_frame()`

## Rebuilding from source

| Directory       | Description                           |
| --------------- | ------------------------------------- |
| `src/fpga/`     | FPGA Project Files                    |
| `linux_kernel/` | Linux kernel, U-Boot, and device-tree |
| `src/cpp/`      | C++ Application Project Files         |
| `os_uSD/`       | Debian rootfs and uSD images          |

### FPGA hardware design

The `src/fpga/` directory contains projects to build a bit-file for each supported platform. Refer to the README file in the `src/fpga` directory for full details on building the hardware projects.

### Boot loader, Linux kernel, and device-tree

The `linux_kernel/` directory contains projects to build a boot-loader, kernel, and device-tree for the supported platforms. The device-tree customizations required for the NDI FPGA hardware are also included in the example projects. Refer to the README file in the `linux_kernel` directory for full details.

#### Petalinux root file system

**NOTE:** For Xilinx platforms, the scripts in `linux_kernel` create a Petalinux root file-system in addition to the kernel and boot loader. This is a very minimal embedded system and it is difficult to customize since the entire OS is cross-compiled. This example ignores the Petalinux generated rootfs and instead uses a standard Debian OS install which allows for self-hosted compiles and easy modification of the OS using standard package management tools.

For details on building the Debian root file-system, see the section on creating a bootable image (below).

### Application software

The `src/cpp/` directory contains example encode and decode software applications which use the FPGA based NDI logic to (de)compress live video data, and the Advanced NDI SDK to send and receive that data as an NDI stream. Refer to the README file in the `src/cpp` directory for details.

### Bootable image including all of the above

The `os_uSD/` directory contains scripts to create a Debian root filesystem as well as a bootable uSD card image. The resulting images contain all prerequisites needed to perform a self-hosted build of the example NDI application (ie: built on the ARM platform and not cross-compiled). Refer to the README file in the `os_uSD` directory for full details.

## Software Applications

## ndi\_encode

This is an example application illustrating the use of the FPGA based NDI encoder with the Embedded NDI SDK. For full details and a theory of operation covering the entire system (software, hardware, and OS), refer to the top-level README file.

### Usage

The ndi\_encode program is simply launched from the command line, eg:

```
ndi_encode
```

Note that this will launch the version installed in /usr/local/bin/. Alternatively, the program can be run from the build directory. Note that root permissions are required so it is necessary to use sudo:

```
sudo /path/to/workdir/ndi_encode
```

#### Run Time Commands

Once running, the program will listen for commands on stdin. These commands are intended to modify the running behavior of the program and illustrate how to integrate with a separate utility monitoring the physical video input source (eg: an HDMI or SDI input) as a signal is acquired, locked, or lost. The commands which change the video resolution and frame rate should only be used when running in pattern generator mode.

*   start | stop

    Start or stop streaming video and audio. This would correspond with the acquisition (start) or loss (stop) of a stable signal.
*   720 | 1080 | 4k

    Change the video resolution
*   24 | 25 | 30 | 50 | 60

    Set the primary video frame rate
*   1000 | 1001

    Set the secondary frame rate: 1000=exact (eg: 60.00), 1001=NTSC (eg: 59.94)
*   exit

    Quit the program

#### Command Line Options

Various command line switches are available to control the program's behavior:

**General Options**

* Set default video resolution
  * \-1 1080p60 (default)
  * \-7 720p60
  * \-4 4Kp60
* Video options
  *   \-P Pattern generator

      Use the built-in pattern generator instead of live video input.
  *   \-v Increase verbosity

      Output more detailed information. This switch may be provided multiple times to further increase the debugging output. The available levels are:

      * Fatal
      * Error (default)
      * Warning
      * Information
      * Debugging
  *   \-q Decrease verbosity (quiet)

      Decrease the amount of output generated. This switch may be provided multiple times. See the discussion of -v (above) for details.
* Audio Options
  *   \-h HDMI input

      Use embedded audio from the HDMI stream.
  *   \-l Line input

      Use analog audio input.
  *   \-p Pattern generator

      The audio input hardware includes a simple saw-tooth pattern generator that can be used as an input source for testing.

**Debugging and Advanced Options**

*   \-X Disable video capture

    This disables the video capture thread. No video frames will be captured, compressed, or sent to the NDI stack. Only audio will be processed.
*   \-x Disable audio capture

    This disables the video capture thread. No audio samples will be captured, compressed, or sent to the NDI stack. Only video will be processed.
*   \-F Full resolution capture only

    This option disables the preview stream and only sends full resolution frames to the NDI stack.
*   \-f fflush() after each log message

    This is useful if the application is crashing to insure log error messages actually reach the terminal.
* \-A Use ACP
*   \-B Bypass ACP

    These options do nothing on the current Zynq development boards and only apply to Cyclone-V based platforms.

## ndi\_decode

This is an example application illustrating the use of the FPGA based NDI decoder with the Embedded NDI SDK. For full details and a theory of operation covering the entire system (software, hardware, and OS), refer to the top-level README file.

### Usage

The ndi\_decode program is simply launched from the command line, eg:

```
# Launch and display the first NDI source found
ndi_decode

# Launch and connect to a specific NDI source
ndi_decode -s "Machine name (NDI channel)"
```

Note that this will launch the version installed in /usr/local/bin/. Alternatively, the program can be run from the build directory. Note that root permissions are required so it is necessary to use sudo:

```
sudo /path/to/workdir/ndi_receive
```

#### Command Line Options

Various command line switches are available to control the program's behavior:

**General Options**

*   \-s `<NDI Source>`

    Specify NDI source to stream. If this setting is not provided, ndi\_receive will attempt to connect to the first NDI source it finds on the network.
* Set output video resolution
  * \-1 1080p (default)
  * \-7 720p
  * \-4 4Kp
* Set output video framerate
  * \-6 60 Hz (default)
  * \-5 50 Hz
* Select output video
  * \-k Output key (alpha) data if available instead of image data
* Select SDRAM burst mode
  * \-b Disable SDRAM burst mode (e.g., 1 macroblock to SDRAM per burst transfer). Default behavior is to burst 4 macroblocks to SDRAM per burst transfer.
*   \-o `<output format>`

    Specifies the output video format, where `output format` is one of the enumerated `video_format_t` values in `hardware.h` in decimal. The default is packed 4:2:2 (e.g., UYVY).

**Debugging and Advanced Options**

*   \-v Increase verbosity

    Output more detailed information. This switch may be provided multiple times to further increase the debugging output. The available levels are:

    * Fatal
    * Error (default)
    * Warning
    * Information
    * Debugging
*   \-q Decrease verbosity (quiet)

    Decrease the amount of output generated. This switch may be provided multiple times. See the discussion of -v (above) for details.
*   \-f fflush() after each log message

    This is useful if the application is crashing to insure log error messages actually reach the terminal.
* \-A Use ACP
*   \-B Bypass ACP

    These options do nothing on the current Zynq development boards and only apply to Cyclone-V based platforms.
* \-c `<capture file>`

Allows for individual frames to be captured to the filesystem. This switch is intended for debugging purposes and assumes that the stride is equal to the length of a row of image data. The default is to capture frame number 25, but this can be modified using the `-n` argument to select a different value. No more than one frame can be captured at a time.

* \-n `<frame capture number>`

If capturing video frames, indicates the frame number to capture (defaults to 25). Requires the use of the `-c` option to capture frames.

## NDI Utilities

This directory contains utilities helpful for debugging or initial testing of the FPGA hardware. For full details and a theory of operation covering the entire system (software, hardware, and OS), refer to the top-level README file.

### ndi\_reg Utility

This utility allows reading and writing of the NDI hardware registers. To read an NDI register, provide the register address on the command line:

```
# Read the Video Input register
ndi_reg 0x200
```

To write an NDI register, provide the register address and write data on the command line:

```
# Write the Video Input register
ndi_reg 0x200 0x00c00100
```

For details on register settings, refer to the corresponding software and hardware files (detailed in the top-level README file). Refer to the following table for the base addresses of the various hardware functions:

| Hardware           | Base Address |
| ------------------ | ------------ |
| NDI Encoder Core 0 | 0x000        |
| NDI Encoder Core 1 | 0x040        |
| NDI Encoder Core 2 | 0x080        |
| NDI Encoder Core 3 | 0x0c0        |
| NDI Decoder Core 0 | 0x100        |
| NDI Decoder Core 1 | 0x140        |
| NDI Decoder Core 2 | 0x180        |
| NDI Decoder Core 3 | 0x1c0        |
| Video Input        | 0x200        |
| Preview Input      | 0x210        |
| Audio Input        | 0x220        |
| Video Tracking     | 0x230        |
| Video Output       | 0x280        |
| Audio Output       | 0x290        |
| Version (Date)     | 0x3fe        |
| Version (Revision) | 0x3ff        |

## Changelog

Versions: Major.Minor.Patch

Changelogs are synchronized for Major.Minor version numbers, but each project has it's own patch version (eg: src/fpga might be version 1.1.2, and src/cpp could be 1.1.4).

### \[6.1.0-rc1] - 2024-09-13

#### Added

* Synchronize major version number with software SDK version
* Encoder support for planar alpha
* Support for new packed and semi-planar video formats
* Support for 16-bit video (11-bits passed through SpeedHQ codec)

### \[1.5.4] - 2024-01-24

#### Added

* Example project for the Kria KV260 development board

#### Fixed

* A couple bugs in the clear-text FPGA logic

### \[1.5.3] - 2023-08-25

#### Changed

* Xilinx projects updated to Vivado 2022.1
* Zynq 7000 HDMI Rx Logic improved
* SoCKit projects deprecated as boards are no longer available for purchase

### \[1.5.0] - 2022-10-09

#### Added

* Alpha support (interleaved and planar) added to NDI\_Dec
* Example projects targeting the Arria-10 SoC Development Kit

#### Changed

* Updates for NDI v5.x

### \[1.4.0] - 2020-03-25

#### Added

* NDI Decode support

#### Changed

* Updates for NDI v4.5
* Xilinx & Petalinux projects updated to 2019.2

### \[1.3.0] - 2019-07-08

#### Changed

* Updates for NDI v4.0
* Refactoring to support both encode and decode

#### Changed

* Updates to device-tree layout and uio device names
* Zybo uSD image modified to include all three examples (Encode, Encode-Lite, Decode)

### \[1.2.1] - 2018-11-17

#### Added

* Zybo-Z7-20-Lite example design (16-bit SDRAM interface with 2 encoder cores)

### \[1.2.0] - 2018-10-03

#### Added

* Auto-detect video format
* Audio is now working

### \[1.1.1] - 2018-09-25

#### Changed

* Sub-project zip files now provided in expanded form
* Restructured project directories

### \[1.1.0] - 2018-08-31

#### Added

* Support new hardware version register

#### Changed

* Updated NDI\_Demo hardware project files
* Updated PetaLinux files

### \[1.0.0] - 2018-07-13

* Initial version

### Changelog Hints & Details

*
