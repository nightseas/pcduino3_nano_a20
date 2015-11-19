# Buildup Linux 4.x & Lubuntu System on pcDuino3 Nano

By Xiaohai Li (haixiaoli@163.com)

## How it works

Well, it's my first try on both Kernel 4.x and pcDuino3 Nano. The early idea was from "Linux on ARM" of eewiki, which used scripts on Robert Nelson's GitHub to compile kernel for A20-OLinuXino-LIME platform. Instead of using scripts to automatically build the system, I’d like to show you each step and how to run Linux on an embedded processor.

 - My host platform: Xeon X5650 + Ubuntu MATE 15.10
 - Target platform: pcDuino3 Nano Lite (should work for Nano too)


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

You can also try other versions of GCC. Download link:
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
#(TBD)

## Fetch RootFS
#(TBD)

## Make A Bootable SD Card
Erase the partition table and boot information on the SD card (change sdX to your SD drive, like sda. If there is a MMC reader on your host, the drive name may also be mmcblkX):
```sh
sudo dd if=/dev/zero of=/dev/sdX bs=1M count=10
```

Format the disk with GParted tool. Here's a recommanded partition table:
| Partition Name 	| Format 	| Start Position 	| Size         	|
|----------------	|--------	|----------------	|--------------	|
| BOOT           	| fat    	| 1MB            	| 100MB        	|
| RootFS         	| ext4   	| 100MB          	| At least 4GB 	|

Write the bootloader generated from U-Boot compiling step to SD card.
```sh
sudo dd if=./u-boot/u-boot-sunxi-with-spl.bin of=/dev/sdX bs=1024 seek=8
```

Mount partitions:
```sh
sudo mkdir -p /media/boot/
sudo mkdir -p /media/rootfs/
sudo mount /dev/sdX1 /media/boot/
sudo mount /dev/sdX2 /media/rootfs/
```

Write the bootloader generated from U-Boot compiling step to SD card.
```sh
sudo dd if=./u-boot/u-boot-sunxi-with-spl.bin of=/dev/sdX bs=1024 seek=8
```

#(TBD)

### Have fine!