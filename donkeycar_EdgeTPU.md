# Donkey Car on the EdgeTPU Dev Board

## Overview
The [Donkey Car project](https://docs.donkeycar.com/) is a fun introduction to embedded AI, autonomous vehicles, and robotics. It has been designed
around the Raspberry Pi 3+ and select RC car bodies. In this guide, we explore swapping in the coral EdgeTPU dev board
in place of the pi to take advantage of fast inference with the EdgeTPU tflite libraries. 


## Setup
* Flash Mendel linux using the Flash from u-boot on an SD card instructions as this is the most reliable way to ensure
  proper imaging of the dev board without interruption -
  (https://coral.withgoogle.com/docs/dev-board/reflash/#flash-from-u-boot-on-an-sd-card). Make sure to use a usb-c cable
  that can be used for syncing when connecting to the OTG port. 

* NOTE: Do not disconnect the dev board without shutting it down via the terminal. This can corrupt the board's image. 
  ```
  sudo shutdown now
  ```

* To configure internet, use the nmtui tool as the terminal has issues with text wrapping around incorrectly when
  connected via serial during setup. To verify connection, run
  ```
  nmcli connection show
  ```
* On your host machine, install mdt tool to easily ssh and scp data to the dev board.
  ```
  pip3 install --user mendel-development-tool
  ```
  To ssh for the first time, you'll need to be connected to the board via the OTG port using the usb-c cable so that the
  mdt tool can set up ssh keys appropriately. After this, you can ssh into the dev board without being physically
  connected to it using:
  ```
  mdt shell
  ```
  To scp anything: ``` mdt push /path/to/file```
  More on the mdt tool: https://coral.withgoogle.com/docs/dev-board/mdt/


## Experimenting with Donkeycar usecase
The Mendel linux distro is super minimal, so expect to have some libraries missing that we'll need to install. I
recommend directly working on the board hooked up to a monitor and keyboard instead of installing over an ssh connection
as the mdt tool will frequently shutdown a shell for inactivity.
To start, make sure the system is updated:
```
sudo apt-get update
sudo apt-get upgrade
```

At the moment, we didn't 
### OpenCV installation 
We'll need to rely on OpenCV to capture image data from a camera device. We'll install this first as it will install a
few low level libraries we'll need later.
From this medium post (https://medium.com/@balaji_85683/installing-opencv-4-0-on-google-coral-dev-board-5c3a69d7f52f)

The board has limited space, so we can mount extra storage via the microSD port. Attach a blank microSD card, format it
for ext4 and mount:
```
#check for disk name
$ lsblk
#should look like mntblk1 with the appropriate size of your microSD card
#format to ext4
$ sudo mkfs -t ext4 /dev/mntblk1
```

Create a 1G temporary swap:
```
$ sudo allocate -l 1G /swapfile
$ sudo chmod 600 /swapfile
$ sudo mkswap /swapfile
$ sudo swapon /swapfile
```

Install some low level dependencies:
```
$ sudo apt-get install build-essential cmake unzip pkg-config
$ sudo apt-get install libjpeg-dev libpng-dev libtiff-dev
$ sudo apt-get install libavcodec-dev libavformat-dev libswscale-dev libv4l-dev
$ sudo apt-get install libxvidcore-dev libx264-dev
$ sudo apt-get install libgtk-3-dev
$ sudo apt-get install libatlas-base-dev gfortran
$ sudo apt-get install python3-dev build-essential
$ sudo apt-get install python3-setuptools python3-pip python3-wheel 
```

Mount the disk we formated earlier:
```
sudo mount /dev/mntblk1 /mnt
```

Download OpenCV and contrib modules
```
$ cd /mnt
$ sudo wget -O opencv.zip https://github.com/opencv/opencv/archive/4.0.0.zip
$ sudo wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/4.0.0.zip
$ sudo unzip opencv.zip
$ sudo unzip opencv_contrib.zip
$ sudo mv opencv-4.0.0 opencv
$ sudo mv opencv_contrib-4.0.0 opencv_contrib
```

Numpy should be installed already, but in case it's not, install:
```
pip3 install numpy
```

Go into the OpenCV directory and cmake
```
cd /mnt/opencv
sudo mkdir build
cd build
```
Once in the build directory:
```
sudo cmake -D CMAKE_BUILD_TYPE=RELEASE \
 -D CMAKE_INSTALL_PREFIX=/usr/local \
 -D INSTALL_PYTHON_EXAMPLES=ON \
 -D INSTALL_C_EXAMPLES=OFF \
 -D OPENCV_ENABLE_NONFREE=ON \
 -D OPENCV_EXTRA_MODULES_PATH=/mnt/opencv_contrib/modules \
 -D PYTHON_EXECUTABLE=~/.virtualenvs/cv/bin/python \
 -D ENABLE_FAST_MATH=1 \
 -D ENABLE_NEON=ON -D WITH_LIBV4L=ON \
 -D WITH_V4L=ON \
 -D BUILD_EXAMPLES=ON ..
```

This should take a few minutes. Then make:
```
sudo make
```

This step should take +4 hours. 
If all goes well, you can finally install opencv
```
$ sudo make install
$ sudo ldconfig
```

Rename the compiled opencv .so file to import it correctly and link it to vm:
```
$ ls /usr/local/python/cv2/python-3.5
cv2.cpython-35m-aarch64-linux-gnu.so

$ sudo mv /usr/local/python/cv2/python-3.5/cv2.cpython-35m-aarch64-linux-gnu.so /usr/local/python/cv2/python-3.5/cv2.so
$ cd ~/litterbug/lib/python3.5/site-packages
$ ln -s /usr/local/python/cv2/python-3.5/cv2.so cv2.so
```

Test it!:
```
$ python3
>>> import cv2
>>> cv2.__version__
'4.0.0'
>>> quit()
```

You can now safely remove the microsd card if you wish. (I'd keep it on for extra storage.)

### Installing Donkeycar library
As mentioned earlier, donkeycar assumes that you are using a raspberry pi as they provide a raspbian image they want you to use to run and control the rc car. Since we will not be running raspbian from a microSD on this platform, we'll install the library they use for the host linux machine. This version does not include the raspberry pi dependencies, which will make it easier to modify to use the edgeTPU board. 

Install some dependencies globally:
```
$ sudo apt-get install cython3
$ sudo apt-get install libhdf5-dev
$ sudo apt-get install python-h5py
$ sudo apt-get install git
$ sudo apt-get install vim
```

Create a vm:
```
$ python3 -m venv --system-site-packages litterbug
$ source litterbug/bin/activate
```

Clone the donkeycar repo from autorope/donkeycar (https://github.com/autorope/donkeycar)
```
(litterbug) $ git clone https://github.com/autorope/donkeycar.git
(litterbug) $ cd donkecar
```
Remove the h5py dependency from the setup.py file since we've installed it globally already and install via pip will fail here
```
#in donkeycar/ directory
$ vim setup.py
```
Should go from this:
```
      install_requires=['numpy',
                        'pillow',
                        'docopt',
                        'tornado==4.5.3',
                        'requests',
                        'h5py',
                        'python-socketio',
                        'flask',
                        'eventlet',
                        'moviepy',
                        'pandas',
],
```
To this:
```
      install_requires=['numpy',
                        'pillow',
                        'docopt',
                        'tornado==4.5.3',
                        'requests',
                        'python-socketio',
                        'flask',
                        'eventlet',
                        'moviepy',
                        'pandas',
],
```

Now run:
```
#in the donkeycar/ directory
$ pip install -e .
```
This should be a few minutes installing all the python dependencies

Scp the car files copied from the original donkeycar raspbian image (included with this readme)
```
$ mdt push /path/to/mycar
```


Installed the Adafruit PCA9685 servo shield library via pip (https://github.com/adafruit/Adafruit_Python_PCA9685):
```
(litterbug) $ pip install adafruit-pca9685
```

Since this library was designed to work with raspberry pi and beaglebone, it'll error out on not detecting the correct
I2C interface for an unknown device. Edit the I2C file of the Adafruit_GPIO library:
```
$ vim /home/mendel/litterbug/lib/python3.5/site-packages/Adafruit_GPIO/I2C.py
```
Change the get_default_bus function around line 55 to go from this:
```
def get_default_bus():
    """Return the default bus number based on the device platform.  For a
    Raspberry Pi either bus 0 or 1 (based on the Pi revision) will be returned.
    For a Beaglebone Black the first user accessible bus, 1, will be returned.
    """
    plat = Platform.platform_detect()
    if plat == Platform.RASPBERRY_PI:
        [...]
    elif plat == Platform.BEAGLEBONE_BLACK:
        [...]
    else:
		raise RuntimeError('Could not determine default I2C bus for platform.')
```

To this:
```
def get_default_bus():
    """Return the default bus number based on the device platform.  For a
    Raspberry Pi either bus 0 or 1 (based on the Pi revision) will be returned.
    For a Beaglebone Black the first user accessible bus, 1, will be returned.
    """
    plat = Platform.platform_detect()
    if plat == Platform.RASPBERRY_PI:
        [...]
    elif plat == Platform.BEAGLEBONE_BLACK:
        [...]
    else:
        return 1
```
 
You can run the test script here
(https://github.com/adafruit/Adafruit_Python_PCA9685/blob/master/examples/simpletest.py) to see if you can manipulate
servos correctly. Make sure your rc car is on and all the wiring is correct (exact wiring as a raspberry pi setup)
