# https://blog.csdn.net/Naisu_kun/article/details/123142958
# http://www.xb6.cn/post/210.html
# FROM ubuntu@sha256:83f0c2a8d6f266d687d55b5cb1cb2201148eb7ac449e4202d9646b9083f1cee0
FROM ubuntu:22.04

ENV ARCH=arm
ENV CROSS_COMPILE=arm-buildroot-linux-gnueabi-
ENV PATH=$PATH:/f1c100s-sdk/buildroot-2022.02/output/host/bin
ENV DEBIAN_FRONTEND=noninteractive
RUN apt update
RUN apt install -y build-essential libusb-1.0-0-dev zlib1g-dev pkg-config swig libncurses-dev libncurses5-dev libssl-dev kpartx mtd-utils cpio python3-dev python2-dev python-is-python3 rsync bc file unzip git wget
RUN apt install -y sudo curl vim # debug tools
# RUN mkdir f1c100s-sdk

# buildroot
WORKDIR /f1c100s-sdk/
RUN wget https://buildroot.org/downloads/buildroot-2022.02.tar.xz && tar -xJf buildroot-2022.02.tar.xz
WORKDIR /f1c100s-sdk/buildroot-2022.02/
RUN wget https://raw.githubusercontent.com/wtwver/hardware/1252466f2198438b764aec3eccc61deb64bc781b/lichee-nano/.config && make

# uboot
RUN cd /f1c100s-sdk/ && git clone -b nano-lcd800480 --depth=1 https://github.com/Lichee-Pi/u-boot.git 
WORKDIR /f1c100s-sdk/u-boot
RUN make licheepi_nano_defconfig
RUN printf "setenv bootargs console=tty0 console=ttyS0,115200 panic=5 rootwait root=/dev/mmcblk0p2 rw\n\
load mmc 0:1 0x80008000 zImage\n\
load mmc 0:1 0x80C00000 suniv-f1c100s-licheepi-nano.dtb\n\
bootz 0x80008000 - 0x80C00000\n" > boot.cmd
RUN sed -i 's/#define CONFIG_BOOTCOMMAND.*$/#define CONFIG_BOOTCOMMAND "run distro_bootcmd"/g' include/configs/suniv.h 
RUN sed -i '20,22d' include/configs/suniv.h
RUN sed -i '634s/^/extern /' scripts/dtc/dtc-lexer.lex.c
RUN sed -i '321s/\\//' scripts/Makefile.lib
RUN make && tools/mkimage -A arm -C none -T script -d boot.cmd boot.scr

# Linux
WORKDIR /f1c100s-sdk/
RUN git clone -b nano-4.14-exp --depth=1 https://github.com/Lichee-Pi/linux.git
WORKDIR /f1c100s-sdk/linux/
RUN wget https://dl.sipeed.com/fileList/LICHEE/Nano/SDK/config && cp config .config

# cd /f1c100s-sdk/buildroot-2022.02/ && make menuconfig && make
# cd /f1c100s-sdk/u-boot make && tools/mkimage -A arm -C none -T script -d boot.cmd boot.scr
# cd /f1c100s-sdk/linux/ make menuconfig && make

CMD bash -c "read -n 1 -p '"'You need to press n many times. Confirm?'"' ; make; sed -i '634s/^/extern /' scripts/dtc/dtc-lexer.lex.c; make; " ; exit 0