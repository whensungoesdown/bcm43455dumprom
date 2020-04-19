# bcm43455dumprom

My Raspberry Pi 4
Linux ubuntu 5.3.0-1017-raspi2 #19~18.04.1-Ubuntu SMP Fri Jan 17 11:14:07 UTC 2020 aarch64 aarch64 aarch64 GNU/Linux

To compile a kernel module alone, the module's source code better comes from the source tree of the current kernel.
Otherwise, the module code may differ a lot.

The kernel may expect some function from the module, but the module code doesn't have it. Then it won't pass
modpost.

Don't need to compile the whole kernel source, at least, the header files are needed.

example makefile for compiling a kernel module

obj-m := hello.o
KDIR := /lib/modules/$(shell uname -r)/build
PWD := $(shell pwd)
default:
        $(MAKE) -C $(KDIR) SUBDIRS=$(PWD) modules 


lib/moudules/$(shell unmame -r)/build is a link to the header directory.

u@ubuntu:~/prjs/bcm43455dumprom$ ls -l /lib/modules/$(uname -r)/build
lrwxrwxrwx 1 root root 40 Jan 17 10:40 /lib/modules/5.3.0-1017-raspi2/build -> /usr/src/linux-headers-5.3.0-1017-raspi2

In header directory, there is the .config file that records all the choices that the current running kernel made.

This is important, otherwise the driver module compiled won't match with the kernel.

For example, in the brcmfmac/Makefile,

brcmfmac-$(CONFIG_BRCMFMAC_PROTO_BCDC) += \
		bcdc.o \
		fwsignal.o
brcmfmac-$(CONFIG_BRCMFMAC_PROTO_MSGBUF) += \
		commonring.o \
		flowring.o \
		msgbuf.o
brcmfmac-$(CONFIG_BRCMFMAC_SDIO) += \
		sdio.o \
		bcmsdh.o

You don't need to define those  CONFIG_XX yourself. It's all in the .config.

CONFIG_BRCMUTIL=m
CONFIG_BRCMSMAC=m
CONFIG_BRCMFMAC=m
CONFIG_BRCMFMAC_PROTO_BCDC=y
CONFIG_BRCMFMAC_PROTO_MSGBUF=y
CONFIG_BRCMFMAC_SDIO=y
CONFIG_BRCMFMAC_USB=y
CONFIG_BRCMFMAC_PCIE=y
CONFIG_BRCM_TRACING=y
CONFIG_BRCMDBG=y
CONFIG_WLAN_VENDOR_CISCO=y
 
