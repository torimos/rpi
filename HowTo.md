# Kernel Compilation
_Based on http://lostindetails.com/blog/post/Compiling-a-kernel-module-for-the-raspberry-pi-2_

## Raspbian:
```
FIRMWARE_HASH=$(zgrep "* firmware as of" /usr/share/doc/raspberrypi-bootloader/changelog.Debian.gz | head -1 | awk '{ print $5 }')
KERNEL_HASH=$(wget https://raw.github.com/raspberrypi/firmware/$FIRMWARE_HASH/extra/git_hash -O -)
echo $KERNEL_HASH
mkdir ~/fbtft
```
## Ubuntu:
```
mkdir ~/rpi
cd ~/rpi
sudo apt-get install crossbuild-essential-armhf
git clone https://github.com/raspberrypi/linux
export CCPREFIX=/usr/bin/arm-linux-gnueabihf-
export KERNEL_SRC=/home/${USER}/rpi/linux/ 
export KERNEL=kernel
cd $KERNEL_SRC
git checkout $COPY_PAST_ABOVE_KERNEL_HASH
make ARCH=arm CROSS_COMPILE=${CCPREFIX} bcmrpi_defconfig
make ARCH=arm CROSS_COMPILE=${CCPREFIX} menuconfig
sudo make ARCH=arm CROSS_COMPILE=${CCPREFIX} zImage modules dtbs-j3
```
# Deployment
## Kernel and modules
```
mkdir /mnt/fat32
mkdir /mnt/ext4
sudo mount /dev/sdb1 /mnt/fat32
sudo mount /dev/sdb2 /mnt/ext4
sudo make ARCH=arm CROSS_COMPILE=${CCPREFIX} INSTALL_MOD_PATH=/mnt/ext4 modules_install
sudo cp arch/arm/boot/zImage /mnt/fat32/$KERNEL.img
sudo cp arch/arm/boot/dts/*.dtb /mnt/fat32/
sudo cp arch/arm/boot/dts/overlays/*.dtb* /mnt/fat32/overlays/
sudo cp arch/arm/boot/dts/overlays/README /mnt/fat32/overlays/
sudo umount /mnt/fat32
sudo umount /mnt/ext4
```
## Install and Test kernel modules on PI
```
sudo rmmod fbtft_device;sudo rmmod fb_ra8875;sudo rmmod fbtft;sudo rmmod spi_bcm2835
sudo modprobe spi_bcm2835;sudo modprobe fbtft;sudo modprobe fb_ra8875;sudo modprobe fbtft_device
```
## Specific kernel module
```
scp ~/rpi/linux/drivers/video/fbtft/*.ko pi@192.168.1.108:~/fbtft
```
## Install and Test Specific modules on PI
### install.sh:

```
#!/bin/bash
sudo rmmod fb_ra8875.ko -s
sudo rmmod fbtft_device.ko -s
sudo rmmod fbtft.ko -s
echo "Modules removed"
if [[ $1 == "1" ]]; then
        sudo insmod ~/fbtft/fbtft.ko
        sudo insmod ~/fbtft/fb_ra8875.ko
        sudo insmod ~/fbtft/fbtft_device.ko name="er_tftm070_48"
        echo "Modules installed"
fi
```
```
sudo rmmod fbtft_device.ko;sudo rmmod fb_ra8875.ko;sudo rmmod fbtft.ko
sudo insmod fbtft.ko;sudo insmod fb_ra8875.ko;sudo insmod fbtft_device.ko name="er_tftm070_48"
```
### bash:
```
./install.sh 1
./fbtest --fbdev /dev/fb1
```
# Testing FB
```
git clone https://git.kernel.org/pub/scm/linux/kernel/git/geert/fbtest.git
cd fbtest
make
./fbtest --fbdev /dev/fb1
```
# Autoload
```
sudo nano /etc/modules-load.d/fbtft.conf
spi-bcm2835
fbtft_device

sudo nano /etc/modprobe.d/fbtft.conf
options fbtft_device name=er_tftm070_48

sudo nano /boot/cmdline.txt
fbcon=map:11
```
