# Steps to build your own s8xx kernel (armhf) that works with Docker short edition!

I'm purposely skipping a lot of details.

If you don't understand, read the original guide in `README.md`.

Prerequisites:
- Cross compile toolchains for armhf [here](https://releases.linaro.org/components/toolchain/binaries/latest-7/arm-linux-gnueabihf/)

1. `git clone --depth=1` or download the zip for the latest active branch of `meson-mx-integration` [here](https://github.com/xdarklight/linux/branches)

2. Get kernel config file:
```
scp root@<s8xx_device_ip>:/boot/config* .config
```

3. Build kernel:
```
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- LOADADDR=0x00208000 uImage modules
# Hold enter to go through the default configs
```

4. Copy compiled linux source code to target device:
```
rsync -av -e ssh --progress linux-meson-mx-integration-x.xx-yyyymmdd root@<s8xx_ssh_ip>:/root/
```

5. Install new headers and modules from the target device:
```
make modules_install && make headers_install
```

6. Backup kernel, install new kernel and reboot:
```
cp /boot/uImage /boot/uImage.bak
cp arch/arm/boot/uImage /boot/
reboot
```

7. Copy new config to /boot:
```
cp .config /boot/config-x.xx
```

8. Delete kernel source:
```
rm -r linux-meson-mx-integration*
```