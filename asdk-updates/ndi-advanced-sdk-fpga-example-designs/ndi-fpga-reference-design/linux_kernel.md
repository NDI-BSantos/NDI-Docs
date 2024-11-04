# linux\_kernel

## Altera Bootloader and Linux Kernel

The U-Boot bootloader and Linux kernel are built from git repositories provided by altera-opensource on github. See RocketBoards.org for full details.

### Directory Structure:

| Directory | Target Development Board     |
| --------- | ---------------------------- |
| a10-socdk | Arria-10 SoC Development Kit |
| SoCKit    | Arrow/Terasic SoCKit         |

### Building an Existing Project

The build instructions from Altera:

https://www.rocketboards.org/foswiki/Documentation/BuildingBootloaderCycloneVAndArria10

...involve quite a few manual steps. These steps have mostly been automated via a Makefile, with some additional changes to customize the kernel as needed for the NDI Advanced SDK (enable UIO devices and create custom device-tree nodes).

The Makefile will checkout the source directories, patch and configure as required, then build the files required to boot and create a uSD image. Warnings about watchdog and timer drivers are expected while building U-Boot from the Altera released sources can be ignored.

The configuration snippet `uio.fragment` enables the required uio kernel modules.

The patch file `0001-NewTek-Altera-NDI-device-tree-entries.patch` adds the required device-tree nodes for logic in the FPGA fabric. See the instructions from Altera for full details:

https://www.rocketboards.org/foswiki/Documentation/HOWTOCreateADeviceTree

#### Note:

If you follow the RocketBoards instructions for generating the Cyclone-V boot files, the FPGA will not be programmed by default. The provided Makefile builds a u-boot.scr file which is executed by the default Altera U-Boot bootcmd. This script programs the FPGA prior to loading the Linux kernel and device-tree.

## PetaLinux Projects for NDI Xilinx Hardware Platforms

All steps below require you have an appropriate version of the Xilinx PetaLinux tools installed on your development machine. Refer to Xilinx UG1144 "PetaLinux tools Documentation: Reference Guide" for detailed instructions on installing PetaLinux and the required prerequisites.

### Generating Example Designs

Example Petalinux projects for the Digilent Zybo Z7-20 and Xilinx ZCU104 are provided as prebuilt Xilinx support archives (XSA files from Vivado), build scripts and a Makefile to allow end users to regenerate complete projects and boot images without being required to download large archives or binary files.

For example:

```
# To create an NDI encoder example design for the Zybo Z7-20
$ make zybo-enc
```

The following boards and NDI configurations are supported (see `make help` for a list of available targets):

| Configuration       | Contents                                                  |
| ------------------- | --------------------------------------------------------- |
| Arty-Z7-20-Enc      | NDI encoder for the Digilent Arty Z7-20                   |
| Zybo-Z7-20-Enc      | NDI encoder for the Digilent Zybo Z7-20                   |
| Zybo-Z7-20-Enc-Lite | NDI encoder for the Digilent Zybo Z7-20 with 16-bit SDRAM |
| Zybo-Z7-20-Dec      | NDI decoder for the Digilent Zybo Z7-20                   |
| ZCU104-Enc          | NDI encoder for the Xilinx ZCU104                         |
| ZCU104-Dec          | NDI decoder for the Xilinx ZCU104                         |

Building SD card images, containing a root filesystem, application software, and operating system, requires that Petalinux projects in these locations be created so that the kernel, device tree, and bootloaders can be found. See `../uSD_os/README.md` for details on how this should be performed.

Generating example designs for all supported platforms requires a significant amount of disk space (at least 150GB) so it is recommended that users only build example designs for their platform of choice. Additionally, PetaLinux 2022.1 must be installed per the instructions detailed in Xilinx UG1144, "PetaLinux Tools Documentation: Reference Guide". Among other things, the petalinux tools must be installed in a location that is writable by the current user. This location needs to be provided prior to running `make` via the `PETALINUX_ROOT` environment variable.

### Rebuilding Example Designs after Modification

#### Xilinx FPGA IP Added or Removed

If you add or remove any Xilinx IP blocks with Linux kernel drivers, the PetaLinux project needs to be updated to incorporate the changes. The easiest way to do this is simply follow the directions for "Building a new project from scratch", below, skipping the petalinux-create command. This will complete much faster than the initial build, since most of the build artifacts are cached and can be reused.

The minimum process to update a project with new hardware definitions is:

*   Export the hardware platform

    In Vivado, select the `File -> Export -> Export Hardware` menu option and make sure that the bitstream is included in the exported Xilinx support archive.
*   Import the new hardware definition and reconfigure the project:

    ```
    cd <projectname>
    petalinux-config --get-hw-description=/path/to/vivadoproject/project.xsa
    ```
*   Reconfigure bootloader or kernel if desired

    ```
    petalinux-config -c u-boot
    petalinux-config -c kernel
    ```
*   Rebuild the device tree and bootloader

    ```
    petalinux-build -c u-boot
    ```
*   Package the output products

    ```
    petalinux-package --boot --fpga /path/to/<bitstream> --u-boot
    ```

This will yield updated files in `projectdir/images/linux`, notably the BOOT.BIN and device tree, which can then be copied to the boot medium.

#### FPGA Bitstream Updated

If you update the FPGA bit file with hardware changes that do not alter the automatically generated device-tree, the update process is much simpler. Simply rebuild the boot files using the updated bit file:

```
# Rebuild the boot files
petalinux-package --boot --fpga /path/to/<bitstream> --u-boot --force
```

#### Device Tree Changes

If you update the system-user.dtsi file, the system device tree needs to be updated and the changes need to be incorporated into the boot loader.

```
# Rebuild just the boot loader
petalinux-build -c u-boot

# Rebuild the boot files
petalinux-package --boot --fpga /path/to/<bitstream> --u-boot --force
```

### Modifying Example Designs

To modify an existing example design after it is generated, use the following general procedure:

```
# Setup the Petalinux environment
source /path/to/petalinux/2022.1/settings.sh

# Change to the appropriate project directory
cd projectdir

# To make project level changes (e.g., image packaging configuration)
petalinux-config

# To reconfigure other components such as U-Boot
petalinux-config -c u-boot

# New output products can then be built, which PetaLinux will place
# at projectdir/images/linux
petalinux-build
```

Once new PetaLinux output products have been created in `images/linux`, new boot images can be built from scripts in the `../uSD_os` directory.

### Building a New Project From Scratch

If you are not using one of the supported development boards you may need to build a PetaLinux project from scratch. Full details for working with PetaLinux can be found in Xilinx UG1144 "PetaLinux Tools Documentation: Reference Guide". A brief summary of one possible work flow is provided below.

* Create a Vivado project for your target board or component. Note that during project creation, the option of a Vitis extensible platform should _not_ be selected. Vitis is not required at any point in this design flow.
* Build the hardware project in Vivado
* Export the hardware platform and include the bitstream in the exported support archive (i.e., the XSA file).
*   Create a new PetaLinux project for the target platform

    ```
    # For Zynq-7000
    petalinux-create -t project -n <projectname> --template zynq

    # For Zynq Ultrascale
    petalinux-create -t project -n <projectname> --template zynqMP
    ```

If your design needs required a board support package (BSP) then project creation will need to reference this via the `--source` argument.

*   Configure the Petalinux project and import the hardware definition, usually through a command of the form

    ```
    cd <projectname>
    petalinux-config --get-hw-descrtipion /path/to/project/exported.xsa
    ```
*   Customize the PetaLinux project (optional)

    ```
    # Configure the bootloader, kernel, and other components
    petalinux-config -c u-boot
    petalinux-config -c kernel
    ```
*   Modify the system-user.dtsi to reflect the FPGA hardware or other device tree changes necessary

    Xilinx IP cores should be automatically added to the device tree, but details for any custom FPGA logic must be added manually. Details will depend on the specifics of your FPGA project. Refer to the existing projects for examples:

    ```
    projectdir/project-spec/meta-user/recipes-bsp/device-tree/files/system-user.dtsi
    ```

    Some additional information can be found by looking at the device tree source files that ship with the example design in the `device-tree` directory
*   Build the PetaLinux project

    ```
    petalinux-build
    ```

    If the build (or the configure) commands fail, attempt to run the build again and examine the log files.
*   Create boot files

    Generally, a BOOT.BIN needs to be created that contains the FSBL, PMU firmware (for the US+), second stage bootloader (e.g., U-Boot) and a device tree. The specific command options needed will depend on the configuration options needed for the target platform. The following example is for a uSD image with the boot loader programming the FPGA prior to booting the Linux kernel:

    ```
    petalinux-package --boot --fpga /path/to/fpga.bit --u-boot --fsbl

    # If you have previously generated the boot files and want to rebuild them,
    # you need to add the --force flag
    petalinux-package --boot --fpga /path/to/fpga.bit --u-boot --fsbl --force
    ```

#### Notes

* The `petalinux-build` step will occasionally fail when performing a full build from scratch. Simply re-run the `petalinux-build` command and the build will continue. Sometimes it is necessary to re-run the `petalinux-build` step several times.
* If the `petalinux-build` step repeatedly fails in the same place, examine the log files as there may be missing dependencies.
* Output files will be in the `projectdir/images/linux/` directory.

### Changelog

#### \[6.1.0-rc1] - 2024-09-13

**Changed**

* Reset version numbering to align with software SDK
* Update Makefile to use new exported FPGA output file names

#### \[1.5.3] - 2023-08-25

**Changed**

* Migrated all PetaLinux projects from 2019.2 to 2022.1
* Kernel version for Xilinx projects changed to 5.15.19
* All Zynq-7000 and UltraScale+ projects are automatically generated via make and bash scripts.
* Petalinux project configuration and device tree modifications automatically incorporated

#### \[1.5.2] - 2023-01-12

**Added**

* SoCKit updated to Quartus 22.1

#### \[1.5.1] - 2022-09-22

**Added**

* Initial support for Arria-10 SoC Development Kit
* ndiname device-tree node for use as NDI machinename

**Changed**

* Removed deprecated Petalinux 2018.1 projects
* Move reserved memory regions in Zybo-Z7-20-Lite device-tree

#### \[1.4.2] - 2021-03-08

**Added**

* Initial support for Arty-Z7-20

#### \[1.4.1] - 2020-03-27

**Fixed**

* R5 remoteproc device-tree entries for ZCU104-Dec project were broken

**Added**

* Added prebuilt boot files required to generate uSD images

#### \[1.4.0] - 2020-03-25

**Changed**

* Updates for NDI v4.5
* Petalniux projects updated to 2019.2
* NDI Decode support

#### \[1.3.3] - 2020-03-08

**Added**

* Petalinux 2019.2 project for ZCU104 ndi\_encode

#### \[1.3.2] - 2020-01-07

**Changed**

* Modified Zybo boot arguments to support new partition layout
* Updated pre-compiled files with latest FPGA bit files

**Fixed**

* Added missing pre-comiled U-Boot files for ZCU104 projects

#### \[1.3.1] - 2019-07-11

**Fixed**

* Updated pre-compiled files to allow update of the various FPGA bit files (NDI Encode, Encode-Lite, and Decode) without rebuilding from scratch

#### \[1.3.0] - 2019-07-01

**Added**

* Updates for NDI v4.0

**Changed**

* Updated system-user.dtsi files to support NDI Decode
* Updated pre-compiled files

#### \[1.2.1] - 2018-11-15

**Fixed**

* Missing dash in `--get-hw-description` petalinux command

#### \[1.2.0] - 2018-09-30

**Added**

* Added pre-compiled boot files, allowing update of the FPGA bit file without recompiling PetaLinux from scratch
* Added tracking logic to system-user.dtsi

**Changed**

* Tally LEDs default to on at boot instead of the heartbeat trigger
* Unused LEDs set to cpu and uSD trigger

#### \[1.1.1] - 2018-09-24

**Changed**

* Update boot.sh scripts to reference bit file in `src/fpga/<project>` directories

**Removed**

* FPGA bit files (migrated to `src/fpga/`)
* Hardware export directories (migrated to `src/fpga/`)

#### \[1.1.0] - 2018-09-05

**Added**

* Hardware export sdk directories from Vivado

**Changed**

* Projects updated with latest FPGA hardware export
* Device-tree entries added for tally LEDs

#### \[1.0.0] - 2018-08-17

* Initial version
