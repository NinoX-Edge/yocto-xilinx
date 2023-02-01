# Use Yocto to build xilinx-layer

## Tools usage
* git
* bitbake

##  Device Check
* board : kv260
* xilinx-layers : rel-v2022.1
* Poky : honister

## Steps

1. Get meta-layer from github
2. Set config file for building
3. Build
4. Package to SD-Card

## Get meta-layer from github
In the beginning, we build yocto by poky. Now, we want to append new layers for our linux. We use xilinx layers from xilinx website,and those layers are used in petalinux.
(Total layers in petalinux)
https://support.xilinx.com/s/article/000034113?language=en_US
### folder structure
*  poky
 ->build (after source ~/poky/oe-init-build-env)
* source
 ->require layers
 ### require layer:

| layer | website | branch |
| -------- | -------- | -------- |
| poky     | https://github.com/Xilinx/poky/tree/rel-v2022.1     | rel-v2022.1     |
|  meta-openembedded      | https://github.com/Xilinx/meta-openembedded/tree/rel-v2022.1     | rel-v2022.1     || -------- | -------- | -------- |
| openamp     | https://github.com/Xilinx/meta-openamp/tree/rel-v2022.1    | rel-v2022.1     || -------- | -------- | -------- |
| clang     | https://github.com/Xilinx/meta-clang/tree/rel-v2022.1     | rel-v2022.1     || -------- | -------- | -------- |
| browser     | https://github.com/Xilinx/meta-browser/tree/rel-v2022.1    | rel-v2022.1     || -------- | -------- | -------- |
| petalinux     | https://github.com/Xilinx/meta-petalinux/tree/rel-v2022.1     | rel-v2022.1     || -------- | -------- | -------- |
| xilinx     | https://github.com/Xilinx/meta-xilinx/tree/rel-v2022.1     | rel-v2022.1     || -------- | -------- | -------- |
| xilinx-tool     | https://github.com/Xilinx/meta-xilinx-tools/tree/rel-v2022.1    | rel-v2022.1     |
|qt5|https://github.com/meta-qt5/meta-qt5/tree/honister|honister
|virtualization|https://github.com/Xilinx/meta-virtualization/tree/rel-v2022.1|rel-v2022.1|
|jupyter|https://github.com/Xilinx/meta-jupyter/tree/rel-v2022.1|rel-v2022.1|
|ros|https://github.com/Xilinx/meta-ros|rel-v2022.1|
## Set config file for building
If we want to append new layers, we need to modify bblayer.conf.
Modify your ~/poky/build/conf/bblayer.conf
```
# POKY_BBLAYERS_CONF_VERSION is increased each time build/conf/bblayers.conf
# changes incompatibly   
POKY_BBLAYERS_CONF_VERSION = "2"

BBPATH = "${TOPDIR}"
BBFILES ?= ""
SOURCEPATH = "/home/chris/kv260-yocto/source"

BBLAYERS ?= " \
  ${SOURCEPATH}/../poky/meta \
  ${SOURCEPATH}/../poky/meta-poky \
  ${SOURCEPATH}/meta-openembedded/meta-perl \
  ${SOURCEPATH}/meta-openembedded/meta-python \
  ${SOURCEPATH}/meta-openembedded/meta-filesystems \
  ${SOURCEPATH}/meta-openembedded/meta-gnome \
  ${SOURCEPATH}/meta-openembedded/meta-multimedia \
  ${SOURCEPATH}/meta-openembedded/meta-networking \
  ${SOURCEPATH}/meta-openembedded/meta-webserver \
  ${SOURCEPATH}/meta-openembedded/meta-xfce \
  ${SOURCEPATH}/meta-openembedded/meta-initramfs \
  ${SOURCEPATH}/meta-openembedded/meta-oe \
  ${SOURCEPATH}/meta-clang \
  ${SOURCEPATH}/meta-browser/meta-chromium \
  ${SOURCEPATH}/meta-xilinx/meta-microblaze \
  ${SOURCEPATH}/meta-xilinx/meta-xilinx-core \
  ${SOURCEPATH}/meta-xilinx/meta-xilinx-bsp \
  ${SOURCEPATH}/meta-xilinx/meta-xilinx-pynq \
  ${SOURCEPATH}/meta-xilinx/meta-xilinx-contrib \
  ${SOURCEPATH}/meta-xilinx/meta-xilinx-standalone \
  ${SOURCEPATH}/meta-xilinx-tools \
  ${SOURCEPATH}/meta-petalinux \
  ${SOURCEPATH}/meta-qt5 \
  ${SOURCEPATH}/meta-openamp \
  ${SOURCEPATH}/meta-ros/meta-ros2 \
  ${SOURCEPATH}/meta-ros/meta-ros2-humble \
  ${SOURCEPATH}/meta-ros/meta-ros-common \
  ${SOURCEPATH}/meta-virtualization \
  ${SOURCEPATH}/meta-jupyter \
  "
```
add those command in your ~/poky/build/conf/local.conf
```
DISTRO ?= "petalinux"
MACHINE ??= "zynqmp-generic"
IMAGE_FSTYPES = "wic.gz"
INHERIT += "rm_work"
```

## Build 
use bitbake to build your linux.
If you see bitbake command not found, go to ~/poky and source oe-init-build-env.
```
$ cd ~/poky/build
$ bitbake petalinux-image-minimal
```
## Package to SD-Card
### Two options.
we have two choices to finish our work.
First, We use Gparted to split sd-card to two partitions(FAT32 and Ext4).The other one is use balenaEtcher to load our sd-card. Then replace files in this sd-card-boot.

https://github.com/JohnsonOUO/yocto-kv260
**First One Option**
1. use Gparted to parition sdcard to two sections.
Partition is like |FAT32|------Ext4-------|

FAT32 : boot src, system.dtb, Image, ramdisk.cpio.gz.u-boot.
* boot.src and system.dtb is from github
* Image and ramdisk.cpio.gz.u-boot, we need to
```
$ cp ~/poky/build/tmp/deploy/image/zynqmp-generic/Image SD-Card-partition-1/Image
$ cp ~/poky/build/tmp/deploy/image/zynqmp-generic/petalinux-...-generic.cpio.gz.u-boot SD-Card-partition-1/ramdisk.cpio.gz.u-boot
```
Ext4 :
unpackage tar.gz to your sd-card-partition-2
```
$ tar xvf ~/poky/build/tmp/deploy/image/zynqmp-generic/petalinux-...-generic.tar.gz SD-Card-partition-2
```
**Second One Option**
Use balenaEtcher load wic.gz to sdcard
(poky/build/tmp/deploy/image/YOUR_MACHINE/***.wic.gz)
Then replace boot src and system.dtb from your sd-card boot folder.


## Configuration
帳號：root 
密碼：nino123x

網路設定：
/etc/network/interfaces 更改ip 以及gateway 等等

無線網路：
/etc/wpa_supplicant/wpa_supplicant下更改帳號密碼即可
Proxmox網頁：
https://ip:8006

Talos k8s部署：
（如果node 關機打不開 
cd ~/Edge-Cloud/01-…/
nano var.ft 先把num_master跟num_worker設為0 在該目錄下執行terraform apply（yes)
在將剛剛num設回1（master)2(worker) 再執行terraform apply （yes)跑好後 
cd ~
./bootstrap.sh （該檔案的最後兩行 執行一次即可）
(選擇overwrite) 即可使用kubectl 去做操作

