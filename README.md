# Steps to build your own s8xx kernel (armhf) that works with Docker!

It is best you read the guide at least once and understand it before you start.

I am not responsible for any damages you cause to your devices.

If you follow the guide correctly, you should not have any issues.

* Note about Docker: Docker's apt repository for armhf only works on Bionic at the time this guide is written. Focal is not supported on armhf for now. You can install Docker by following this guide here: https://docs.docker.com/engine/install/ubuntu/

* Note: I am using Armbian for s8xx built by balbes150 downloaded from here: https://users.armbian.com/balbes150/s8xx/

* Note: The current version of the s8xx build is with linux v5.10. (As of 2021/07/31)

## Getting the kernel source code and using your existing config

The kernel source code we are using is located at https://github.com/xdarklight/linux/

I am using Linux 5.10 to build Linux 5.11 and to make it work with Docker.

From here, I will use the `root` user and the `/root/` directory for my workspace on my s8xx device.

Power on your s8xx device on Linux and get the 5.11 source code like so: (10-15 minutes Class 10 SD card)
```
sudo su
cd /root/
wget https://github.com/xdarklight/linux/archive/refs/heads/meson-mx-integration-5.11-20201225.zip
unzip meson-mx-integration-5.11-20201225.zip
```

Copy the config file inside /boot, paste it inside the linux kernel folder, rename it to .config
```
cp /boot/config-5.10.0-aml-s812 /root/linux-meson-mx-integration-5.11-20201225/.config
cd /root/linux-meson-mx-integration-5.11-20201225
```

## Editing the .config

Edit the .config file using any text editor.

You can copy it on your host os (Windows or your main computer's OS) if it is easier for you, then copy it back when you are done editing it.

Steps to check what missing config you need for Docker to work: (optional)
```
cd /root/
curl https://raw.githubusercontent.com/docker/docker/master/contrib/check-config.sh > check-config.sh
bash ./check-config.sh
```

This will show you what is missing for Docker.

<h4>⚠️ I will add the missing configs that I needed to add to make Docker work in .config at the end of the guide. ⚠️</h4>

You can just add those missing configs at the end of the .config file and continue on with this guide.

After you are finished editing and saved your .config file, follow these steps:

## Steps to build uImage Linux kernel:

The following step will take more than 2 hours to complete. For me it took 2h45m to finish on my s805 1gb ram Sandisk SD card class 10 U1.

Make sure you are root user or use sudo for all commands: (2h45m duration)
```
cd /root/linux-meson-mx-integration-5.11-20201225/
make -j4 LOCALVERSION="-aml-s812" LOADADDR=0x00208000 uImage modules && make modules_install && make headers_install
```
(The AML s8xx series SoC all have 4 cpu cores, otherwise change the `4` to the number of cpu cores on the machine that you will be compiling it on)

When you execute the make command it may ask you to set new config values in addition to the .config's ones.

The new configs can just be set to default most of the time.

I will just spam the `Enter` key until there is no more config to set. (Sets all new config to default)

You can abort this command at anytime with `CTRL + C`

There is no more config to set when you will see the compiler slowly print one line at the time. This is where the waiting game starts.

After the command is finished, you are almost done.

Make a backup of the current Linux Kernel and apply the new one.
```
# still inside /root/linux-meson-mx-integration-5.11-20201225/
cp /boot/uImage /boot/uImage.bak
cp arch/arm/boot/uImage /boot 
```
Restart your device and hope it works.

If it does not work and you are stuck in a black/blank screen, plug the sd card to your computer, delete /boot/uImage, rename /boot/uImage.bak to /boot/uImage

## Optional: copy the new finalized config file to the /boot partition
```
# still inside /root/linux-meson-mx-integration-5.11-20201225/
cp .config /boot/config-5.11
```
## Checking if Docker works

Install Docker for armhf by following the guide here if you haven't yet: 

https://docs.docker.com/engine/install/ubuntu/ or https://docs.docker.com/engine/install/debian/

	sudo docker run hello-world

The hello-world image should work if you followed the my steps.

The error about POSIX_MQUEUE should be gone.

## References:

Thanks to [ntux](https://forum.armbian.com/profile/11841-ntux/), [alienven](https://www.right.com.cn/forum/space-uid-615766.html) for the build steps and config, [balbes150](https://forum.armbian.com/profile/1215-balbes150/) for the armbian builds, [xdarklight](https://github.com/xdarklight/) for the linux kernel!

https://github.com/xdarklight/linux/branches

https://forum.armbian.com/topic/3023-armbian-for-amlogic-s805-and-s802s812/page/14/?tab=comments#comment-74018

https://forum.armbian.com/topic/3023-armbian-for-amlogic-s805-and-s802s812/page/20/?tab=comments#comment-98704

https://www.right.com.cn/forum/thread-4115554-1-1.html

## Missing config for docker, add these lines at the very end of the .config file:

(5.10 => 5.11) \
CONFIG_POSIX_MQUEUE=y \
CONFIG_POSIX_MQUEUE_SYSCTL=y \

(below here is old)
~~(5.9 => 5.10) \
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
