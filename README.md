# Kernel 5.5 with MPTCP support

Guide to install MPTCP (from [here](https://www.multipath-tcp.org/)), Jorge Navarro-Ortiz (jorgenavarro@ugr.es), University of Granada, 2021.

Used for the [virtual testbed](https://github.com/jorgenavarroortiz/5g-clarity_testbed_v0) for the H2020 European Project [5G-CLARITY](https://www.5gclarity.com/).

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

## Kernel compilation (AMD x86-64 architecture)

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
