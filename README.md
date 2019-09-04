# Coral EdgeTPU Dev Board Resources
A collection of useful resources and hacks around the new Coral EdgeTPU dev board. 

## Quick Overview

From Google Coral [official docs](https://coral.withgoogle.com/docs/dev-board/get-started/), the Coral Dev Board is a
single-board computer that contains an Edge TPU coprocessor. It's ideal for prototyping new projects that demand fast
on-device inferencing for machine learning models.


The dev board runs Mendel OS, a Debian based linux OS developed by Google specifically designed for the hardware. This
board requires the operating system to be flashed onto the device, using fastboot from the Android SDK platform tools. 
[Here](https://coral.withgoogle.com/docs/dev-board/get-started/#flash-the-board) are the official instructions for flashing Mendel OS onto a coral dev board.

## Mendel OS
Inspecting the linux kernel that comes by default:
```
$ uname -a
Linux undefined-calf 4.9.51-imx #1 SMP PREEMPT Tue May 14 20:34:37 UTC 2019 aarch64 GNU/Linux
```

We can see that the linux kernel version is 4.9.51 which is close to the version that was released with Ubuntu 16.04.
The architecture is ``` aarch64 ``` which is the same as ``` ARM64 ```, which is useful to keep in mind when installing 
packages for embedded devices.
Looking at the libc version and printing the debian version:
```
$ /lib/aarch64-linux-gnu/libc.so.6
GNU C Library (Debian GLIBC 2.24-11+deb9u4) stable release version 2.24, by Roland McGrath et al.
Copyright (C) 2016 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.
There is NO warranty; not even for MERCHANTABILITY or FITNESS FOR A
PARTICULAR PURPOSE.
Compiled by GNU CC version 6.3.0 20170516.
Available extensions:
	crypt add-on version 2.1 by Michael Glad and others
	GNU Libidn by Simon Josefsson
	Native POSIX Threads Library by Ulrich Drepper et al
	BIND-8.2.3-T5B
libc ABIs: UNIQUE
For bug reporting instructions, please see:
<http://www.debian.org/Bugs/>.
```

```
$ cat /etc/debian_version
9.7
```

We can see that Mendel OS is based on Debian 9 (stretch).
Overall, Mendel OS is a cousin of Ubuntu 16.04 ARM64 and Raspbian stretch, which means we can use these debian packages
when packages for Mendel OS are not available. This is relevant for installing packages like ROS Kinectic ([install guide
here](https://github.com/smellslikeml/EdgeTPU-resources/blob/master/ROS_kinetic.md)). 

## Guides
* [Install ROS Kinetic on EdgeTPU dev board](https://github.com/smellslikeml/EdgeTPU-resources/blob/master/ROS_kinetic.md)
* [EdgeTPU-powered Donkey Car](https://github.com/smellslikeml/EdgeTPU-resources/blob/master/donkeycar_EdgeTPU.md)
* [FLIR Cameras on the EdgeTPU dev board](http://smellslikeml.com)

## Resources
* [Official Getting Started Guide](https://coral.withgoogle.com/docs/dev-board/get-started/)
* [Coral Dev Board Datasheet](https://coral.withgoogle.com/docs/dev-board/datasheet/)
* [EdgeTPU Compiler](https://coral.withgoogle.com/docs/edgetpu/compiler/#)
* [Community Coral guide](https://github.com/f0cal/google-coral)

