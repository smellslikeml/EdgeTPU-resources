# FLIR Cameras on the EdgeTPU dev board

## Overview
The EdgeTPU dev board was designed to easily prototype edge AI applications and scale to production. For higher
production computer vision projects, you may want to scale up your camera hardware from the Google Coral Camera or a
webcam. This guide will walk you through intalling FLIR's Spinnaker SDK and python bindings to interface with industrial
grade FLIR cameras.

## Requirements
* EdgeTPU dev board
* Mendel OS
* [Spinnaker SDK + python bindings](https://flir.app.boxcn.net/v/SpinnakerSDK)
* Additional ffmpeg related packages (included here)
* libusb-1.0-0

## Installation
Although Mendel OS is not officially supported by FLIR, it is similar enough in architecture to Ubuntu 16.04 that we can
use their ARM packages and install the sdk and bindings successfully.

The Spinnaker SDK requires on some additional ffmpeg packages that are not readily available with Mendel OS's package
manager. You can find these additional packages on [Ubuntu Launchpad](https://launchpad.net/ubuntu?) specifically for
Ubuntu 16.04 running on ARM64 architecture. We've included all the additional packages required in a zip file called
spin_extradebs.zip in this repo.

Navigate to the FLIR Spinnaker SDK box [download link](https://flir.app.boxcn.net/v/SpinnakerSDK) and choose the Linux
directory, then traversing down to the Ubuntu 16.04 directory. You want to make sure you select the package that is for ARM64
architecture. Download onto your local computer and scp it to your dev board using the mdt tool

You also want to download the pip wheel for the python bindings. Choose the python folder, then traversing down to the
ARM64 directory. Download the pip wheel for your prefered python version, in this case 3.5. Scp this wheel and the
spin_extradebs.zip file in this repo to your dev board as well.
```
# On your local computer
$ mdt push /path/to/spinnaker-1.26.0.31-Ubuntu16.04-arm64-pkg.tar.gz 
$ mdt push /path/to/spinnaker_python-1.26.0.31-Ubuntu16.04-cp35-cp35m-linux_aarch64.tar.gz
$ mdt push /path/to/EdgeTPU-resources/spin_extradebs.zip
```

On the dev board, expand the packages:
```
$ tar xvzf spinnaker-1.26.0.31-Ubuntu16.04-arm64-pkg.tar.gz
$ tar xvzf spinnaker_python-1.26.0.31-Ubuntu16.04-cp35-cp35m-linux_aarch64.tar.gz
$ unzip spin_extradebs.zip
```

Install dependencies:
```
$ sudo apt-get install libusb-1.0-0
$ sh install.sh
```

This should install all the prerequisites required before installing the Spinnaker SDK. If this is a fresh install of
Mendel OS on your dev board, you may run into other missing packages. Normally your package manager will warn you of
other dependencies. If this is the case, simply run ``` sudo apt-get install --fix-broken ``` to install the missing
dependencies.

Install Spinnaker SDK:
```
$ cd ~/spinnaker-1.24.0.60-arm64
$ sudo sh install_spinnaker_arm.sh
```

The install script will prompt you along the way. Say "yes" to all prompts and type "mendel" as the user when prompted
to add a user.

Next install the python bindings
```
$ cd ~/
$ sudo -H pip3 install spinnaker_python-1.24.0.60-cp35-cp35m-linux_aarch64.whl
```

Now enter a python3 session and check that everything was installed correctly:
```
$ python3
>>> import PySpin
```

### Troubleshooting
If you can recognize a camera but streaming is failing, it is most likely that the usb interface is underpowering the
camera. Externally power the camera via the GPIO power supply.

You may also encounter failures when transfering large images being captured by your FLIR camera. You can increase your
USB buffer size to fix this.

```
$ sudo sh -c 'echo 1000 > /sys/module/usbcore/parameters/usbfs_memory_mb'
```

## References
* [Spinnaker on ARM and Embedded Systems](https://www.flir.com/support-center/iis/machine-vision/application-note/using-spinnaker-on-arm-and-embedded-systems/)
* [Spinnaker SDK download](https://flir.app.boxcn.net/v/SpinnakerSDK)
* [Launchpad](https://launchpad.net/ubuntu/xenial/arm64/)
