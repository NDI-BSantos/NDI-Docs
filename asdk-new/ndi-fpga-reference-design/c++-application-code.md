# C++ Application Code

This is a set of example applications illustrating the use of the FPGA based NDI encoder with the Advanced NDI SDK. For full details and a theory of operation covering the entire system (software, hardware, and OS), refer to the top-level README file.

Source code for the examples is organized into the following subdirectories:

| Directory    | Contents                                                   |
| ------------ | ---------------------------------------------------------- |
| ndi\_common/ | Files common to all NDI applications                       |
| ndi\_decode/ | Example NDI Decoder application (eg: TV or Monitor)        |
| ndi\_encode/ | Example NDI Encoder application (eg: Camera or NDI Source) |
| ndi\_util/   | Debug and utility applications                             |

## Getting Started

### Prerequisites

The following prerequisites must be installed before you can build the example applications.

#### libuio

* https://github.com/Linutronix/libuio

libuio is used to access the FPGA hardware via entries in the device-tree

The available uSD card images have libuio packages compiled from source (see `~/libuio`) and the debian packages installed. For other platforms, libuio will need to be compiled and installed manually.

#### NDI Advanced SDK Library

The NDI Advanced SDK allows the sending and receiving of compressed frames, which is required to use the FPGA encoder. To install the library, simply copy the files to an appropriate location for your system. The example uSD card images have the NDI library files already installed in `/usr/local/lib` and `/usr/local/include`.

### Building and Installing

Once the prerequisites have been installed, building the applications is straight-forward:

```
cd <NDI_SDK>/fpga_reference_design/src/cpp
make
sudo make install
```

The applications will be installed with setuid privileges to the `/usr/local/bin/` directory.

## Usage

See the `README.md` files in each subdirectory for usage details specific to each available application.

## Changelog

### \[6.1.0-rc1] - 2024-09-13

#### Changed

* Reset version numbering to align with software SDK

#### Added

* Encoder support for planar alpha
* Support for new packed and semi-planar video formats
* Support for 16-bit video (11-bits passed through SpeedHQ codec)
* Additional decoder application features:
  * Burst vs. no-burst mode SDRAM transfers
  * New video capture class and command line options for capturing video frames
  * Output video format can be selected at run-time (default to 8-bit UYVY)

### \[1.5.1] - 2023-01-13

#### Added

* Support for Arria-10 SoC Developemnt Kit platform

#### Changed

* Disable CPU intensive protocols on lower-end platforms

### \[1.5.0] - 2022-08-05

#### Changed

* Updates for NDI 5.x
  * Remove frame copy functions in video\_compress.cpp
  * Pass scatter gather list and let NDI SDK perform the copy
  * Update examples with new NDI Vendor ID format

### Fixed

* Changed pattern generator line stride to avoid issues in 4:2:0 mode

### \[1.4.2] - 2022-07-12

#### Fixed

* SSM2603 Line Input was not working

### \[1.4.1] - 2020-03-27

#### Changed

* Switch ZCU104 to I2S serial audio output format

### \[1.4.0] - 2020-03-25

#### Changed

* Updates for NDI v4.5
* NDI Decode support

### \[1.3.3] - 2020-02-05

#### Changed

* Switched to NDI SDK function to generate next Q value for video encoding
* Updated single-character debug output so each character is unique

#### Fixed

* Addressed the various `FIXME` comments in the ndi\_encode application code

### \[1.3.2] - 2020-01-07

#### Added

* Support for SoCKit platform

### \[1.3.1] - 2019-09-26

#### Changed

* Code migrated to separate sub-directories for each target application
* Makefile updates to better manage auto dependency generation

### \[1.3.0] - 2019-07-08

#### Changed

* Updates for NDI v4.0
* Refactoring to support both encode and decode

### \[1.2.2] - 2018-11-17

#### Added

* Support for changing reserved memory region size in device-tree
* Support for fewer than 4 encoder cores

### Fixed

* Unused variable warning when performing a release build

### \[1.2.1] - 2018-10-10

#### Added

* Support for changing frame rate in pattern generator mode

#### Changed

* Zybo audio codec setup migrated to ndi\_send application

#### Fixed

* Pattern Generator is working again
* Pattern Generator 4K video mode is working
* Video tracking thread was consuming excessive CPU cycles

### \[1.2.0] - 2018-10-03

#### Added

* Auto-detect video format
* Audio is now working

### \[1.1.1] - 2018-08-25

#### Changed

* Sub-project zip files now provided in expanded form
* Restructured project directories
* ndi\_device renamed ndi\_send
* Generate proper timestamps

#### Fixed

* Issue with potential use after free related to addition of copy thread

### \[1.1.0] - 2018-08-31

#### Added

* ndi\_reg utility
* Support new hardware version register

#### Changed

* General cleanup and removal of deprecated/commented code
* Pull details on raw video memory from device tree if available
* Add copy thread to video path

### \[1.0.0] - 2018-07-13

* Initial version
