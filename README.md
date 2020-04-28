WD My Cloud (Gen2) - wdmc-gen2 - Marvell ARMADA 375
===================================================

## For detailed Information have a look in the [Wiki](https://github.com/Zeik0s/wdmc-gen2/wiki)! Feel free to contribute!

* mainline kernel support tested with 4.18.9
	
	files needed for compiling:
	- device tree source -> [armada-375-wdmc-gen2.dts](https://github.com/Zeik0s/wdmc-gen2/blob/master/armada-375-wdmc-gen2.dts)
	- kernel config -> [kernel-4.18.9.config](https://github.com/Zeik0s/wdmc-gen2/blob/master/4.18.9/kernel.config)
	- kernel source code -> latest or stable [here](https://www.kernel.org)
	
	it's pretty easy doing this on your WDMC, but takes a long time.
	**Instructions for Cross-compiling [here](#Cross-Compile)**
	
	**Before you start, get sure to make a Backup of your current /dev/sda3 Partition! That's the partition that holds the system itself and all the configuration.**
	**And for the worst Case use/buy some Adapter or Dockingstation to restore the Backup when mounting the Drive!**

	- download kernel source from https://www.kernel.org/
	- extract the kernel archive
	- copy kernel-OLD_VERSION_NUMBER.config to linux-XXXXXXX/.config 
	- copy armada-375-wdmc-gen2.dts to linux-XXXXXXX/arch/arm/boot/dts/
	- ready to build the kernel
		```
		cd linux-XXXXXXXX
		make -j2 zImage
		make armada-375-wdmc-gen2.dtb
		cat arch/arm/boot/zImage arch/arm/boot/dts/armada-375-wdmc-gen2.dtb > zImage_and_dtb
		mkimage -A arm -O linux -T kernel -C none -a 0x00008000 -e 0x00008000 -n 'WDMC-Gen2' -d zImage_and_dtb uImage
		rm zImage_and_dtb
		make -j2 modules modules_install
		```
		copy uImage to /boot of your boot partition
		
		reboot and enjoy your new Kernel!


---
* Cross-Compile<a name="Cross-Compile"></a>:

	- Utilities required: 
		- https://sourceforge.net/projects/dsgpl/files/DSM%206.2.2%20Tool%20Chains/Marvell%20Armada%2038x%20Linux%203.10.105/armada38x-gcc493_glibc220_hard-GPL.txz/download
		- packages: build-essential libncurses5 u-boot-tools git bison flex
		- kernel source code (https://www.kernel.org)
		- all other WD Mycloud files that are mentioned in the upper lines.
		
	- download and extract new kernel source code
	- download and extract the Marvell Armada Toolchain (from Sourceforge)
	- enter the following: 
	```
	alias makehelp='make -j8 CROSS_COMPILE=/home/user/arm-unknown-linux-gnueabi/bin/arm-unknown-linux-gnueabi- ARCH=arm'
	```
	replace the following _/home/user_ directory with the directory where the toolchain is downloaded
	change the number after _-j_ according to your Number of Threads (Cores x 2) on your PC, this speeds up significantly the compiling process.
	then these commands follow:
	```
	makehelp zImage
	makehelp armada-375-wdmc-gen2.dtb
	mkdir -p ../kernel-bin/boot
	cat arch/arm/boot/zImage arch/arm/boot/dts/armada-375-wdmc-gen2.dtb > ../kernel-bin/zImage_and_dtb
	mkimage -A arm -O linux -T kernel -C none -a 0x00008000 -e 0x00008000 -n 'WDMC-Gen2' -d ../kernel-bin/zImage_and_dtb ../kernel-bin/boot/uImage
	rm ../kernel-bin/zImage_and_dtb
	makehelp modules
	makehelp INSTALL_MOD_PATH=../kernel-bin modules_install
	```
	This creates a folder called _kernel-bin_, where the Kernel and Boot-Code and all the compiled Kernel Modules are generated. Move the content of the folder to the root folder your Mycloud Device.
	
## Recovery/other Stuff

* uRamdisk modified (Original from AllesterFox)

	Can boot original firmware, debian from AllesterFox, alpine linux. It looks for a linux partition as 2nd on usb stick and boots from there.  If there is none, it boots from 3rd partition on SATA drive.  Copy to /boot on 1st partition of the usb stick or to 3rd partition of the harddrive.

* build-initramfs.sh

	builds a minimal initramfs.  Can boot from kernel commandline,
	usb-stick 2nd partition, hdd 3rd partition. Placed in /boot/uInitrd. rename it to uRamdisk.
		

* [alpine linux](https://alpinelinux.org/) diskless image

	- alpine-wdmc-gen2-3.6.2-armhf.tar.gz

		contains a complete bootable diskless image of alpine linux.
		modified to ignore "root=". "root" is not setable on
		WD My Cloud. And enabled SSH daemon, you can ssh to it after
		booting. *root account has no password.*
		Kernel 4.12 is included, but no modules.

	- alpine initramfs image with ssh (without password!)

* install
	- Use an USB 2.0 stick!
		They had the problem, that usb 3.0 sticks aren't always booting.
		If you have only an usb 3.0 stick, remove stick, turn wdmc2 on,
		when the blue light blinks insert usb stick fast, using this i
		could boot from usb 3.0 sick.

	- make a dos partition table (not gpt) on usb stick.
	- make a partition "W95 FAT32 (LBA)" numeric 0x0C on usb stick
	- make a FAT32 filesystem on usb stick.
	- extact alpine-wdmc-gen2-3.6.2-armhf.tar.gz to the root of the
	  usb stick.

	  You should have:
	  ```
	  -rwxr-xr-x    1 root     root            26 Jun 17 11:47 .alpine-release
	  -rwxr-xr-x    1 root     root          3427 Jan  1  1980 alpine.apkovl.tar.gz
	  drwxr-xr-x    3 root     root          4096 Jun 17 11:47 apks
	  drwxr-xr-x    5 root     root          4096 Jul  5 12:02 boot
          
	  ```
	  on the usb stick.
