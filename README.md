
---

# Custom Yocto Image for Raspberry Pi 

> - detailed instructions for building a custom Linux image for the Raspberry Pi using the Yocto Project. 
> - from setting up the Yocto environment to booting the Raspberry Pi with the custom-built image.

## Prerequisites
- A Linux-based development system with at least 50 GB of free space.
- Basic knowledge of Linux command line and Git.
- An SD card (8GB or larger recommended) and an SD card reader.
- A Raspberry Pi board (the steps here are tailored for Raspberry Pi 4).

## Step 1: Setting Up the Yocto Environment

### 1.1. Install Required Packages
Ensure your development system has necessary packages:
```bash
sudo apt-get update
sudo apt-get install gawk wget git-core diffstat unzip texinfo gcc-multilib \
     build-essential chrpath socat cpio python python3 python3-pip python3-pexpect \
     xz-utils debianutils iputils-ping libsdl1.2-dev xterm bmap-tools
```

### 1.2. Clone the Poky Repository
Clone the Yocto Project's Poky repository (dunfell branch recommended for stability):
```bash
git clone -b dunfell https://git.yoctoproject.org/git/poky.git
```
### 1.2. Clone the Openembedded-Core Repository
Clone the Openembedded-Core repository (dunfell branch recommended for stability):
```bash
git clone -b dunfell https://github.com/openembedded/openembedded-core.git
```

### 1.3. Clone Additional Layers
If your project needs additional layers (e.g., meta-raspberrypi), clone them:
```bash
git clone -b dunfell https://github.com/agherzan/meta-raspberrypi.git
```

### 1.4. Initialize the Build Environment
```bash
source poky/oe-init-build-env rpi-build 
```
Replace `rpi-build` with your desired build directory name. Mine is `rpi_yoct`

## Step 2: Configuring the Build

### 2.1. Local Configuration
Check available images to build: 
```bash
cd ..
ls meta-raspberrypi/conf/machine
 
include                  raspberrypi0.conf       raspberrypi3-64.conf  raspberrypi4.conf     raspberrypi.conf
raspberrypi0-2w-64.conf  raspberrypi0-wifi.conf  raspberrypi3.conf     raspberrypi-cm3.conf
raspberrypi0-2w.conf     raspberrypi2.conf       raspberrypi4-64.conf  raspberrypi-cm.conf
```
Edit `conf/local.conf` using a text editor (e.g., nano, vi) to set machine and other parameters:
```bash
vi rpi_yoct/conf/local.conf
```
Add or modify the following lines:
```plaintext
MACHINE ??= "raspberrypi4-64"
```
use "/" for search in vi 
```plaintext
/MACHINE # - search 
i # - insert
click Esc button to quit insert mode 
:wq # - write and quit 

```

You can also customize other parameters like image size, license settings, etc.

### 2.2. bblayers Configuration
Edit `rpi_yoct/conf/bblayers.conf` to add the paths of your custom layers:
```bash
vi rpi_yoct/conf/bblayers.conf
```
Add the paths under `BBLAYERS`, e.g.:
```plaintext
BBLAYERS ?= " \
  /path/to/poky/meta \
  /path/to/poky/meta-poky \
  /path/to/poky/meta-yocto-bsp \
  /path/to/meta-raspberrypi \
"
```
or directly insert layers path 
```bash
bitbake-layers add-layer ./meta-openembedded
bitbake-layers add-layer ./meta-raspberrypi
bitbake-layers add-layer ./meta-openembedded/meta-oe
bitbake-layers add-layer ./meta-openembedded/meta-python
bitbake-layers add-layer ./meta-openembedded/meta-networking
bitbake-layers add-layer ./meta-openembedded/meta-multimedia
```

check added layers 
```bash
bitbake-layers show-layers # command to print layers
NOTE: Starting bitbake server...
layer                 path                                      priority
==========================================================================
meta                  /home/amrith/yocto/poky/meta              5
meta-poky             /home/amrith/yocto/poky/meta-poky         5
meta-yocto-bsp        /home/amrith/yocto/poky/meta-yocto-bsp    5
meta-oe               /home/amrith/yocto/poky/build/meta-openembedded/meta-oe  5
meta-multimedia       /home/amrith/yocto/poky/build/meta-openembedded/meta-multimedia  5
meta-networking       /home/amrith/yocto/poky/build/meta-openembedded/meta-networking  5
meta-python           /home/amrith/yocto/poky/build/meta-openembedded/meta-python  5
meta-raspberrypi      /home/amrith/yocto/poky/build/meta-raspberrypi  9

```


## Step 3: Building the Image

By default it writes only writes image in `<image-name>.wic.bz2` fromat. You can use `bmaptool` to copy image into sdcard. 
 If you want to create `<image-name>.rpi-sdimg` then you need to set in `conf/local.conf`
```bash
vi rpi_yoct/conf/local.conf
i
IMAGE_FSTYPES = "wic.bz2 rpi-sdimg"
Esc
:wq
```

Inside `meta-raspberrypi/recipes-core/images` you should be able to see some recipes for rpi, we are building `rpi-test-image`

Run the bitbake command with your desired image name, in my case `rpi-test-image`:
```bash
bitbake rpi-test-image --runall=fetch
bitbake rpi-test-image
```

This process can take a few hours depending on your machine's capabilities.

## Step 4: Preparing the SD Card

Insert the SD card into your development system and identify its device file using `lsblk` (e.g., `/dev/sdc`). **Be very careful to choose the correct device, as you could overwrite important data.**

## Step 5: Copying the Image to the SD Card

### 5.1 Check the created image
```bash
ls rpi_yoct/tmp/deploy/images/raspberrypi4
```
you should see images like `<image-name>.wic.bz2` for bmaptool or `<image-name>.rpi-sdimg` for dd tool. In my case `rpi-test-image-raspberrypi4.tar.bz2`
### 5.1 Copy the Image Using `dd` or `bmaptool`
For `dd`:
```bash
sudo dd if=rpi_yoct/tmp/deploy/images/raspberrypi4/<image-name>.rpi-sdimg of=/dev/sdX bs=4M conv=fsync status=progress
```
For `bmaptool`:
```bash
sudo bmaptool copy rpi_yoct/tmp/deploy/images/raspberrypi4/<image-name>.wic.bz2 /dev/sdX
```
Replace `/dev/sdX` with your SD card device file. `bmaptool` can be more efficient. Both creates partitions in the sdcard defined by the image. 

## Step 6: Booting the Raspberry Pi

### 6.1 Insert the SD Card
Safely eject the SD card from your development system and insert it into your Raspberry Pi.

### 6.2 Connect Peripherals
Connect a monitor, keyboard, and mouse to the Raspberry Pi.

###

 6.3 Power On
Connect the power supply to boot the Raspberry Pi.

### 6.4 Initial Setup
Follow any on-screen instructions for initial setup and configuration.

## Troubleshooting
- Ensure all cables and SD card are properly connected.
- If the Raspberry Pi doesn't boot, try re-imaging the SD card.
- Check the `local.conf` and `bblayers.conf` files for correct paths and settings.
- For specific errors during the build process, consult the Yocto Project documentation and community forums.

---

**Note:** This README assumes a basic Yocto build for Raspberry Pi 4. Modify the steps according to your specific hardware and software requirements.