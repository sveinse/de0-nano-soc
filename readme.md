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
development steps often happen independently of the software Linux development,
so this framework allows for handovers from step to step.

The assumptions in this setup are:

 * Quartus and SoCEDS installed on system
 * Build for DE0-Nano-SoC / Atlas-SoC
 * Run Linux on the SoC
 * Boot from a SDCard


## Quick start

To build a complete FPGA based SoC, there are multiple steps involved

 1. Build the FPGA system
 2. Build the boot system
 3. Build the target file system, including kernel
 4. Create a SDcard

This project implements a simple build framework for building the FPGA aspects.


### 1) Build the FPGA system

### 2) Build the boot system

### 3) Build the target file system

### 4) Create a bootable SDcard
