# DE0-Nano-SoC / Atlas-SoC

This project contains a simple FPGA build framework for the
[DE0-Nano-SoC / Atlas-SoC](http://www.terasic.com.tw/cgi-bin/page/archive.pl?Language=English&CategoryNo=163&No=941&PartNo=1)
development platform.

![Image of DE0-Nano-Soc / Atlas-SoC](img/de0-nano-soc.jpg)

More info about the kit can be found in
https://rocketboards.org/foswiki/view/Documentation/AtlasSoCDevelopmentPlatform

Maintained by Svein Seldal <sveinse@seldal.com>. Last updated 2017-11-19.


## Introduction

This repo is based on the GHRD design found in Terasic System CD v1.1.0. The
build system, the Makefile, is based on Altera's reference design found in the
SoCEDS installation files. This Makefile has some shortcomings and this project
has made some improvements to it.

The goal of the project is to make a simple workflow for compiling software for
the DE0-Nano-SoC / Atlas-SoC kit. The framework also acknowledge that the FPGA
development often happen independently of the software Linux development,
so this framework allows for handovers from step to step and from machine to
machine.

The assumptions in this setup are:

 * Intel Quartus Prime and SoC EDS installed on system
 * Build for DE0-Nano-SoC / Atlas-SoC
 * Run Linux on the SoC
 * Boot from a SDCard


## Setup

The project is based on a single `Makefile`. Typing `make` or `make help` in
the command-line will print a simple overview of the major commands. Much of
the Makefile relies on having the Intel Quartus Prime tools available.
To get full use of the makefile, please use it from the SoC EDS command shell.

To clean an project directory use `make scrub`. Note that the cleaning is
quite aggressive on unknown files, so be careful to not have other files in
the project tree. The `Makefile` has a `FILES_KEEP=` setting close to
protect files from being scrubbed.


## Overview

Multiple steps are required to build the image for a complete FPGA based
SoC. 

 1. Build the FPGA system
 2. Build the boot system
 3. Build the target file system, including kernel
 4. Create a SDcard


## Step 1 - Build the FPGA system

This step is compiling the FPGA image. It uses the Intel Quartus Prime
toolsuite. It supports both an interactive GUI based development and compiled
from scripts.

The output from this step is the handoff file found in
`output_files/fpga_files.tar.gz`.


### Project setup

To make the Quartus project compatible with the build structure required
by the Makefile, please set the following project setting using either of
these two methods:

  1) In Quartus GUI using Settings > Compilcation Process Settings > Save
     project output files in specified directory and enter 'output_files'

  2) Set in Quartus settings file *.qsf 

```
     set_global_assignment -name PROJECT_OUTPUT_DIRECTORY output_files
```


### Step 1a - Interactive FPGA Compilation in Quartus Prime

If GUI development is preferred, the project can be built using the following
steps:

  1. Open project (*.qpf) in Quartus Prime
  2. Open QSys and open *.qsys file and generate HDL. Finish
  3. Run Analysis & Sythesis (Ctrl+K)
  4. Run tcl scripts for SDRAM assignments
```
     soc_system/submodules/hps_sdram_p0_paramters.tcl
     soc_system/submodules/hps_sdram_p0_pin_assignments.tcl
```
  5. Compile project (Ctrl+L)
  6. Use programmer to program device

To build the handoff file, `output_files/fpga_files.tar.gz`, open SoC EDS
Command Shell:

```
     make quartus_ok
     make fpga
```

When FPGA outputs are generated from the GUI, the make system must be told with
`make quartus_ok` that the build is complete. It will ensure that the build
isn't started over.


### Step 1b - Scripted FPGA Compilation

An alterative to interactive FPGA compilation is to compile it from the
command shell. To generate the handoff file file, `output_files/fpga_files.tar.gz`
use, open the SoC EDS Command Shell

```
    make fpga
```

Other manual operation exists:

  * `make quartus_compile`
  * `make quartus_edit`
  * `make qsys_compile`
  * `make qsys_edit`
  * `make program_fpga`


## Step 2 - Build the boot system

The boot system provides all necessary files which the FPGA requires for
starting up the device. The boot system consist of the preloader, bootloader,
boot script, the device tree and the FPGA image in a format that can be used
by the bootloder. These files are placed in a separate FAT partition of the
SD-Card

For Intel SoC the SD-Card boot files consists of:
  
  * `preloader-mkpimage.bin` - Preloader image
  * `u-boot.img` - Bootloader image
  * `boot.script` - Bootloader script. Generated from `boot/boot.script`
  * `soc_system.sof` - FPGA image file
  * `soc_system.rbf` - FPGA image file, generated from the *.sof
  * `soc_system.dts` - Linux Device Tree Source
  * `soc_system.dtb` - Linux Binary Device Tree, generated from *.dts
  * `zImage` - Linux kernel -- *) Not included in the handoff file

The output from this step is the handoff file found in
`output_files/sd_fat.tar.gz`.


### Step 2a - Prepare using FPGA handoff file

This step is needed if starting from an empty or cleaned project. The FPGA
handoff file must be copied into the project and unpacked to set the project
up for building the boot files. This step is not necessary to do if the build
already contains a completed compilation from step 1.

To unpack a FPGA handoff file:

```
    make FPGA_TGA=file fpga-unpack
```

If the `FPGA_FILE=file` option is omitted, make will search for it in
`output_files/fpga_files.tar.gz`


### Step 2b - Build the boot files

To compile the boot handoff file, `output_files/sd_fat.tar.gz`

```
    make sd-fat
```


## Step 3 - Build the target Linux file system

This repository does not include any tools for building the Linux file system.

FIXME: Where to go next

The expected handoff files from this step is the Linux kernel, `zImage`, and
the Linux root file system, `rootfs.tar.gz`.


## Step 4 - Create a bootable SD-card

To create a bootable SD-card, you need the following files from previous steps:

  * The boot files, `sd_fat.tar.gz` -- REQUIRED
  * A Linux root file system, `rootfs.tar.gz` -- REQUIRED
  * A Linux kernel, `zImage` -- can be omitted if included in either the boot
    files or the main file system
  * Preloader, `preloader-mkpimage.bin` -- Normally included in the boot files
  * Bootloader, `u-boot.img` -- Normally included in the boot files

This project includes `bin/mksdcard`, a tool which can be used to generate
SD-Cards. It creates the required partitions and layout and copies in the data,
including the special preloader and bootloader data partition. It can generate
both image files and write to actual device. See
    `bin/mksdcard --help`
for help and options.

To make the SD-Card (image), collect the above files into the current directory
and run the following command:

```
    # Write to a file image
    sudo bin/mksdcard sdcard.img

    # Write to a block device
    sudo bin/mksdcard /dev/mmcblk0
```

`mksdcard` have options for specifying the location of the various files if
copying the files to the local directory isn't what you want.

NOTE! While it is possible to write to an image file and then later write
      to an actual SD-Card using e.g. `dd`, it is not a particular efficient
      scheme. The resulting file image will contain a lot of empty sectors,
      which dd and similar tools will have to write. It is generally much
      faster to write the physical SD-Card directly from this tool, since this
      involves far less write operation. Additionally the partitions will 
      partitioned to fill the card, which with the image it might not.
