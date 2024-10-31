---
description: >-
  This page details how to obtain and use prebuilt Micro SD card images with
  supported FPGA development boards to easily create a working NDI encoder or
  decoder.
---

# Prebuilt uSD Images

## Login Details

```txt
Default username: debian
Default password: temppwd

sudo is enabled without password for root access
root login is disabled
```

## Obtaining the uSD Image Files

The uSD images may be downloaded via the following URLs:

| Manufacturer | Board      | uSD Image Download URL      |
| ------------ | ---------- | --------------------------- |
| Altera       | A10-SoCDK  | http://ndi.link/NDIUSDA10   |
| Digilent     | Zybo-Z7-20 | http://ndi.link/NDIUSDZYBO  |
| Digilent     | Arty-Z7-20 | http://ndi.link/NDIUSDZYBO  |
| Xilinx       | ZCU104     | http://ndi.link/NDIUSDZCU   |
| Xilinx       | Kria KV260 | http://ndi.link/NDIUSDKV260 |

**NOTE:** The same uSD image is used for the Zybo-Z7-20 board, and the Arty-Z7-20. The uSD image contains boot files for both systems. To use the image with the Arty-Z7-20, you must update the boot files on the FAT partition prior to booting on the Arty development board.

Each uSD image is a zip archive of three files:

| Extension | Description                                                  |
| --------- | ------------------------------------------------------------ |
| .img.xz   | The image file (xz compressed)                               |
| .bmap     | A block map file for use with bmaptools (highly recommended) |
| .md5      | Checksum of the above two files                              |

Most users will likely only need to use the img.xz file.

## Writing the Image to a uSD Card

Use of bmaptool at a Linux command line is the most efficient way to write the image file to a uSD card. If this is not an option or is considered too complex, any uSD imaging tool which supports xz compression can be used. [Etcher](https://etcher.io/) is a good choice if you do not already have a preference.

## Using the Image

Insert the programmed uSD card into the development board and boot as usual. It is recommended you connect to the serial terminal (via USB). All images use the following console settings:

```sh
115200 baud, 8 data bits, 1 stop bit, no parity
```

The system will boot and display the IP address, as well as the default username and password at the serial console login prompt. You may login either via the serial console or remotely via ssh.

### First Boot

The initial boot will take somewhat longer than normal as a new set of ssh keys are generated and the root partition is expanded to fill the uSD card.

## Running the Example NDI Encode Application

An example NDI source application is provided. This application is launched automatically at boot and may also be run from the command line. The utility is named ndi\_encode and is capable of using a generated video pattern or live HDMI input as the video source. The application includes various command-line options to control the initial video mode as well as a command interpreter that can be used to change the operating mode at run-time. To launch the demo from the command line, simply run one of the following commands:

```sh
# Live HDMI input:
ndi_encode

# Pattern generator:
ndi_encode -P
```

Once running, the operating mode can be changed by typing commands at the console. Type "help" to get a list of commands.

If running with HDMI input on the ZCU104, the second virtual serial port is used as a console for the Cortex-R5 running the HDMI monitoring software. Connection details are the same as the Linux console:

```sh
115200 baud, 8 data bits, 1 stop bit, no parity
```

The R5 will report changes in the incoming HDMI stream on this port.

### Known Issues and Limitations

While the NDI Encoder core is fully functional the example software and demo platforms currently have some limitations:

* This version of the NDI Advanced SDK is designed for development use and will run on a stream for 30 minutes. For a commercial use license, please email support@ndi.video
* Due to the asynchronous nature of the startup scripts, the system clock may be adjusted after the ndi\_encode example is launched. If this happens, it may trigger the 30 minute timeout even though the application may not have been running for 30 minutes of "wall clock" time. Kill and re-start the demo application to resume NDI streaming.
* The HDMI input hardware for the ZCU104 uses an evaluation license, so HDMI video will stop being received and the image will turn "green" after apx. 1 hour of operation. Reprogram the FPGA (eg: reboot) to restore normal operation.
* While HDMI input on the ZCU104 is functional, the HDMI management software (running on one of the R5 cores) does not communicate with the ndi\_encode application. The video format tracking hardware is used to detect video format changes, however more details regarding the HDMI signal are available via the HDMI management software.

## Running the Example NDI Decode Application

An example NDI receive application is provided. This application is launched manually by executing `ndi_decode`. When launched with no options the utility will attempt to locate NDI sources on the local network and will automatically connect to the first source found. A specific NDI source may be specified using the -s parameter:

```sh
ndi_decode -s "NDI Device (Chan 1)"
```

NOTE: The default boot files on the uSD image must be switched to the NDI Decode version prior to running the ndi\_decode application. See the "Available Examples" section for your specific board for details.

### Known issues and limitations

* "Scaling" is performed by manipulating line stride when the NDI stream and video output resolutions do not match
* Format conversion is not performed between 4:2:2 and 4:2:0 video sources. If the output video format does not match the NDI stream format, a warning is displayed on the Linux console.

## Board Details

Connections and settings required to boot using the uSD images. Refer to the vendor documentation for full details.

### Altera Arria-10 SoC Development Kit

#### Available Examples

Two FPGA examples are available for the ZCU104 board:

* Decode example with 4 NDI Decode cores
* Encode example with 4 NDI Encode cores

The default uSD image uses the Enc files, however files for all examples are included on the uSD card. To change the example design, simply update the FPGA programming files in the `/boot` directory and **cycle power**:

```sh
# Use the Decode example
sudo cp /boot/fit_spl_fpga_dec.itb /boot/fit_spl_fpga.itb

# Use the Encode example
sudo cp /boot/fit_spl_fpga_enc.itb /boot/fit_spl_fpga.itb

# You MUST cycle power to reprogram the FPGA!!!
```

#### Jumpers and Switches

All jumpers and switches are set per the defaults listed in section 3 of the Arria 10 SoC Development Kit User Guide.

| Ref   | Function         | Setting            |
| ----- | ---------------- | ------------------ |
| SW1   | Boot Mode        | All off            |
| SW2   | User Switches    | All off            |
| SW3.1 | A10 JTAG         | Off                |
| SW3.2 | Max V JTAG       | Off                |
| SW3.3 | FMCA JTAG        | On                 |
| SW3.4 | FMCB JTAG        | On                 |
| SW3.5 | PCIe JTAG        | On                 |
| SW3.6 | MSTR0 JTAG       | Off                |
| SW3.7 | MSTR1 JTAG       | Off                |
| SW3.8 | MSTR2 JTAG       | Off                |
| SW4   | MSEL             | All off            |
| J16   | OSC2 Clk Sel     | Short              |
| J17   | OSC2 Clk Sel     | Short              |
| J30   | HPS Core Voltage | Short              |
| J32   | FMCBVADJ         | Short 9 and 10     |
| J42   | FMCAVADJ         | Short 9 and 10     |
| J3    | BSEL0            | Short left 2 pins  |
| J4    | BSEL1            | Short upper 2 pins |
| J5    | BSEL2            | Short upper 2 pins |

#### Connections

| Ref | Function / Label | Connect to                           |
| --- | ---------------- | ------------------------------------ |
| J19 | FMC Port B       | Bitec FMC HDMI Daughter Card Rev. 11 |
| J5  | HPS Ethernet     | Local Ethernet network               |
| J10 | PROG/UART        | Host system USB port                 |
| J36 | Power            | Power Supply Module                  |
| RX  | Bitec HDMI Rx    | HDMI video source (NDI Encode)       |
| TX  | Bitec HDMI Tx    | HDMI video output (NDI Decode)       |

#### LEDs

| LED        | Function                |
| ---------- | ----------------------- |
| FPGA\_LED0 | Tally (Main or Preview) |
| FPGA\_LED1 | Tally (Main)            |
| FPGA\_LED2 | Tally (Preview)         |
| FPGA\_LED3 | CPU load                |

### Digilent Zybo-Z7-20

#### Available Examples

Three FPGA examples are available for the Zybo-Z7 board:

* Decode.Xil: 4-core NDI Decoder with 1 GB of 32-bit SDRAM
* Encode.Xil: 4-core NDI Encoder with 1 GB of 32-bit SDRAM
* Enc-Lite.Xil: 2-core NDI Encoder with 512 MB of 16-bit SDRAM (1 SDRAM chip is unused)

The default uSD image uses the 4-core Encoder files, however files for all three examples are included on the uSD card. To change the example design, simply update the files in the `/boot` directory and reboot:

```sh
# Use the Decode example with 4 cores and 1 GB of 32-bit SDRAM
sudo cp /boot/Decode.Xil/* /boot/

# Use the Encode example with 4 cores and 1 GB of 32-bit SDRAM
sudo cp /boot/Encode.Xil/* /boot/

# Use the "Lite" Encode example with 2 cores and 512 MB of 16-bit SDRAM
sudo cp /boot/Enc-Lite.Xil/* /boot/
```

The "Enc-Lite" Encode example is intended to allow performance evaluation of a typical "minimal" Zynq platform that uses only a single SDRAM memory chip with a 16-bit memory bus. This design is still capable of processing 1080p60 video.

The "Full" Encode example will have lower latency at all resolutions and can in theory compress up to 4Kp60 4:2:0 video (with enough available SDRAM bandwidth). A source of 4K video would also be required (eg: Analog Devices ADV7619, ITE IT68059) since the HDMI I/O on the Zybo-Z7 does not support resolutions beyond 1080p60.

#### Jumpers and Switches

| Ref | Function  | Setting |
| --- | --------- | ------- |
| JP5 | Boot Mode | SD      |
| JP6 | Power     | WALL    |

#### Connections

| Ref | Function / Label | Connect to                     |
| --- | ---------------- | ------------------------------ |
| J9  | HDMI Rx          | HDMI video source (NDI Encode) |
| J8  | HDMI Tx          | HDMI video output (NDI Decode) |
| J7  | Line In          | Analog audio in                |
| J5  | Headphone        | Analog audio out               |
| J3  | Ethernet         | Local Ethernet network         |
| J17 | Power            | Appropriate 5V power supply    |
| J12 | PROG/UART        | Host system USB port           |

#### LEDs

| LED | Function                       |
| --- | ------------------------------ |
| LD0 | Tally (Main)                   |
| LD1 | Tally (Preview)                |
| LD2 | CPU load                       |
| LD3 | uSD activity                   |
| LD4 | Tally (either Main or Preview) |

### Digilent Arty-Z7-20

#### Available Examples

One FPGA example is available for the Arty-Z7 board:

* Arty-Enc.Xil: NDI Encoder based on Zybo "Lite" design

The default uSD image uses the Zybo boot files, however the Arty boot files are included. To modify the uSD image to boot on the Arty, simply update the files in the FAT partition before booting the Arty board:

```sh
# Mount the first partition on the uSD
sudo mount /dev/sdx1 /mnt

# Copy the Arty version of the "Lite" Encode example to the filesystem root
sudo cp /mnt/Arty-Enc.Xil/* /mnt/

# Unmount the partition
sudo umount /mnt

# Remove the uSD and use to boot the Arty-Z7
```

#### Jumpers and Switches

| Ref | Function  | Setting                               |
| --- | --------- | ------------------------------------- |
| JP4 | Boot Mode | SD                                    |
| JP5 | Power     | USB (when using USB power)            |
| JP5 | Power     | REG (when using J18 with 7-15V power) |

#### Connections

| Ref | Function / Label | Connect to                     |
| --- | ---------------- | ------------------------------ |
| J10 | HDMI Rx          | HDMI video source (NDI Encode) |
| J11 | HDMI Tx          | HDMI video output (NDI Decode) |
| J8  | Ethernet         | Local Ethernet network         |
| J14 | PROG/UART        | Host system USB port           |

#### LEDs

| LED | Function        |
| --- | --------------- |
| LD0 | Tally (Main)    |
| LD1 | Tally (Preview) |
| LD2 | CPU load        |
| LD3 | uSD activity    |

### Xilinx ZCU104

#### Available Examples

Two FPGA examples are available for the ZCU104 board:

* Dec: Decode example with 4 NDI Decode cores
* Enc: Encode example with 4 NDI Encode cores

The default uSD image uses the Enc files, however files for all examples are included on the uSD card. To change the example design, simply update the files in the `/boot` directory and reboot:

```sh
# Use the Decode example
sudo cp /boot/Dec/* /boot/

# Use the Encode example
sudo cp /boot/Enc/* /boot/
```

#### Jumpers and Switches

| Ref   | Function        | Setting        |
| ----- | --------------- | -------------- |
| J85   | POR\_OVERRIDE   | 2-3 (default)  |
| J12   | SYSMON I2C addr | 1-2 (default)  |
| J13   | SYSMON I2C addr | 1-2 (default)  |
| J20   | PS\_POR\_B      | 1-2 (default)  |
| J21   | PS\_SRST\_B     | 1-2 (default)  |
| J22   | Reset Seq       | open (default) |
| SW6 1 | PS\_MODE        | On             |
| SW6 2 | PS\_MODE        | Off            |
| SW6 3 | PS\_MODE        | Off            |
| SW6 4 | PS\_MODE        | Off            |

#### Connections

| Ref  | Function / Label | Connect to           |
| ---- | ---------------- | -------------------- |
| P7   | HDMI Rx (bottom) | HDMI video source    |
| J52  | Power            | Power supply         |
| P12  | Ethernet         | Ethernet network     |
| J164 | JTAG/UART        | Host system USB port |

#### LEDs

| LED  | Function                |
| ---- | ----------------------- |
| LED0 | Tally (Main or Preview) |
| LED1 | Tally (Main)            |
| LED2 | Tally (Preview)         |
| LED3 | CPU load                |

### Xilinx Kria KV260

#### Available Examples

One FPGA example is available for the Kria KV260:

* Enc: Encode example with 4 NDI Encode cores

#### Board setup

The NDI example design requires the same board configuration as the Xilinx "Smart Camera" example, including the FSBL programmed into the on-module flash memory and the AR1335 camera module as provided in the [Basic Accessory Pack](https://www.xilinx.com/products/som/kria/kv260-vision-starter-kit/basic-accessory-pack.html) connected to J7.

See the [Getting Started](https://www.xilinx.com/products/som/kria/kv260-vision-starter-kit/kv260-getting-started/getting-started.html) instructions from Xilinx for full details.

There are no jumpers or switches to set on the KV260, and there are no user LEDs available to indicate tally.
