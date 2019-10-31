# Installing ROS Kinectic on the Coral EdgeTPU dev board

## Overview
ROS Kinetic is an ROS version that is readily available for ARM architectures. There are various supported
methods for installing it on devices like the Raspberry Pi or NVIDIA Jetson but little documentation for other devices
such as the coral dev board. Since Mendel OS is a cousin of Ubuntu 16.04 ARM64 and a derivative of Debian 9 (stretch),
we can tweak the [build installation instructions for the raspberry
pi](https://wiki.ros.org/ROSberryPi/Installing%20ROS%20Kinetic%20on%20the%20Raspberry%20Pi) to successfully install ROS
Kinetic on the coral dev board. This guide will be based on the raspberry pi build instructions.

## Requirements
Hardware:
* Google Coral EdgeTPU dev board
Software:
* Mendel OS
* dirmngr
* libboost-all-dev
* Assimp

## Installation

### Step 1: Setup dependencies
Install dirmngr
```
$ sudo apt-get install dirmngr
```

Grab ROS repository
```
$ sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu stretch main" > /etc/apt/sources.list.d/ros-latest.list'
$ sudo apt-key adv --keyserver hkp://ha.pool.sks-keyservers.net:80 --recv-key 421C365BD9FF1F717815A3895523BAEEB01FA116
```

In the original instructions, the first line uses ```lsb_release -sc ``` to get the name of the release. For Mendel OS,
the release name is ```chef``` for which there are no repositories. We'll use Mendel OS's resemblance to Debian stretch
to replace the name.

Update packages:
```
$ sudo apt-get update
$ sudo apt-get upgrade
```
You may get an error stating that the signatures couldn't be verified because the public key is not available and will
print out the unavailable key. Run the following to resolve it and update/upgrade again
```
$ sudo apt-key adv --keyserver ha.pool.sks-keyservers.net --recv-keys <PRINTED-KEY-HERE>
```

Install Bootstrap dependencies:
```
$ sudo apt-get install -y python-rosdep python-rosinstall-generator python-wstool python-rosinstall build-essential cmake
```

Install Boost library
```
$ sudo apt-get install libboost-all-dev
```

Initialize rosdep
```
$ sudo rosdep init
```

Running ``` rosdep update ``` will try to autodetect the os which will lead to an error stating that the OS
is not among the supported OS distros. To resolve this, edit the os_detect.py file and change the following:
```
# with editor of choice
$ sudo vim /usr/lib/python2.7/dist-packages/rospkg/os_detect.py

# On line 643, update the else clause to look like this:
else:
    self._os_name = "debian"
    self._os_version = "9.0"
    self._os_codename = "stretch"
    self._os_detector = None
```
Save the edits and run:
```
$ rosdep update
```

### Step 2: Download and build ROS Kinetic
Create a catkin workspace:
```
$ mkdir -p ~/ros_catkin_ws
$ cd ~/ros_catkin_ws
```

Fetch core ROS packages and build:
```
$ rosinstall_generator ros_comm --rosdistro kinetic --deps --wet-only --tar > kinetic-ros_comm-wet.rosinstall
$ wstool init src kinetic-ros_comm-wet.rosinstall
```

Build the assimp library inside the catkin workspace:
```
$ mkdir -p ~/ros_catkin_ws/external_src
$ cd ~/ros_catkin_ws/external_src
$ wget http://sourceforge.net/projects/assimp/files/assimp-3.1/assimp-3.1.1_no_test_models.zip/download -O assimp-3.1.1_no_test_models.zip
$ unzip assimp-3.1.1_no_test_models.zip
$ cd assimp-3.1.1
$ cmake .
$ make
$ sudo make install
```

Resolve dependencies using rosdep:
```
$ cd ~/ros_catkin_ws
$ rosdep install -y --from-paths src --ignore-src --rosdistro kinetic -r --os=debian:stretch
```

This should have ROS Kinetic built on your device. For a quick check, run:
```
$ rosversion roscpp
```
And it should return *kinetic*.

### Step 3: Build catkin workspace
This step may take the longest and we recommend not using all your cores when running ``` make ``` near the end as this
can overwhelm the board and get stuck. We'll also alocate some swap memory to help with a smooth build.

Create the temporary swap space of 1G:
```
$ sudo fallocate -l 1G /swapfile
$ sudo chmod 600 /swapfile
$ sudo mkswap /swapfile
$ sudo swapon /swapfile
```

Now invoke catkin_make_isolated from within the ros_catkin_ws directory. We only used one core to make sure the board
was not overwhelmed, but increase core count as you wish.
```
# cd ~/ros_catkin_ws
$ sudo ./src/catkin/bin/catkin_make_isolated --install -DCMAKE_BUILD_TYPE=Release --install-space /opt/ros/kinetic -j1
```

### Troubleshoot
You may run into Boost library version issues. Earlier, we installed libboost-all-dev which will install all the newest
boost modules which will be around version ~1.62. If this newer version is causing errors, purge the package installed by the package
manager and manually install boost 1.58 for ARM64 which can be found
[here](https://launchpad.net/ubuntu/xenial/arm64/libboost-all-dev/1.58.0.1ubuntu1). 
```
# purge new boost library
$ sudo apt-get remove --purge libboost-all-dev
# download the boost 1.58 deb package and install with dpkg
$ sudo dpkg -i /path/to/libboost-all-dev_1.58.0.1ubuntu1_arm64.deb
```
Invoke catkin_make_isolated again.

## Resources
* [Install ROS Kinetic on the Raspberry Pi](https://wiki.ros.org/ROSberryPi/Installing%20ROS%20Kinetic%20on%20the%20Raspberry%20Pi)
* [ROS installation for Ubuntu](https://wiki.ros.org/kinetic/Installation/Ubuntu)
* [Fix missing GPG Keys](https://askubuntu.com/questions/127326/how-to-fix-missing-gpg-keys)
* [Adding swap to the raspberrypi](http://raspberrypimaker.com/adding-swap-to-the-raspberrypi/)
* [boost 1.58 deb package](https://launchpad.net/ubuntu/xenial/arm64/libboost-all-dev/1.58.0.1ubuntu1)
