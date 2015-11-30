# Buildup Linux 4.x & Ubuntu System on pcDuino3 Nano

By Xiaohai Li (haixiaoli@163.com)

## How it works

Well, it's my first try on both Kernel 4.x and pcDuino3 Nano. The early idea was from "Linux on ARM" of eewiki, which used scripts on Robert Nelson's GitHub to compile kernel for A20-OLinuXino-LIME platform. Instead of using scripts to automatically build the system, I’d like to show you each step and how to run Linux on an embedded processor.

 - My host platform: Xeon X5650 + Ubuntu MATE 15.10
 - Target platform: pcDuino3 Nano Lite (should work for Nano too)

A Linux based software system for ARM usually contains these components:
 - Bootloader: U-Boot and a small boot running before it (xloader, boot0, FSBL... could be close-source)
 - Kernel: Now 4.x is the mainline version, and 2.x is still widely used. Also large number of modules, mostly drivers, just to reduce the size of kernel image.
 - Root File System: Can be built by busybox or buildroot tools, or fetched from Ubuntu, Debian, Fedora...
 - You also need a local or cross platform compiler (ARM Linux GCC) to compile all the softwares. 

## Cross-compiler: Linaro GCC

While we usually use x86 machines for development, to build executable programs on ARM/Power-PC/MIPS based processor you need a cross compiler. And 64-bit users also need to install 32-bit libraries for the pre-built compiler.

For Ubuntu 14.04 ~ 15.10 hosts:
```sh
sudo apt-get install libc6:i386 libstdc++6:i386 libncurses5:i386 zlib1g:i386
```

For Debian 7 ~ 9 hosts:
```sh
sudo dpkg --add-architecture i386
sudo apt-get install libc6:i386 libstdc++6:i386 libncurses5:i386 zlib1g:i386
```

Download & extract Linaro GCC tools:

```sh
wget -c https://releases.linaro.org/14.09/components/toolchain/binaries/gcc-linaro-arm-linux-gnueabihf-4.9-2014.09_linux.tar.xz
tar xf gcc-linaro-arm-linux-gnueabihf-4.9-2014.09_linux.tar.xz
```

Set environment variables to define the processor architecture and path to compiler:
```sh
cd gcc-linaro-arm-linux-gnueabihf-4.9-2014.09_linux
export ARCH=arm
export CROSS_COMPILE=${PWD}/bin/arm-linux-gnueabihf-
```

You can also try other versions of GCC. Download linck:
http://www.linaro.org/downloads/

## Build U-Boot

U-Boot, aka Das U-Boot, is short for the Universal Boot Loader. It's a small program to initialize the hardware and load operating system like Linux. For more information visit the website:
http://www.denx.de/wiki/U-Boot

Fetch the U-Boot source code and switch to the latest stable branch:
```sh
git clone git://git.denx.de/u-boot.git
cd u-boot/
git checkout v2015.10
```

Install libs before configuration & compilation:
```sh
sudo apt-get install libncurses5-dev libncursesw5-dev device-tree-compiler u-boot-tools
```

Configure U-Boot with the default settings of Nano, and you can also change the configuration by with GUI menu:
```sh
make distclean
make Linksprite_pcDuino3_Nano_defconfig
make menuconfig
```

Compile the U-Boot source code, with '-j' parameter you can choose how many thread you want to use to speed up the compilation on a multi-core processor. As I have 12-cores in my X5650:
```sh
make -j 12
```

After compiling you will get an binary file: u-boot-sunxi-with-spl.bin
That is the executable U-Boot image with Allwinner's first stage bootloader integrated.


## Build Linux Kernel

Fetch stable kernel source from kernel.org:
```sh
git clone https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git
```

Switch to the branch (kernel version) you want.

Kernel 4.3.y:
```sh
git checkout linux-4.3.y
```

Kernel 4.2.y:
```sh
git checkout linux-4.2.y
```

Kernel 4.1.y (it doesn't contain the dts file for Nano):
```sh
git checkout linux-4.1.y
```

Configure Kernel. The default configuration for Allwinner(sunxi) is a baseline but not good enough to use. Additional menuconfig is needed to enable basic drivers, such as USB keyboard/mouse or wireless dongle.
```sh
make sunxi_defconfig
make menuconfig
```

I've prepared a lite version for Nano, download the latest one:
```sh
wget -c https://github.com/nightseas/pcduino3_nano_a20/raw/master/kernel/pcduino3_nano_config
mv pcduino3_nano_config .config
make menuconfig
```

Compile Kernel (zImage), device tree & modules (add '-j' param to speed up):
```sh
make zImage
make dtbs
make modules
```

Create deploy folder and copy the compiled files:
```sh
cp arch/arm/boot/zImage deploy/
cp arch/arm/boot/dts/sun7i-a20-pcduino3-nano.dtb deploy/
make modules_install INSTALL_MOD_PATH=deploy/
```

## Create RootFS
Download pre-built Ubuntu 14.04 image from Linaro (here I choose the NANO image):

#### NANO
Description from Linaro: NANO provides a very minimum rootfs that's contains a fully function support for apt (the package management system used by Ubuntu). This image only provides console support, and should be used in case you want a fast and small image to develop and verify the kernel and system functionality. 

```sh
wget -c http://releases.linaro.org/14.10/ubuntu/trusty-images/nano/linaro-trusty-nano-20141024-684.tar.gz
sudo tar xfvp linaro-trusty-nano-20141024-684.tar.gz
```

#### DEVELOPER
Description from Linaro: Due demands for a minimum-console based image with extra developer tools integrated by default, the developer rootfs was created. This image is based on Nano, but also deploying a working toolchain, debuggers and also additional development and profiling tools needed by kernel and system developers. 

Download Link: 

```sh
http://releases.linaro.org/14.10/ubuntu/trusty-images/developer/linaro-trusty-developer-20141024-684.tar.gz
```

#### ALIP
Description from Linaro: ALIP is a small distribution used for bringing up ARM boards both by ARM internally and by various customers. Currently ALIP is based on LXDE (Lubuntu), with lightdm, X11 and chromium as the default graphic applications. 

Download Link: 

```sh
http://releases.linaro.org/14.10/ubuntu/trusty-images/alip/linaro-trusty-alip-20141024-684.tar.gz
```


Copy modules and firmware built with kernel to lib folder:
```sh
cd binary/
sudo cp -R <path to kernel>/deploy/lib/* lib/
```

Edit fstab to mount SD card when Nano boots up. 
```sh
sudo nano /media/rootfs/etc/fstab
```

Add these two lines:
```sh
/dev/mmcblk0p2  /      auto  errors=remount-ro  0  1
/dev/mmcblk0p1  /boot  auto  errors=remount-ro  0  1
```

Edit network configuration to enable Ethernet and DHCP:
```sh
sudo nano /media/rootfs/etc/network/interfaces
```

Add these lines:
```sh
auto lo
iface lo inet loopback
  
auto eth0
iface eth0 inet dhcp
```

## Make A Bootable SD Card
Erase the partition table and boot information on the SD card (change sdX to your SD drive, like sda. If there is a MMC reader on your host, the drive name may also be mmcblkX):

```sh
sudo dd if=/dev/zero of=/dev/sdX bs=1M count=10
```

Format the disk with GParted tool. Here's a recommended partition table:

|    Partition Name|Format    |    Start Position    |    Size    |
|    ---    |    ---    |    ---    |    ---    |
|    BOOT    |    fat    |    1MB    |    100MB    |
|    RootFS    |    ext4    |    100MB    |    At least 4GB    |

Write the bootloader to SD card.

```sh
sudo dd if=<path to u-boot>/u-boot-sunxi-with-spl.bin of=/dev/sdX bs=1024 seek=8
```

Mount partitions:

```sh
sudo mkdir -p /media/boot/
sudo mkdir -p /media/rootfs/
sudo mount /dev/sdX1 /media/boot/
sudo mount /dev/sdX2 /media/rootfs/
```

Copy Kernel image and device tree to boot partition of SD card.

```sh
sudo cp -R <path to kernel>/deploy/zImage /media/boot/
sudo cp -R <path to kernel>/deploy/sun7i-a20-pcduino3-nano.dtb /media/boot/
```

Create a settings file on boot partition:

```sh
sudo mkdir -p /media/boot/extlinux/
sudo nano /media/boot/extlinux/extlinux.conf
```

Add label in extlinux.conf to set the Kernel boot parameters:

```sh
label Linux 4.1.13
kernel ../zImage
append root=/dev/mmcblk0p2
fdtdir ../
```

Copy the root file system to rootfs partition of SD card.

```sh
sudo cp -RP <path to rootfs>/binary/* /media/rootfs/
```

The card is ready for use now, umount SD drive.

```sh
sudo umount /media/boot/
sudo umount /media/rootfs/
```

## Fisrt Boot Up

Insert the SD card to pcDuino3 Nano, and connect 5V/2A USB power, HDMI, Ethernet, USB keyboard/mouse to Nano. Then there will be boot information showed on the screen, first from U-Boot, and then Linux. Linux Kernel will load RootFS of Ubuntu at late boot phase.

Finally the software will auto-login as root user. But the default user is linaro with blank password. Change your password and update packages.

```sh
passwd linaro
#( input your password twice)
apt-get update 
apt-get upgrade
```

So far the Linux 4.x + Ubuntu system is running well, but with an ugly terminal interface if you choose NANO or DEVELOPER image. Let's install Lubuntu desktop (about 1.3GB) online with apt tool.

```sh
sudo apt-get install lubuntu-desktop
```

For other desktops:

```sh
sudo apt-get install ubuntu-desktop  # Ubuntu Unity
sudo apt-get install kubuntu-desktop  # Kubuntu
```

The installation will take hours. Get a cup of coffee and wait. After it's done, reboot pcDuino and ...

### Have fine!