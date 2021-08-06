# Kernel 5.5 with MPTCP support

Guide to install MPTCP with Linux kernel 5.5 for PCs (x86-64 architecture) (from [here](https://www.multipath-tcp.org/)), Jorge Navarro-Ortiz (jorgenavarro@ugr.es), University of Granada, 2021.

Used for the [virtual testbed](https://github.com/jorgenavarroortiz/5g-clarity_testbed_v0) for the H2020 European Project [5G-CLARITY](https://www.5gclarity.com/). We had some issues with the drivers for the Intel AX201 WiFi6 card, which were solved with Linux kernel 5.5. So this is the main reason to use this kernel (and not others available at [here](http://multipath-tcp.org/patches/)).

You can generate a Linux kernel 5.5 with MPTCP support by following these steps.

1) Download Linux kernel 5.5:

```
wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.5.tar.gz
cd linux
```

2) Apply the patch:

```
patch -p1 < ../mptcp-v5.5-9d3f35bb152a.fromgit.patch
```

This patch is based on the one for kernel 5.4 ([here](http://multipath-tcp.org/patches/mptcp-v5.4-d8e3bd88da5f.patch)), modified to work with kernel 5.5.

Alternatively, you can download the source code from this repo, which has this patch already applied (no need to apply it again).

## Kernel compilation (x86-64 architecture)

Please execute the following commands:

```
sudo make clean
make -j`nproc`
make -j`nproc` bindeb-pkg
```

This will generate `linux-headers-*.deb`, `linux-image-*.deb` and `linux-libc-*.deb` files. Save these files.

## Kernel installation

Please execute the following commands:

```
sudo dpkg -i *.deb
sudo update-grub
```

If you have several kernel versions, you may need to update the file `/etc/default/grub`. For that purpuse, you may use the `grub-menu.sh` script from https://askubuntu.com/questions/1019213/display-grub-menu-and-options-without-rebooting:

![image](https://user-images.githubusercontent.com/17797704/122587125-c1fc3780-d05d-11eb-8dc3-e6f73504d854.png)

Then, edit the file `/etc/default/grub` and modify the value assigned to `GRUB_DEFAULT` (e.g. `GRUB_DEFAULT="1>2"`). Finally, update the grub (`sudo update-grub`) and reboot.

## Create/apply a patch for MPTCP

To create a patch with the current MPTCP implementation:
 - Rename the directory with a clean kernel to 'a'
 - Rename the directory with the modified kernel to 'b'
 - From the previous directory, execute "diff -Naur a b > mptcp-vXXX.patch"

If you want to submit the patch to the MPTCP mailing list, you have to follow the instructions from [here](https://multipath-tcp.org/pmwiki.php/Developer/SubmitAPatch).

To apply the patch:
 - The patch file shall be in the parent directory of the directory with a clean kernel.
 - Enter the directory of the clean kernel.
 - Execute "patch -p1 < ../mptcp-vXXX.patch"


# Kernel 5.5 with MPTCP support and WRR for Raspberry Pi 4

Guide to install MPTCP with Linux kernel 5.5 for Raspberry Pi 4, Jorge Navarro-Ortiz (jorgenavarro@ugr.es), University of Granada, 2021.

Used for the [virtual testbed](https://github.com/jorgenavarroortiz/5g-clarity_testbed_v0) for the H2020 European Project [5G-CLARITY](https://www.5gclarity.com/).

You can generate a Linux kernel 5.5 with MPTCP support for Raspberry Pi 4 by following these steps.

```
git clone --depth=1 --branch rpi-5.5.y https://github.com/raspberrypi/linux
cd linux

patch -p1 < ../mptcp-v5.5_RPi4.patch
```

This patch is based on the one for kernel 5.4 ([here](http://multipath-tcp.org/patches/mptcp-v5.4-d8e3bd88da5f.patch)), modified to work with kernel 5.5.

## Kernel compilation for RPi4 (ARMv8 architecture)

Execute the following commands:

```
KERNEL=kernel8

make clean
make distclean

make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- bcm2711_defconfig
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- menuconfig

# Change CONFIG_LOCALVERSION="-mptcp" (or other string) in .config

make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- Image modules dtbs LOCALVERSION=-v8-mptcp -j 6
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- bindeb-pkg LOCALVERSION=-v8-mptcp -j 6

mkdir ../boot
mkdir ../boot/overlays

sudo cp arch/arm64/boot/dts/broadcom/*.dtb ../boot/
sudo cp arch/arm64/boot/dts/overlays/*.dtb* ../boot/overlays/
sudo cp arch/arm64/boot/dts/overlays/README ../boot/overlays/
sudo cp arch/arm64/boot/Image ../boot/$KERNEL.img

cd ..
tar cvfz boot_rpi_kernel55_mptcp.tar.gz boot
```

Copy linux-image*.deb, linux-headers*.deb, linux-libc*.deb and boot_rpi_*.tar.gz to a USB drive.

## Kernel installation

We assume that we have copied the *.deb files and the boot_rpi_*.tar.gz to e.g. `$HOME/tmp`. Then, execute:

```
sudo dpkg -i *.deb
tar xvfz boot_rpi_kernel55_mptcp.tar.gz
sudo cp -R boot /boot
sudo reboot
```

Please make a full backup of your RPi's card to avoid loosing data. In addition, please make sure that you copy the required files to the boot directory. If not, your RPi may get bricked!

## Create/apply a patch for MPTCP

To create a patch with the current MPTCP implementation:
 - Rename the directory with a clean kernel to 'a'
 - Rename the directory with the modified kernel to 'b'
 - From the previous directory, execute "diff -Naur a b > mptcp-vXXX.patch"

If you want to submit the patch to the MPTCP mailing list, you have to follow the instructions from [here](https://multipath-tcp.org/pmwiki.php/Developer/SubmitAPatch).

To apply the patch:
 - The patch file shall be in the parent directory of the directory with a clean kernel.
 - Enter the directory of the clean kernel.
 - Execute "patch -p1 < ../mptcp-vXXX.patch"
