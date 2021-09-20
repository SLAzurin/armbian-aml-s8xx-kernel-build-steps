# Steps to build your own s8xx kernel (armhf) that works with Docker!

## Download my prebuilt image that works with Docker:
### Armbian Ubuntu 20.04 focal-cli with Linux 5.14-rc-2 [here](https://github.com/SLAzurin/build-armbian-custom/releases/tag/v2021.11-s8xx)
<br/>
<br/>

### Check out [SHORT_GUIDE.md](https://github.com/SLAzurin/armbian-aml-s8xx-kernel-build-steps/blob/main/SHORT_GUIDE.md) for the quick straightforward list of steps if you know what you are doing.  
<br/>
<br/>

It is best you read the guide at least once and understand it before you start.

I am not responsible for any damages you cause to your devices.

If you follow the guide correctly, you should not have any issues.

* Note about Docker: Docker's apt repository for armhf only works on Bionic at the time this guide is written. Focal is not supported on armhf for now. You can install Docker by following this guide here: https://docs.docker.com/engine/install/ubuntu/

* Note: I am using Armbian for s8xx built by balbes150 downloaded from here: https://users.armbian.com/balbes150/s8xx/

* Note: The current version of the s8xx build is with linux v5.10. (As of 2021/08/16)

## Pre-requisites
This guide's duration used to be 2 hours plus when the new Linux kernel was build on the s8xx device itself.  
To speed things up, you can use your windows/linux PC with Intel/AMD cpu.  
I have an i7-10700 and it builds the kernel in 5-10min, compared to 2h45m on my AML s805.  
If you want to use your PC to build the kernel, you need to be using a Linux distro.  
It is easier to just use a Linux PC, but you can technically do these same steps on WSL2 for windows or Ubuntu on Docker, granted you know what you're doing.  
I am using Ubuntu in WSL2 on my windows computer, but the steps are the same as if you're on native Linux.  
Make sure you have already installed `build-essential` like so: `sudo apt-get update && sudo apt-get -y install build-essential`  
Make sure you can SSH to your s8xx device.

For simplicity, I'll be using the root user on Ubuntu.  

You will need to download the cross-compile toolchain from [here](https://releases.linaro.org/components/toolchain/binaries/latest-7/arm-linux-gnueabihf/gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf.tar.xz).  
Extract it to `~/gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf`  
Add this at the end of this file `~/.bashrc`: (create it if it doesnt exist) 
```
PATH=$PATH:"~/gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf/bin"
```
then run: `source ~/.bashrc` or logout and log back in  

## Getting the kernel source code and using the existing config

The kernel source code we are using is located at https://github.com/xdarklight/linux/

From here, I will use the `root` user and the `/root/` directory for my workspace.

Click on the latest most active branch here: https://github.com/xdarklight/linux/branches/active  
(Currently: `meson-mx-integration-5.14-20210718` as of Aug 16 2021).  
Then click on the green `Code` button and click `Download ZIP`.  
Extrct it in the `/root` directory.

s8xx or Command line only steps: 
```
sudo su
cd /root/
wget https://github.com/xdarklight/linux/archive/refs/heads/meson-mx-integration-5.14-20210718.zip
unzip meson-mx-integration-5.14-20210718.zip
```

Copy the config file inside /boot, paste it inside the linux kernel folder, rename it to .config
```
cp /boot/config-5.10.0-aml-s812 /root/linux-meson-mx-integration-5.14-20210718/.config
cd /root/linux-meson-mx-integration-5.14-20210718
```

## Editing the .config

Run this command:
```
# inside /root/linux-meson-mx-integration-5.14-20210718
sed -i -e 's@# CONFIG_POSIX_MQUEUE is not set@CONFIG_POSIX_MQUEUE=y\nCONFIG_POSIX_MQUEUE_SYSCTL=y@g' .config
```
This adds what's missing for the Linux kernel to be compatible with Docker

## Steps to build uImage Linux kernel:
Make sure you are root user or use sudo for all commands below.

If you're building the kernel on the s8xx device and are not using SSH, just run this `make` command.  
The following step will take more than 2 hours to complete on the s8xx device. For me it took 2h45m to finish on my s805 1gb ram Sandisk SD card class 10 U1.
```
cd /root/linux-meson-mx-integration-5.14-20210718/
make -j$(nproc) LOCALVERSION="-aml-s812" LOADADDR=0x00208000 uImage modules && make modules_install && make headers_install
```
If you are using SSH to build the kernel, do it this way instead:
```
screen # press enter to dismiss the license text
cd /root/linux-meson-mx-integration-5.14-20210718/
make -j$(nproc) LOCALVERSION="-aml-s812" LOADADDR=0x00208000 uImage modules && make modules_install && make headers_install
```
To detach the `screen`, Press these keys: `CTRL + a` and `CTRL + d`  
To re-attach to the `screen`, use this command `screen -r`
You can also use `screen -r` if you disconnect from SSH to reconnect to the `screen`.

If you are using a Linux PC to compile the kernel, execute this command instead:
```
cd /root/linux-meson-mx-integration-5.14-20210718/
make -j$(nproc) ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- LOCALVERSION="-aml-s812" LOADADDR=0x00208000 uImage modules
```

When you execute the make command it may ask you to set new config values in addition to the .config's ones.  
The new configs can just be set to default most of the time.  
I just hold the `Enter` key until there is no more config to set. (Sets all new config to default)

You can abort compiling the kernel at anytime with `CTRL + C`

There is no more config to set when you will see the compiler slowly print one line at the time. This is where the waiting game starts.

After the command is finished, you are almost done.

If you are using a Linux PC to compile the kernel, do these extra steps:
```
cd /root/
rsync -av -e ssh --progress linux-meson-mx-integration-5.14-20210718 root@<s8xx_ssh_ip>:/root/
ssh root@<s8xx_ssh_ip>
cd /root/linux-meson-mx-integration-5.14-20210718
make modules_install && make headers_install
```
Alternatively, you can copy the `linux-meson-mx-integration-5.14-20210718` to the s8xx devive however you like

Make a backup of the current Linux Kernel and apply the new one.
```
# still inside /root/linux-meson-mx-integration-5.14-20210718/
cp /boot/uImage /boot/uImage.bak
cp arch/arm/boot/uImage /boot/
```
Restart your device and hope it works.

If it does not work and you are stuck in a black/blank screen, plug the sd card to your computer, delete /boot/uImage, rename /boot/uImage.bak to /boot/uImage

## Optional: copy the new finalized config file to the /boot partition
```
# still inside /root/linux-meson-mx-integration-5.14-20210718/
cp .config /boot/config-5.14
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
