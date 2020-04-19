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
 
---

# How it works

The brcmfmac generate 3 new file in debugfs, original_rom, rom, ram.

original_rom is save before loading the firmware.

rom and ram are the current conent in memory.

e.g.

root@ubuntu:/sys/kernel/debug/ieee80211/phy0# cat original_rom | head -n 10  
ROM before firmware loaded, start at 0x0, size: 0xa0000 bytes  

00000000  00 F0 3E B8 00 00 00 00  00 00 00 00 00 00 00 00  
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  
00000020  41 EA 00 03 13 43 9B 07  30 B5 10 D1 0C 68 03 68  
00000030  63 40 13 60 4C 68 43 68  63 40 53 60 8C 68 83 68  
00000040  63 40 93 60 C9 68 C3 68  4B 40 D3 60 30 BD 00 23  
00000050  CD 5C C4 5C 6C 40 D4 54  01 33 10 2B F8 D1 30 BD  
00000060  2D E9 F0 4F 8F B0 01 24  0E AD 07 46 0E 46 91 46  
00000070  11 46 0D F1 19 00 0D 22  05 F8 20 4D 05 93 DD F8  
root@ubuntu:/sys/kernel/debug/ieee80211/phy0# cat rom | head -n 10  
ROM start at 0x0, size: 0xa0000 bytes  

00000000  98 F1 3E B8 99 F1 E4 BA  99 F1 F0 BA 99 F1 FC BA  
00000010  99 F1 0B BB 99 F1 1A BB  99 F1 29 BB 99 F1 38 BB  
00000020  41 EA 00 03 13 43 9B 07  30 B5 10 D1 0C 68 03 68  
00000030  63 40 13 60 4C 68 43 68  63 40 53 60 8C 68 83 68  
00000040  63 40 93 60 C9 68 C3 68  4B 40 D3 60 30 BD 00 23  
00000050  CD 5C C4 5C 6C 40 D4 54  01 33 10 2B F8 D1 30 BD  
00000060  2D E9 F0 4F 8F B0 01 24  0E AD 07 46 0E 46 91 46  
00000070  11 46 0D F1 19 00 0D 22  05 F8 20 4D 05 93 DD F8  
root@ubuntu:/sys/kernel/debug/ieee80211/phy0# cat ram | head -n 10  
RAM start at 0x198000, size: 0xc8000 bytes  

00198000  98 F1 3E B8 99 F1 E4 BA  99 F1 F0 BA 99 F1 FC BA  
00198010  99 F1 0B BB 99 F1 1A BB  99 F1 29 BB 99 F1 38 BB  
00198020  98 F1 3E B8 99 F1 E4 BA  99 F1 F0 BA 99 F1 FC BA  
00198030  99 F1 0B BB 99 F1 1A BB  99 F1 29 BB 99 F1 38 BB  
00198040  FA FA FA FA FA FA FA FA  FA FA FA FA FA FA FA FA  
00198050  FA FA FA FA FA FA FA FA  FA FA FA FA FA FA FA FA  
00198060  FA FA FA FA FA FA FA FA  FA FA FA FA FA FA FA FA  
00198070  FA FA FA FA 44 42 50 50  58 3C 1E 00 00 80 19 00  
root@ubuntu:/sys/kernel/debug/ieee80211/phy0#  
