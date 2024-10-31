# FPGA Projects

## Directory structure:

| Directory            | Contents                                                                                                                                                                                                  |
| -------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| NDI\_Dec/            | VHDL files for the NDI Decoder core. All files should be compiled into the NDI\_Dec VHDL library rather than the default work library                                                                     |
| NDI\_Enc/            | VHDL files for the NDI Encoder core. All files should be compiled into the NDI\_Enc VHDL library rather than the default work library                                                                     |
| ip/altera            | Altera specific implementations of memory and FIFO blocks                                                                                                                                                 |
| ip/common            | Source code common to all example designs                                                                                                                                                                 |
| ip/extern            | External (non-NDI) IP used in the example designs                                                                                                                                                         |
| ip/packages          | VHDL Packages defining various types used in the logic                                                                                                                                                    |
| ip/xilinx            | Xilinx specific implementations of memory and FIFO blocks                                                                                                                                                 |
| a10-socdk-enc/       | Quartus Pro 22.2 NDI Encode project directory targeting the [Arria-10 SoC DevKit](https://www.intel.com/content/www/us/en/products/details/fpga/development-kits/arria/10-sx.html)                        |
| a10-socdk-dec/       | Quartus Pro 22.2 NDI Decode project directory targeting the [Arria-10 SoC DevKit](https://www.intel.com/content/www/us/en/products/details/fpga/development-kits/arria/10-sx.html)                        |
| SoCKit-Dec/          | **Deprecated** Quartus 22.1 NDI Decode project directory targeting the [Terasic SoCKit](https://www.terasic.com.tw/cgi-bin/page/archive.pl?Language=English\&No=816)                                      |
| ZCU104-Enc/          | Vivado 2022.1 NDI Encode project directory targeting the [Xilinx ZCU104](https://www.xilinx.com/products/boards-and-kits/zcu104.html)                                                                     |
| ZCU104-Dec/          | Vivado 2022.1 NDI Decode project directory targeting the [Xilinx ZCU104](https://www.xilinx.com/products/boards-and-kits/zcu104.html)                                                                     |
| Zybo-Z7-20-Enc/      | Vivado 2022.1 NDI Encode project directory targeting the [Digilent Zybo-Z7-20](https://store.digilentinc.com/zybo-z7-zynq-7000-arm-fpga-soc-development-board/)                                           |
| Zybo-Z7-20-Dec/      | Vivado 2022.1 NDI Decode project directory targeting the [Digilent Zybo-Z7-20](https://store.digilentinc.com/zybo-z7-zynq-7000-arm-fpga-soc-development-board/)                                           |
| Zybo-Z7-20-Enc-Lite/ | "Lightweight" Vivado 2022.1 NDI Encode project directory targeting the [Digilent Zybo-Z7-20](https://store.digilentinc.com/zybo-z7-zynq-7000-arm-fpga-soc-development-board/) with 16-bit SDRAM interface |
| Arty-Z7-20-Enc/      | Vivado 2022.1 NDI Encode project directory targeting the [Digilent Arty-Z7-20](https://store.digilentinc.com/arty-z7-zynq-7000-soc-development-board/)                                                    |
| KV260-Enc/           | Vivado 2022.1 NDI Encode project directory targeting the [Xilinx Kria KV260](https://www.xilinx.com/products/som/kria/kv260-vision-starter-kit.html)                                                      |

## Build Dependencies

### Xilinx

All Xilinx projects require Vivado version 2022.1. Any edition of Vivado 2022.1 will work, including the "WebPACK" edition available via free download from Xilinx.

#### ZCU104

You must have a valid license for the Xilinx HDMI IP core used in the ZCU104-based example designs. You may use a full license or a free evaluation license, but the license must be installed **prior to launching Vivado** to build the project: https://www.xilinx.com/products/intellectual-property/hdmi.html

#### Zybo-Z7-20 / Zybo-Z7-20-Lite / Arty-Z7-20

Board files from Digilent must be installed **prior to launching Vivado** in order to build the Zybo example designs. The [board files](https://github.com/Digilent/vivado-boards/) may be obtained from github and [installation instructions](https://reference.digilentinc.com/reference/software/vivado/board-files) are on the Digilent website.

Digilent periodically updates their board files repository which can lead to errors when openijng projects caused by a discrepancy in the `BoardPart` property. For reference, the current version of Zybo and Arty example designs were built with commit hash `3f67d8f827bf83bb6a16b4879e4fdad68b8443e8`.

Due to changes in Xilinx recommendations, these projects show critical warnings when loaded in recent versions of Vivado. These warnings may be ignored, as explained by the Digilent [reference manual](https://digilent.com/reference/programmable-logic/zybo-z7/reference-manual#hardware\_errata).

#### Kria KV260

The Kria KV260 example design deviates significantly from others supplied with the NDI Advanced SDK. See the `README` file in the `KV260-Enc` directory for details on how to create designs targeting this platform.

### Intel/Altera

The contents of the file `NDI_Enc/Encode_Altera.lic` (for Encode) and `NDI_Dec/Decode_Altera.lic` (for Decode) must be appended to your Quartus license file. If you do not have a license file, you must create one.

Arria-10 projects require Quartus Pro 22.2.0.

Cyclone-V projects require Quartus 22.1. The examples were compiled using the free "Lite" version, however the "Standard" edition should work as well.

Follow the Altera instructions for installation, including the required manual installation of WSL if you are using Windows.

If you plan to rebuild the NIOS code which reconfigures the GXB transceivers and PLLs, you need to manually install Eclipse per the Altera Quartus installation instructions. These instructions can also typically be found in the file: `<Quartus installation directory>/nios2eds/bin/README`

## Rebuilding

### Xilinx

Once you have all required dependencies installed (see above), simply open the desired project and build normally. The contents for the block design elements (hard processor system, PLLs, etc) will be regenerated on the first build.

### Altera

Before you can build the Altera projects, you must generate the hps platform logic from the qsys file.

#### Rebuild the hps platform

* Launch Platform Designer
* Open ndi\_hps.qsys
* Click "Generate HDL..."
* Select VHDL Synthesis output (optional)
* Click "Generate"
* Once the qsys project has been generated, run a normal Quartus compilation

## Known Issues and Limitations:

The video I/O logic (eg: HDMI to SDRAM and SDRAM to HDMI) is intended as an easy to understand minimally functional example and is not recommended for use in production. In particular, the input logic does not gracefully handle corrupt video data, unexpected cable removal, and similar signal quality issues.

It is expected the user of this SDK will have their own custom video I/O logic specific to their target hardware. If this is not the case and a production capable video I/O solution is desired, there are numerous IP cores available from Xilinx, Altera, and 3rd parties capable of supporting all major video interfaces (SDI, HDMI, Display Port, CSI/DSI, etc.).

## NDI IP Files and Use

### NDI Encoder

Two VHDL files are required to use the NDI Encoder. Both files should be compiled into the `NDI_Enc` library in the following order:

*   Xilinx Projects:

    ```
    Encode_Xilinx.vhdp
    Encode_x4.vhd
    ```
*   Altera Projects:

    ```
    Encode_Altera.vhd
    Encode_x4.vhd
    ```

There is also a component declaration file `Enc_Core_Comp.vhd` for use as a reference when instantiating the encoder core. Each encoder core `Enc_Core_E` includes a register I/O interface, a read-only bus master interface for reading raw video data, and a write-only bus master interface for writing compressed NDI data. Note the raw video memory and the compressed NDI memory do not need to be the same physical bank of memory. Using two memory banks can improve system performance, particularly on lower-end devices such as the Zynq 7000 and Cyclone-V families.

### NDI Decoder

Two VHDL files are required to use the NDI Decoder. Both files should be compiled into the `NDI_Dec` library in the following order:

*   Xilinx Projects:

    ```
    Decode_Xilinx.vhdp
    Decode_x4.vhd
    ```
*   Altera Projects:

    ```
    Decode_Altera.vhd
    Decode_x4.vhd
    ```

There is also a component declaration file `Dec_Core_Comp.vhd` for use as a reference when instantiating the encoder core. Each decoder core `Dec_Core_E` includes a register I/O interface, a read-only bus master interface for reading compressed NDI data, and a write-only bus master interface for writing raw video data. Note the raw video memory and the compressed NDI memory do not need to be the same physical bank of memory. Using two memory banks can improve system performance, particularly on lower-end devices such as the Zynq 7000 and Cyclone-V families.

### SDRAM Bus Interfaces

The bus-mastering memory interfaces are natively a sub-set of the Altera Avalon interface specification. These buses may be tied directly to an Avalon interface port in an Altera design. For Xilinx designs, clear-text bus shim logic is provided to convert the Avalon bus subset used by the Encoder core into a more standard AXI interface:

| File              | Function                                             |
| ----------------- | ---------------------------------------------------- |
| Avl\_Axi\_Rd.vhd  | Translates Avalon read bus to AXI read bus           |
| Avl\_Axi\_Wr.vhd  | Translates Avalon write bus to AXI write bus         |
| Avl2\_Axi\_rd.vhd | Merges 2 Avalon read buses into one AXI read bus     |
| AXI\_Reg\_Wr.vhd  | Interface AXI write bus to simple register write bus |
| AXI\_Reg\_Rd.vhd  | Interface AXI read bus to simple register read bus   |

Usage examples for these files can be found in the top-level files for the example designs.

#### NDI Encode

**Writes (compressed NDI data)**

There is clear-text logic in the Encode\_x4 file which merges the four lower bandwidth compressed write streams into a single write stream. The raw (uncompressed) audio data is also merged into this stream.

This write bus is connected to a cache coherent port to the hard processor system, eliminating the need for a kernel mode DMA driver.

**Reads (raw video data)**

For maximum performance, the four read streams are all brought out to the top level to provide maximum flexibility in scheduling SDRAM transactions. The NDI Encoder cores will tolerate fairly high latency on raw video reads, but require sufficient overall bandwidth to avoid stalling the NDI cores. Each of the NDI cores can process one pixel element (Y or C) per clock.

#### NDI Decode

**Reads (compressed NDI data)**

There is clear-text logic in the Decode\_x4 file which merges the four lower bandwidth compressed Read streams into a single AXI read stream.

This read bus is connected to a cache coherent port to the hard processor system, eliminating the need for a kernel mode DMA driver.

**Writes (raw video data)**

For maximum performance, the four write streams are all brought out to the top level to provide maximum flexibility in scheduling SDRAM transactions. The NDI Decoder cores will tolerate fairly high latency on raw video writess, but require sufficient overall bandwidth to avoid stalling the NDI cores. Each of the NDI cores can process one pixel element (Y or C) per clock. For maximum performance, the four write streams are all brought out to the top level.

### Full vs. Lite Encode Designs

The `Zybo-Z7-20-Lite` and `Arty-Z7-20` example designs are intended as a reference for a more limited platform. Only one SDRAM chip is used (16-bit SDRAM interface) and only two encoder cores are instantiated. This implementation is still capable of operation at resolutions up to 1080p60.

All other example designs instantiate a "full" implementation which includes four NDI Encode or Decode cores providing maximum performance and minimum latency.

## HDMI Input Logic

### Arria-10 SoC DevKit

The a10-socdk-enc design targeting the Arria-10 SoC Development Kit uses a slightly modified version of the Altera Arria10 HDMI Rx-Tx Retransmit HDMI example design. The top-level of this example design is modified to export the recevied HDMI video, audio, and AVI data to the top-level where they are connected to the video and audio input logic.

#### Rebuilding the NIOS HDMI software

* Insure you have manually installed Eclipse per the Altera documentation
* In Quartus, select Tools -> Nios II Software Build Tools for Eclipse
* Use `<NDI SDK Path>/ndi-fpga-refernce/a10-socdk-enc/software` as the Workspace directory
* Create a new project and BSP in Eclipse
  * File -> New -> Nios II Application and BSP from Template
  * SOPC Information File name: `<NDI SDK Path>/ndi-fpga-reference/a10-socdk-enc/rtl/nios/nios.sopcinfo`
  * Project Name: tx\_control
  * Project location: `<NDI SDK Path>/ndi-fpga-refernce/a10-socdk-enc/software/tx_control`
    * NOTE: Remove `rtl` from the default path
  * Project Template: Blank Project
  * Click: Finish
* Modify BSP settings
  * Right-click `tx_control_bsp`
  * Nios II -> BSP Editor...
  * In the Main tab, under Settings -> Common
    * Check: enable\_small\_c\_library
    * Check: enable\_reduced\_device\_drivers
    * File -> Save
    * Click: Generate BSP
  * Close BSP Editor
* Using a file browser, navigate to `<NDI SDK Path>/ndi-fpga-refernce/a10-socdk-enc/software/tx_control_src`
* Select all files, right-click, and select: Copy
* In Eclipse, right-click tx\_control and select: Paste
* Build: Project -> Build All (or press ctrl+B)
* Update MIF file used to build FPGA
  * Right-click tx\_control -> Make Targets -> Build
  * Select mem\_init\_generate
  * Click: Build

### Zynq 7000 HDMI Input

The Zynq-7000 based designs (i.e., the Arty and Zybo Z7 boards) use RTL based on two open-source projects to process serial HDMI data into the parallel video required by the NDI encoder. Low-level, serial-to-parallel data conversion and lane alignment are performed by RTL from the Digilent DVI2RGB core. The aligned 10-bit characters are then processed as HDMI packets and subpackets by RTL from the Hamsterworks HDMI core. Additional RTL is also provided which merges these two cores together and produces the required video and audio data formats for the NDI encoder. For additional details, see the `ip/extern/README.md` file.

### ZCU104 HDMI Input

The ZCU104 board uses the Xilinx HDMI 2.0 IP core. A full or evaluation license must be installed for this core prior to building the design with Vivado. This IP core also requires control software to monitor the incoming HDMI signal and make any adjustments required. This software is currently running "bare-metal" on one of the Cortex-R5 cores.

#### Rebuilding the Cortex-R5 HDMI Software

* If you have made any changes to the FPGA project:
  * Compile the ZCU104 FPGA design
  * Export the hardware to the SDK
    * Select `File -> Export -> Export Hardware...`
    * Select `<Include bitstream>` and click `Next`
    * Adjust path/filename if desired and click `Finish`
* Launch Vitis `Tools -> Launch Vitis IDE`
  * Set Workspace directory as desired
  * Click `Launch`
* Create a new Platform Project `File -> New -> Platform Project`
  * Set Project name to `ZCU104-Enc`
  * Click `Next`
  * Select `Create a new platform from hardware (XSA)` tab
  * Click `Browse`
  * Select file `ZCU104_Enc.xsa`
  * Set Operating System to `standalone`
  * Set CPU to `psu_cortexr5_0`
  * Uncheck `Generate boot components`
  * Click `Finish`
  * The `Platform Editor` window should open for the ZCU104-Enc platform
  * Select `ZCU104-Enc -> psu_cortexr5_0 -> standalone -> Board Support Package`
  * Click `Modify BSP Settings`
  * Select the `libmetal` and `openamp` libraries (used to communicate with Linux), leave all other libraries unselected
  * Switch the console to UART1
    * Select `Standalone`
    * Set the Value for stdin to `psu_uart_1`
    * Set the Value for stdout to `psu_uart_1`
  * Click `OK`
  * Be patient, the BSP generation can take a while.
* Import the HDMI Rx example project
  * In the Board Support Package window
  * Scroll down to `v_hdmi_rx_ss` under `Drivers` (it is near the bottom)
  * Select `Import Examples`
  * Select `RxOnly_R5`
  * Click `OK`
  * Be patient!
  * Expand `RxOnly_R5_1_system -> RxOnly_R5_1 -> src`
  * All file names below are relative to this path
* Update the DDR memory region in the linker script
  * Open the file `src/lscript.ld`
  * Set the `psu_r5_ddr_0_MEM_0` region details to match the rproc\_0\_reserved reserved memory region in the device-tree and save the file Base Address : 0x3ed00000 Size : 0x80000
* Change HDMI code to use UART1
  * Open `src/xhdmi_example.h`
  * At line 86, change UART\_BASEADDR to use UART1 and save the file #define UART\_BASEADDR XPAR\_XUARTPS\_1\_BASEADDR
* Update EDID Data (Optional)
  * Open `src/xhdmi_edid.h`
  * Edit constant `Edid[]` as desired
  * NOTE: EDID contents used for the example are in the file `ndi.4k.yuv.edid.txt`
* Build the project `Project -> Build Project`
* Copy the elf file to the ZCU104
  * The elf file can be found at the following path
  * `<workspace>/RxOnly_R5_1/Debug/RxOnly_R5_1.elf`
  * Copy this file to `/lib/firmware/` on the ZCU104
  * Rename the file as desired
  * Make sure you reference this filename when launching the R5 via remoteproc!

## HDMI Output Logic

### Arria-10 SoC DevKit

The a10-socdk-dec design targeting the Arria-10 SoC Development Kit uses a modified version of the Altera Arria10 HDMI Rx-Tx Retransmit HDMI example design. All Rx logic and the RxTx loopback logic is removed and the HDMI Tx fpll and iopll blocks use the REFCLK\_SDI signal as a clock reference instead of the HDMI Rx clock. The tx\_control NIOS code has been updated to reconfigure the output divisor for the fpll and iopll instance to switch between video modes.

#### Rebuilding the NIOS HDMI software

* Insure you have manually installed Eclipse per the Altera documentation
* In Quartus, select Tools -> Nios II Software Build Tools for Eclipse
* Use `<NDI SDK Path>/ndi-fpga-refernce/a10-socdk-dec/software` as the Workspace directory
* Create a new project and BSP in Eclipse
  * File -> New -> Nios II Application and BSP from Template
  * SOPC Information File name: `<NDI SDK Path>/ndi-fpga-reference/a10-socdk-dec/rtl/nios/nios.sopcinfo`
  * Project Name: tx\_control
  * Project location: `<NDI SDK Path>/ndi-fpga-refernce/a10-socdk-dec/software/tx_control`
    * NOTE: Remove `rtl` from the default path
  * Project Template: Blank Project
  * Click: Finish
* Modify BSP settings
  * Right-click `tx_control_bsp`
  * Nios II -> BSP Editor...
  * In the Main tab, under Settings -> Common
    * Check: enable\_small\_c\_library
    * Check: enable\_reduced\_device\_drivers
    * File -> Save
    * Click: Generate BSP
  * Close BSP Editor
* Using a file browser, navigate to `<NDI SDK Path>/ndi-fpga-refernce/a10-socdk-dec/software/tx_control_src`
* Select all files, right-click, and select: Copy
* In Eclipse, right-click tx\_control and select: Paste
* Build: Project -> Build All (or press ctrl+B)
* Update MIF file used to build FPGA
  * Right-click tx\_control -> Make Targets -> Build
  * Select mem\_init\_generate
  * Click: Build

### Zybo-Z7-20 HDMI Output

The Zybo-Z7 board uses the DVI output logic from the "Hamsterworks" HDMI project (ip/extern/HDMI/src/dvid\_output.vhd). This is a simple DVI output and no HDMI features (e.g., emedded audio) are currently supported. Both 74.25 MHz (720p) and 148.5 MHz (1080p60) pixel rates are supported.

### ZCU104 HDMI Output

The ZCU104 board uses the Xilinx HDMI 2.0 IP core. A full or evaluation license must be installed for this core prior to building the design with Vivado. This IP core also requires control software to configure the HDMI IP core. This software is currently running "bare-metal" on one of the Cortex-R5 cores.

#### Rebuilding the Cortex-R5 HDMI Software

* If you have made any changes to the FPGA project:
  * Compile the ZCU104 FPGA design
  * Export the hardware to the SDK
    * Select `File -> Export -> Export Hardware...`
    * Select `<Include bitstream>` and click `Next`
    * Adjust path/filename if desired and click `Finish`
* Launch Vitis `Tools -> Launch Vitis IDE`
  * Set Workspace directory as desired
  * Click `Launch`
* Create a new Platform Project `File -> New -> Platform Project`
  * Set Project name to `ZCU104-Dec`
  * Click `Next`
  * Select `Create a new platform from hardware (XSA)` tab
  * Click `Browse`
  * Select file `ZCU104_Dec.xsa`
  * Set Operating System to `standalone`
  * Set CPU to `psu_cortexr5_0`
  * Uncheck `Generate boot components`
  * Click `Finish`
  * The `Platform Editor` window should open for the ZCU104-Enc platform
  * Select `ZCU104-Dec -> psu_cortexr5_0 -> standalone -> Board Support Package`
  * Click `Modify BSP Settings`
  * Select the `libmetal` and `openamp` libraries (used to communicate with Linux), leave all other libraries unselected
  * Switch the console to UART1
    * Select `Standalone`
    * Set the Value for stdin to `psu_uart_1`
    * Set the Value for stdout to `psu_uart_1`
  * Click `OK`
  * Be patient, the BSP generation can take a while.
* Import the HDMI Rx example project
  * In the Board Support Package window
  * Scroll down to `v_hdmi_rx_ss` under `Drivers` (it is near the bottom)
  * Select `Import Examples`
  * Select `TxOnly_R5`
  * Click `OK`
  * Be patient!
  * Expand `TxOnly_R5_1_system -> TxOnly_R5_1 -> src`
  * All file names below are relative to this path
* Update the DDR memory region in the linker script
  * Open the file `src/lscript.ld`
  * Set the `psu_r5_ddr_0_MEM_0` region details to match the rproc\_0\_reserved reserved memory region in the device-tree and save the file Base Address : 0x3ed00000 Size : 0x80000
* Change HDMI code to use UART1
  * Open `src/xhdmi_example.h`
  * At line 96, change UART\_BASEADDR to use UART1 and save the file #define UART\_BASEADDR XPAR\_XUARTPS\_1\_BASEADDR
* Update the default video mode
  * Open `src/xhdmi_example.c`
  * Edit the default video resolution on line 2457 (optional)
  * Change the colorspace on line 2458 to XVIDC\_CSF\_YCRCB\_420 for 4Kp60 and XVIDC\_CSF\_YCRCB\_422 for other modes
  * Possible values are listed in the `XVidC_VideoMode` enum in the file `<workspace>/ZCU104-Dec/psu_cortexr5_0/standalone_domain/bsp/psu_cortexr5_0/include/xvidc.h`
* Fix build errors related to not having a test pattern generator:
  * Copy the tpg header files xv\_tpg.h and xv\_tpg\_hw.h from the Xilinx SDK install directory into the src folder
    * `C:\Xilinx\Vitis\2022.1\data\embeddedsw\XilinxProcessorIPLib\drivers\v_tpg_v8_4\src`
    * `workspace/TxOnly_R5_1/src`
  * Edit `xhdmi_example.c`
    * Comment the contents of the `XV_ConfigTpg` function (lines 322-373)
    * Comment the contents of the `ResetTpg` function (lines 378-383)
    * Comment the assignment to `Pattern` (line 1353)
    * Comment the call to XV\_ConfigTpg (line 1356)
  * Edit `xhdmi_menu.c`
    * Comment the assignment to `Pattern` (line 1514)
    * Comment the call to XV\_ConfigTpg (line 1517)
* Build the project `Project -> Build Project`
* Copy the ELF file to the ZCU104
  * The ELF file can be found at the following path
  * `<workspace>/TxOnly_R5_1/Debug/TxOnly_R5_1.elf`
  * Copy this file to `/lib/firmware/` on the ZCU104
  * Rename the file as desired
  * Make sure you reference this filename when launching the R5 via remoteproc!

### SoCKit-Dec

**WARNING:** The SoCKit project should be considered deprecated as the development board is no longer available for purchase. This design will be removed from future versions of the NDI Advanced SDK.

The SoCKit-Dec design targeting the Cyclone-V SoCKit Development board video output implementation (DVI\_TX.vhd) converts the YUV video data from the NDI decoder into RGB and drives both the on-board VGA output and the (optional) Terasic DVH-HSMC daughtercard via the HSMC connector.

This is a simple DVI output and no HDMI features (e.g., emedded audio) are currently supported. Both 74.25 MHz (720p) and 148.5 MHz (1080p60) pixel rates are supported.

## MIPI CSI Input Logic

### Xilinx Kria KV260

This project uses the Xilinx MIPI CSI-2 Rx subsystem as configured for the "Smart Camera Accelerated Application" from Xilinx. A tap is added to the video output of the MIPI CSI core allowing pixels to be sent to the NDI Encode logic.

## Changelog

### \[6.1.0-rc1] - 2024-09-13

#### Changed

* Reset version numbering to align with software SDK
* Memory simulation files now explicitly labeled as encoder or decoder specific
* Many changes to decoder and encoder cores (see NDI 6.1 hardware changes for details)
* Upgraded the message severity in Xilinx projects for missing memory initialization files (these are reported as errors now)
* Rename output files to be consistent with top-level project name

#### Added

* Encoder support for planar alpha
* Support for new packed and semi-planar video formats
* Support for 16-bit video (11-bits passed through SpeedHQ codecs)
* Add 6.1 decoder and encoder encrypted IP and memory initialization files to all example designs

#### Fixed

* NDI\_Enc change handling of VID\_BURST\_WIDTH generic so only legal Avalon burst lengths are generated. Previous logic used b"0000" to represent a 16 word burst when the correct value should be b"1\_0000", which becomes b"1111" when converted into an AXI arlen value
* NDI\_Enc VID\_BURST\_WIDTH generic set to 5 for Zynq 7000 projects

### \[1.5.4] - 2024-01-24

#### Added

* Example project for the Kria KV260 development board

#### Fixed

* Local reset logic in Vid\_In, Vid\_Out, and Vid\_Track was ignoring rst
* Bug in Avl\_Axi\_Wr could cause bus lockup under some conditions

### \[1.5.3] - 2023-08-25

#### Changed

* Xilinx projects updated to Vivado 2022.1
* Switch to performance optimized synthesis and implementation strategies to ease timing closure
* Merged Hamsterworks HDMI handling with Digilent PHY
* Update Xilinx encryption key to xilinxt\_2019\_02

### \[1.5.2] - 2023-01-12

#### Changed

* Update SoCKit-Dec project to Quartus 22.1

### \[1.5.1] - 2022-09-19

#### Added

* Initial example design for Arria-10 SoC Devkit
* NDI\_Dec now supports writing planar alpha when PLANAR\_ALPHA generic is true
* PLANAR\_ALPHA generic added to disable planar alpha logic if not needed

#### Fixed

* Inferred multiplier output register did not map to DSP block in Arria-10

#### Changed

* Update generics for NDI\_Enc to support different read and write bus parameters
* Update SoCKit-Dec project to Quartus 20.1.1
* Update SoCKit-Dec software files to SoC EDS 20.1 for U-Boot socfpga\_2021.10

### \[1.4.9] - 2022-07-12

#### Fixed

* Updated cache and user bits in Avl\_Axi\_Wr.vhd to address cache coherency issues on the Zynq 7000 Encoder example causing corrupted bitstreams

### \[1.4.8] - 2022-01-10

#### Added

* Semantic versioning to NDI cores version register
* wr\_alpha control bit to NDI\_Dec to enable writing alpha data

### \[1.4.7] - 2021-11-24

#### Added

* Disable bit to Vid\_Out.vhd
* Support for variable counter widths in Vid\_Track.vhd

#### Fixed

* Encode\_x4 updated to properly merge audio input data with fewer than 4 cores (previous fix was incorrect)
* Parallel audio left/right data swapped in Aud\_In.vhd

### \[1.4.6] - 2021-11-04

#### Added

* Missing dtsi files for Altera kernel

### \[1.4.5] - 2021-03-30

#### Fixed

* Encode\_x4 updated to properly merge audio input data with fewer than 4 cores
* Update encode projects to read back zeros when accessing decoder addresses

#### Changed

* Routed FPGA SDRAM status signals to LEDs for SoCKit design

### \[1.4.4] - 2021-03-02

#### Added

* Initial version of Arty-Z7-20-Enc

### \[1.4.3] - 2020-06-02

#### Fixed

* Bug in Preview logic when set to divide by 2 in wide mode (720p on the ZCU104)
* 4:2:2 to 4:4:4 conversion logic in DVI\_Tx.vhd

### \[1.4.2] - 2020-05-20

#### Fixed

* Problem with Altera specific Decode logic

### \[1.4.1] - 2020-03-25

#### Fixed

* Audio Output register address for ZCU104 Decode project

### \[1.4.0] - 2020-03-20

#### Changed

* Xilinx projects updated to Vivado 2019.2
* Intel (Altera) project updated to Quartus-Lite 19.1
* NDI Decode support

### \[1.3.4] - 2020-03-17

#### Added

* Initial version with audio output

#### Fixed

* Updated FIFO and command reset logic in Vid\_Out

### \[1.3.3] - 2019-10-07

#### Fixed

* Fixed quantized coefficient rounding in FPGA Encode logic so it matches the software

### \[1.3.2] - 2019-09-20

#### Changed

* NDI Decoder updated for Altera DSP blocks

### \[1.3.1] - 2019-08-29

#### Added

* ZCU104 decode reference design
* Decode and output support for 4Kp60 4:2:0 video

#### Changed

* Update FPGA logic to improve timings
* 4 Macroblock burst mode added to NDI\_Dec to improve SDRAM efficiency

### \[1.3.0] - 2019-07-08

#### Added

* Initial version with NDI Decode

#### Changed

* Added support for targeting all Encoder cores with a single write
* Updated Altera licenses

### \[1.2.2] - 2019-04-04

#### Added

* Overview of Encoder core: NDI\_SoC+FPGA\_Encoder.pdf

#### Changed

* Updated block diagram and migrated it into Encoder overview pdf file

#### Fixed

* Update Avl\_Axi\_Wr.vhd to avoid bus lockup under rare conditions
* Fix case mis-match with local.xdc file in Zybo-Z7-20-Lite project file

### \[1.2.1] - 2018-11-17

#### Added

* Zybo-Z7-20-Lite example design (16-bit SDRAM interface with 2 encoder cores)

#### Fixed

* Preview filter was not passing locked bit to output
* Missing signal declaration in "Hamsterworks" HDMI audio logic
* Encode\_x4 updated to properly support fewer than 4 cores

### \[1.2.0] - 2018-09-30

#### Added

* Audio support
* Video tracking and auto-format detection

#### Changed

* Renamed ZCU104 project directory

### \[1.1.2] - 2018-09-21

#### Added

* Altera IP core and license file (example design coming soon!)
* Hardware export directories
* Compiled bit files

#### Fixed

* Slightly improved NDI encoder efficiency to improve performance at 4Kp60 when using a 200 MHz clock
* Fix issue when switching between SD and HD modes
* Fix wedging issue with specific memory latency and wait state patterns

### \[1.1.1] - 2018-09-13

#### Added

* HDMI embedded audio extraction
* Digilent dvi2rgb added as an alternative video input

#### Fixed

* RGB to YCbCr color space conversion (B and R coefficients were swapped)

### \[1.1.0] - 2018-08-31

#### Added

* HDMI Input logic for Zybo platform, based on open-source code from Mike Field [hamster@snap.net.nz](mailto:hamster@snap.net.nz). Thanks Mike!!!
* Version register including platform specifier
* PS block design: axi\_gpio to interface to PL LEDs and push-button switches
* PS block design: axi\_iic for audio codec I2C bus (Zybo)
* Audio input logic added to Zybo project (not yet tested)
* Details on compiling HDMI Rx code for the Cortex-R5 on the ZCU104
* Automated zip file package builds thanks to git archive (git ROCKS! :) )

#### Fixed

* Critical warning regarding VIDEO\_CLK when building ZCU104 project
* Issues caused by running on platforms with both 128-bit (ZCU104) and 64-bit (Zybo-Z7) interfaces to SDRAM.

#### Changed

* README.txt switched to markdown format and renamed to README.md
* General code cleanup and removal of unnecessary files, comments, and deprecated code.
* Added actual purging logic to VidIn, rather than just resetting the FIFO when VSYNC is active.

### \[1.0.2] - 2018-08-07

#### Changed

* Add Build Dependencies section to the README file indicating Digilent board files must be installed to properly build the example Zybo project and an HDMI license is required for the ZCU104.

#### Fixed

* Deprecated file "NDI\_Pkg.vhd" removed from Zybo-Z7-20 project file.

### \[1.0.1] - 2018-07-26

#### Added

* Changelog.md

#### Fixed

* Deprecated file "NDI\_Pkg.vhd" removed from zynqmp.NDI project file.

### \[1.0.0] - 2018-07-13

* Initial version

### Changelog Hints & Details

* [Changelog format](https://keepachangelog.com/en/1.0.0/)
* [Semantic Versioning](https://semver.org/)
