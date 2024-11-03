---
layout: post
author: Madeye
title: Build the Linux kernel
tags:
- kernel build
---

Recently, I started reading the book Linux Device Drivers with the goal of
writing my own driver someday. Although the book is based on a rather old
kernel version (v2.6.10), I decided to follow its instructions to build and
run an example driver module on the latest kernel release, v6.11.5. To my
surprise, the instructions still worked perfectly!

Before I could perform the experiments in the book, I needed to build the
kernel myself, as the book mentions that building modules for v2.6.x requires
a complete built kernel tree on my system. However, the book does not provide
any instructions for building a kernel, so I documented the entire process here.

## Build and install the kernel
There are 5 steps:
- install the source
- install dependcies
- configure the kernel
- build the kernel
- install the kernel

### Install the source
- Download the latest release from [The Linux Kernel Archives](https://www.kernel.org/) as
  a compressed tar ball `linux-6.11.5.tar.xz`
- Uncompress it using tar `tar xf <path-to-downloaded-source>`
- `cd` into the kernel source directory and do `make mrproper` to make sure the directory is clean

Now, the source is properly installed on my system.

### Install the dependcies
Here is the list of dependcies I installed on a freshly installed Debian 12.7.0:
- make
- gcc
- flex
- bison
- libncurses-dev (needed to run `make menuconfig`)
- pkg-config
- libssl-dev (build error if not installed)
- libelf-dev (build error if not installed)

Note that there are other dependcies not listed, as they come bundled with the Debian distribution.
For a complete list, refer to the kernel documentation at `Documentation/process/changes.rst`.

### Configure the kernel
Before building the kernel, you need to configure it first. Since there are thousands of configuration
parameters, configuring them all from scratch is impratical. A more efficient approach is to copy the
configuration from the currently running kernel and use that as a starting point.
```shell
mkdir -p ${HOME}/kernel-build/v6.11.5
cp -v /boot/config-$(uname -r) ${HOME}/kernel-build/v6.11.5/.config
```

### Build the kernel
This step is straightforward but time consuming (it took me nearly 2 hours!). Run the following command:
```shell
make -j2 O=${HOME}/kernel-build/v6.11.5
```
To reduce the build time, I may need to further configure the kernel to exclude unnecessary components.

### Install the kernel
To install the new kernel, run another `make` command with `sudo`:
```shell
sudo make O=${HOME}/kernel-build/v6.11.5 modules_install install
```
The `modules_install` target is necessary if any kernel components are configured as modules.

If everything goes well, reboot the system, and the bootloader (***GRUB***) should select the new kernel
by default. Unfortunately, I encountered an issue: the `initramfs` image was too large and the reboot failed.
While the Debian kernel's initramfs is about 30 MB, the initramfs for my new kernel exceeded 400 MB! 
The `initramfs` of the Debian kernel is 30M, but the one of my new kernel is above 400M!
To resolve this, I edited the configuration file for `update-initramfs`, changing the `MODULES` option
from `most` to `dep`, and then regenerated the initramfs with `sudo update-initramfs -u`.

#### Summary of the Kernel Installation Process:
- `make install` triggers the `/scripts/install.sh` script
- On my system, `install.sh` calls `installkernel`, a Debian-provided kernel installation script
- `installkenel` executes all scripts in `/etc/kernel/postinst.d/` using `run-parts`, which runs:
  - `run-parts --verbose --exit-on-error --arg="$ver" --arg="$dir/$img_dest-$ver" /etc/kernel/postinst.d`
  - the first script, `initramfs-tools`, has a command line at its last line:
    `update-initramfs -c -k ${version}" "${bootopt}" >&2`
  - the second script, `zz-update-grub`, runs `exec update-grub`, which updates the GRUB boot menu to
    set the new kernel as the default

The above sequence produces the following installation output:
```shell
  INSTALL /boot
run-parts: executing /etc/kernel/postinst.d/initramfs-tools 6.11.5 /boot/vmlinuz-6.11.5
update-initramfs: Generating /boot/initrd.img-6.11.5
W: Possible missing firmware /lib/firmware/i915/mtl_gsc_1.bin for module i915
W: Possible missing firmware /lib/firmware/i915/dg2_huc_gsc.bin for module i915
W: Possible missing firmware /lib/firmware/i915/mtl_huc_gsc.bin for module i915
W: Possible missing firmware /lib/firmware/i915/mtl_guc_70.bin for module i915
W: Possible missing firmware /lib/firmware/i915/bmg_dmc.bin for module i915
W: Possible missing firmware /lib/firmware/i915/xe2lpd_dmc.bin for module i915
run-parts: executing /etc/kernel/postinst.d/zz-update-grub 6.11.5 /boot/vmlinuz-6.11.5
Generating grub configuration file ...
Found background image: /usr/share/images/desktop-base/desktop-grub.png
Found linux image: /boot/vmlinuz-6.11.5
Found initrd image: /boot/initrd.img-6.11.5
Found linux image: /boot/vmlinuz-6.1.0-26-amd64
Found initrd image: /boot/initrd.img-6.1.0-26-amd64
Found linux image: /boot/vmlinuz-6.1.0-25-amd64
Found initrd image: /boot/initrd.img-6.1.0-25-amd64
Warning: os-prober will not be executed to detect other bootable partitions.
Systems on them will not be added to the GRUB boot configuration.
Check GRUB_DISABLE_OS_PROBER documentation entry.
done
```
