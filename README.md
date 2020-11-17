# Steps to build your own s8xx kernel (armhf)

It is best you read the guide at least once and understand it before you start.

I am not responsible for any damages you cause to your devices.

If you follow the guide correctly, you should not have any issues.

* Note about Docker: Docker's apt repository for armhf only works on Bionic at the time this guide is written.

* Note: I am using Armbian downloaded from by balbes150 here: https://forum.armbian.com/topic/14232-single-armbian-image-for-rk-aml-aw-armhf-armv7/

## The guide starts now.

Head over to https://github.com/xdarklight/linux/branches

Click on the most recent active branch for meson-mx (for me it is meson-mx-integration-5.10-20201115)

* Note: even if your current kernel version of Linux is older than the most recent one, you can still build the newest linux version.

I am using Linux 5.9 to build Linux 5.10 and to make it work with Docker.

Click on the green code dropdown button and click download zip. (the git clone command will take forever so don't clone.)

From here, I will use the root account and the /root/ directory for my workspace on my s8xx device

Copy that downloaded zip into your s8xx device in the /root/ directory and extract it.

	cd /root/

	unzip linux-meson-mx-integration-5.10-20201115.zip


On your s8xx device, copy the config file inside /boot, paste it inside the linux kernel folder, rename it to .config

	cp  /boot/config-5.9.0-rc7-aml-s812 /root/linux-meson-mx-integration-5.10-20201115/.config

Edit the .config file using any text editor.

You can copy it on your host os (Windows or your main computer's OS) if it is easier for you, then copy it back when you are done editing it.

Steps to check what missing config you need for Docker to work:

	cd /root/

	curl https://raw.githubusercontent.com/docker/docker/master/contrib/check-config.sh > check-config.sh

	bash ./check-config.sh

This will show you what is missing for Docker.

You do not need zfs, it is optional even if it is missing (unless you plan to add it).

For me I will not add zfs for Docker.

I will add the missing configs that I needed to add in .config at the end of the guide.

After you are finished editing and saved your .config file, follow these steps:

## Steps to build uImage Linux kernel:

The following step will take more than 2 hours to complete. For me it took 2h45m to finish on my s805 1gb ram Sandisk SD card class 10 U1.

Make sure you are root user or use sudo for all commands:

	cd /root/linux-meson-mx-integration-5.10-20201115/

	make -j 4 LOADADDR=0x00208000 uImage dtbs modules && make modules_install && make headers_install

(The AML s8xx series SoC all have 4 cpu cores, otherwise change the 4 to the number of cpu cores on the machine that you will be compiling it on)

When you execute the make command it may ask you to set new config values in addition to the .config's ones.

The new configs can just be set to default most of the time.

I will just spam the Enter key until there is no more config to set.

You can abort this command at anytime with CTRL + C

After the command is finished, you are almost done.

Make a backup of the current Linux Kernel and apply the new one.

	cp /boot/uImage /boot/uImage.bak

	cp arch/arm/boot/uImage /boot 

Optional: copy the new config file to the /boot partition

	cp .config /boot/config-5.10

Check if Docker works.

Install Docker for armhf by following the guide here: 

https://docs.docker.com/engine/install/ubuntu/

	sudo docker run hello-world

The hello-world image should work if you followed the my steps.

## References:

https://github.com/xdarklight/linux/branches

https://forum.armbian.com/topic/3023-armbian-for-amlogic-s805-and-s802s812/page/14/?tab=comments#comment-74018

https://forum.armbian.com/topic/3023-armbian-for-amlogic-s805-and-s802s812/page/20/?tab=comments#comment-98704

## Missing config for docker, add these lines at the very end of the .config file:

CONFIG_NF_NAT_IPV4=y \
CONFIG_NF_NAT_NEEDED=y \
CONFIG_POSIX_MQUEUE=y \
CONFIG_MEMCG_SWAP_ENABLED=y \
CONFIG_BLK_DEV_THROTTLING=y \
CONFIG_IOSCHED_CFQ=y \
CONFIG_CFQ_GROUP_IOSCHED=y \
CONFIG_CGROUP_HUGETLB=y \
CONFIG_RT_GROUP_SCHED=y \
CONFIG_IP_VS_NFCT=y \
CONFIG_IP_VS_PROTO_TCP=y \
CONFIG_IP_VS_PROTO_UDP=y \
CONFIG_IP_VS_RR=y \
CONFIG_EXT3_FS_XATTR=y \
CONFIG_EXT3_FS_POSIX_ACL=y \
CONFIG_EXT3_FS_SECURITY=y \
CONFIG_INET_XFRM_MODE_TRANSPORT=y \
CONFIG_AUFS_FS=y \
CONFIG_BTRFS_FS_POSIX_ACL=y
