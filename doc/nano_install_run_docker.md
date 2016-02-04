# Install Docker for pcDuino3 Nano (based on Kernel 4.4 & Ubuntu 15.04)

By Xiaohai Li (haixiaolee@gmail.com)

## Before you start
First of all you need a workable Ubuntu Linux system. You can download the pre-build packages:

https://drive.google.com/open?id=0B4h2oLN0oRtVd3BMN1NhMTZKRGM

Or just build your own bootable image by following the system build-up guide here:

https://github.com/nightseas/pcduino3_nano_a20

## Check your Kernel features

I've prepared a Kernel config file for Nano that contained the basic required features needed by Docker. Fetch it from my GitHub and replace the '.config' file.

```sh
wget -c https://raw.githubusercontent.com/nightseas/pcduino3_nano_a20/master/kernel/pcduino3_nano_config
cp pcduino3_nano_config /path-to-kernel/.config
```

Also you can double check if the necessary features are enabled, with this script from Docker official GitHub:

```sh
cd /path-to-kernel
curl -L https://raw.githubusercontent.com/docker/docker/master/contrib/check-config.sh | /bin/bash /dev/stdin .config
```

Features in my config:

```
Generally Necessary:
- cgroup hierarchy: properly mounted [/sys/fs/cgroup]
- apparmor: enabled and tools installed
- CONFIG_NAMESPACES: enabled
- CONFIG_NET_NS: enabled
- CONFIG_PID_NS: enabled
- CONFIG_IPC_NS: enabled
- CONFIG_UTS_NS: enabled
- CONFIG_DEVPTS_MULTIPLE_INSTANCES: enabled
- CONFIG_CGROUPS: enabled
- CONFIG_CGROUP_CPUACCT: enabled
- CONFIG_CGROUP_DEVICE: enabled
- CONFIG_CGROUP_FREEZER: enabled
- CONFIG_CGROUP_SCHED: enabled
- CONFIG_CPUSETS: enabled
- CONFIG_MEMCG: enabled
- CONFIG_MACVLAN: enabled
- CONFIG_VETH: enabled
- CONFIG_BRIDGE: enabled (as module)
- CONFIG_BRIDGE_NETFILTER: enabled (as module)
- CONFIG_NF_NAT_IPV4: enabled
- CONFIG_IP_NF_FILTER: enabled
- CONFIG_IP_NF_TARGET_MASQUERADE: enabled
- CONFIG_NETFILTER_XT_MATCH_ADDRTYPE: enabled
- CONFIG_NETFILTER_XT_MATCH_CONNTRACK: enabled
- CONFIG_NF_NAT: enabled
- CONFIG_NF_NAT_NEEDED: enabled
- CONFIG_POSIX_MQUEUE: enabled

Optional Features:
- CONFIG_USER_NS: enabled
- CONFIG_SECCOMP: enabled
- CONFIG_MEMCG_KMEM: enabled
- CONFIG_MEMCG_SWAP: enabled
- CONFIG_MEMCG_SWAP_ENABLED: enabled
- CONFIG_BLK_CGROUP: enabled
- CONFIG_IOSCHED_CFQ: enabled
- CONFIG_BLK_DEV_THROTTLING: enabled
- CONFIG_CGROUP_PERF: enabled
- CONFIG_CGROUP_HUGETLB: missing
- CONFIG_NET_CLS_CGROUP: enabled (as module)
- CONFIG_CGROUP_NET_PRIO: enabled
- CONFIG_CFS_BANDWIDTH: enabled
- CONFIG_FAIR_GROUP_SCHED: enabled
- CONFIG_RT_GROUP_SCHED: enabled
- CONFIG_EXT3_FS: enabled
- CONFIG_EXT3_FS_XATTR: missing
- CONFIG_EXT3_FS_POSIX_ACL: enabled
- CONFIG_EXT3_FS_SECURITY: enabled
    (enable these ext3 configs if you are using ext3 as backing filesystem)
- CONFIG_EXT4_FS: enabled
- CONFIG_EXT4_FS_POSIX_ACL: enabled
- CONFIG_EXT4_FS_SECURITY: enabled
- Storage Drivers:
  - "aufs":
    - CONFIG_AUFS_FS: missing
  - "btrfs":
    - CONFIG_BTRFS_FS: enabled
  - "devicemapper":
    - CONFIG_BLK_DEV_DM: enabled
    - CONFIG_DM_THIN_PROVISIONING: enabled
  - "overlay":
    - CONFIG_OVERLAY_FS: enabled
  - "zfs":
    - /dev/zfs: present
    - zfs command: available
    - zpool command: available
```

After that re-compile the Kernel & modules and copy them to your SD card. If you have any problem with it, read the system build-up guide.

## Install and run Docker

Install the dependences needed by docker.io package:
```sh
udo apt-get install lxc aufs-tools cgroup-lite apparmor
```

While the apt source from port.ubuntu.com has an old version docker.io package (1.5.0) which is no longer supported by the Docker hub, we need to download and install 1.6.2 package from launchpad.net.
```sh
wget -c http://launchpadlibrarian.net/221352448/docker.io_1.6.2~dfsg1-1ubuntu4~15.04.1_armhf.deb
sudo dpkg -i docker.io_1.6.2~dfsg1-1ubuntu4~15.04.1_armhf.deb
```

Once the package is installed, verify it by these commands:

```sh
sudo docker version   #show version of docker.io and related packages
sudo docker info      #show system information 
```

The Docker running now. Then you can search and run available images. The binaries for pcDuino3 Nano are ARM hard float format (armhf), so only armhf Docker images can be used on Nano.

```sh
sudo docker search armhf
```

For example: to run a Ubuntu image, use the command:

```sh
sudo docker run -it armv7/armhf-ubuntu bash
```

Other Linux distros we can use:
```sh
armv7/armhf-debian
armv7/armhf-fedora
armbuild/opensuse-disk 
armbuild/archlinux-disk 
armbuild/busybox
resin/rpi-raspbian
```

A screenshot of running Docker and multi-distro on Nano:
![Dockor on Nano](https://raw.githubusercontent.com/nightseas/pcduino3_nano_a20/master/doc/nano_docker_multi_distro.jpg)

