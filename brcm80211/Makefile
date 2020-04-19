all:
	make -C /lib/modules/$(shell uname -r)/build M=$(shell pwd)/brcmfmac modules

install: 
	@printf "\033[0;31m cp brcmfmac/brcmfmac.ko /lib/modules/$(shell uname -r)/kernel/drivers/net/wireless/broadcom/brcm80211/brcmfmac/brcmfmac.ko\033[0m\n" $@
	$ cp brcmfmac/brcmfmac.ko /lib/modules/$(shell uname -r)/kernel/drivers/net/wireless/broadcom/brcm80211/brcmfmac/brcmfmac.ko 

clean:
	make -C /lib/modules/$(shell uname -r)/build M=$(shell pwd)/brcmfmac clean
