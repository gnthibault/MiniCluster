# Installing Linux on the H96 pro+

## Introduction
H96 pro+ is a box equipped with the S912 SOC (system on chip) from Amlogic.
Fortunately, Amlogic is working on upstreaming its linux kernel patch, see for instance their dedicated website: [](http://linux-meson.com/doku.php#bit_socs_gxbb_s905_or_newer)

It is also interesting to notice that they give away their buildroot tree, ready for compiling 3.14 and 4.9 kernel, see [](http://openlinux.amlogic.com/wiki/index.php/Arm/Buildroot)

## Our work

Our first choice is to decide if we want to build our own distribution fro scratch, using buildroot for instance, or try to use a prebuilt image, like the one built by balbes150, which designed a nice set of tools for building ubuntu images for embedded system, see [](https://github.com/150balbes/lib) and [](https://forum.armbian.com/index.php?/topic/2138-armbian-for-amlogic-s912/&)

In a first approach, we decided to use the most simple solution: use a prebuilt image of ubntu Xenial 16.04

## The dtb file

Our first task was to get the device tree file from our H96 pro + box. we hereafter report the solution described on the armbian forum:


