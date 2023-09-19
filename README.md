# dlan-openwrt

This is the devolo dLAN package feed for OpenWrt that works with OpenWRT 21.02 ath79 since commit 
https://github.com/openwrt/openwrt/commit/624b85e646328c5f4ff271e6cf0edc5923d40c67

It contains the tools for the Powerline (PLC) interface of devolo dLAN devices.

code with tag openwrt-21.02 will work for openwrt 21.02 with ebtables, master works with openwrt 22.03 and fw4 and netfilter.

openwrt 22.03 and fw4 will only work if plc-tools are used on br-lan interface.

It is different from [the original dlan-openwrt feed](https://github.com/devolo/dlan-openwrt) as it doesn't need the setsid and works with normal openWRT images.
So far it is unknown if there is any issues due to not using setsid as normal operations seems not to need it,
especially if PLC network is manually configured using open-plc-utils instead of using push button pairing. 

NOTE: Due to legal reasons it is not allowed to distribute PLC firmware and pib files needed to build the dLAN firmware packages for OpenWRT.
These files must be acquired from the official Develo firmware package and only then dLAN firmware packages for OpenWRT can be built.

## Usage

### Build the dLAN firmware packages for OpenWRT
This repository is intended to be layered on-top of an OpenWrt buildroot. If you do not have an OpenWrt buildroot installed, see the documentation at: [OpenWrt Buildroot – Installation](http://wiki.openwrt.org/doc/howto/buildroot.exigence) on the OpenWrt support site.

Debian or ubuntu based Linux is assumed to be used for building the dLAN package.

To add the dLAN package to the buildroot add the following line to feeds.conf.default (or feeds.conf if it exists):
```
git clone https://github.com/openwrt/openwrt.git
cd openwrt/
echo "src-git dlan https://github.com/garyttirn/dlan-openwrt.git" >> feeds.conf.default 
```

To install all package definitions, run:
```
./scripts/feeds update -a
./scripts/feeds install -a
```

The procedure for getting the PLC firmware and pib-file is based on [Andre Borie's script](https://bitbucket.org/Rjevski/dlan-1200-ac-firmware-downloader/src/master/)
```
mkdir -p /tmp/dlan
curl -o /tmp/dlan/devolo-firmware-dlan1200-wifiac_5.6.1-1_i386.deb http://update.devolo.com/linux2/apt/pool/main/d/devolo-firmware-dlan1200-wifiac/devolo-firmware-dlan1200-wifiac_5.6.1-1_i386.deb
dpkg -x /tmp/dlan/devolo-firmware-dlan1200-wifiac_5.6.1-1_i386.deb /tmp/dlan
binwalk /tmp/dlan/firmware/devolo-firmware-dlan1200-wifiac/delos_dlan-1200-ac_5.6.1_2020-10-23.bin.dvl -e -C /tmp/dlan

ls /tmp/dlan/firmware/devolo-firmware-dlan1200-wifiac/_delos_dlan-1200-ac_5.6.1_2020-10-23.bin.dvl.extracted/squashfs-root/lib/firmware/plc/dlan-pro-1200-ac/
default_mode_name.txt              patches_mt2731_mimo_vdsl35b.txt  patches_mt2973_siso_full.txt     reset_patch_mt2674.txt
fwconfig                           patches_mt2731_siso_full.txt     qca7500-pib15-devolo-mt2673.pib  reset_patch_mt2675.txt
led_scheme_off                     patches_mt2732_mimo_vdsl35b.txt  qca7500-pib15-devolo-mt2674.pib  reset_patch_mt2676.txt
led_scheme_on                      patches_mt2732_siso_full.txt     qca7500-pib15-devolo-mt2675.pib  reset_patch_mt2730.txt
MAC-7500-v2.8.0-01-NW6__-X-CS.nvm  patches_mt2910_mimo_vdsl35b.txt  qca7500-pib15-devolo-mt2676.pib  reset_patch_mt2731.txt
patches_mt2673_mimo_vdsl35b.txt    patches_mt2910_siso_full.txt     qca7500-pib15-devolo-mt2730.pib  reset_patch_mt2732.txt
patches_mt2673_siso_full.txt       patches_mt2911_mimo_vdsl35b.txt  qca7500-pib15-devolo-mt2731.pib  reset_patch_mt2910.txt
patches_mt2674_mimo_vdsl35b.txt    patches_mt2911_siso_full.txt     qca7500-pib15-devolo-mt2732.pib  reset_patch_mt2911.txt
patches_mt2674_siso_full.txt       patches_mt2912_mimo_vdsl35b.txt  qca7500-pib15-devolo-mt2910.pib  reset_patch_mt2912.txt
patches_mt2675_mimo_vdsl35b.txt    patches_mt2912_siso_full.txt     qca7500-pib15-devolo-mt2911.pib  reset_patch_mt2913.txt
patches_mt2675_siso_full.txt       patches_mt2913_mimo_vdsl35b.txt  qca7500-pib15-devolo-mt2912.pib  reset_patch_mt2966.txt
patches_mt2676_mimo_vdsl35b.txt    patches_mt2913_siso_full.txt     qca7500-pib15-devolo-mt2913.pib  reset_patch_mt2973.txt
patches_mt2676_siso_full.txt       patches_mt2966_mimo_vdsl35b.txt  qca7500-pib15-devolo-mt2966.pib
patches_mt2730_mimo_vdsl35b.txt    patches_mt2966_siso_full.txt     qca7500-pib15-devolo-mt2973.pib
patches_mt2730_siso_full.txt       patches_mt2973_mimo_vdsl35b.txt  reset_patch_mt2673.txt
```

Now you can refer to [OpenWRT devolo dLAN pro 1200+ WiFi ac devide page](https://openwrt.org/toh/devolo/dlan_pro_1200_wifi_ac#powerline_interface)
for details on selecting proper firmware and PIB-file for your particular dLAN 1200+ AC device.

To get the right firmware, you can spot something like MT2555 in top right of the device as seen on [this image](https://openwrt.org/_media/media/devolo/dlan-pro-wireless-500-plus-bottom.jpg). 

Here we use MT2910 found on a dLAN 1200+ AC device (non-pro, white variant) as an example 

For MT2910 you copy the config file 2910.pib and the firmware file MAC-7500-v2.8.0-01-NW6__-X-CS.nvm to openwrt/feeds/dlan/dlan-fw/qca/devolo,dlan-pro-1200plus-ac/

```
cp /tmp/dlan/firmware/devolo-firmware-dlan1200-wifiac/_delos_dlan-1200-ac_5.6.1_2020-10-23.bin.dvl.extracted/squashfs-root/lib/firmware/plc/dlan-pro-1200-ac/{*mt2910.pib,MAC-7500-v2.8.0-01-NW6__-X-CS.nvm} feeds/dlan/dlan-fw/qca/devolo,dlan-pro-1200plus-ac/

ls feeds/dlan/dlan-fw/qca/devolo,dlan-pro-1200plus-ac/
bin  MAC-7500-v2.8.0-01-NW6__-X-CS.nvm  qca7500-pib15-devolo-mt2910.pib
```

Build the dlan firmware and tools packages
```
make download
make prereq
make prepare
```

Make sure to build for DEVICE devolo_dlan-pro-1200plus-ac and include packages dlan-fw-pro-1200plus-ac and dlan-plc
```
echo -e "CONFIG_TARGET_ath79_generic_DEVICE_devolo_dlan-pro-1200plus-ac=y\n
CONFIG_TARGET_PROFILE="DEVICE_devolo_dlan-pro-1200plus-ac"\n
CONFIG_FEED_dlan=y\n
CONFIG_PACKAGE_dlan-fw-pro-1200plus-ac=m\n
CONFIG_PACKAGE_dlan-plc=y" >> .config

make package/dlan-fw/{clean,compile}

ls bin/packages/mips_24kc/dlan
dlan-fw-pro-1200plus-ac_1.2-1_mips_24kc.ipk  dlan-plc_1.2-1_mips_24kc.ipk  Packages  Packages.gz  Packages.manifest
```

Now these packages can be used for subsequent OpenWRT image builds using ImageBuilder making it unneccessary to build the entire OpenWRT image from sources everytime.

### Build the OpenWRT image with dlan-plc and dlan-fw-packages using ImageBuilder

Download ImageBuilder
```
curl -o openwrt-imagebuilder-ath79-generic.Linux-x86_64.tar.xz https://downloads.openwrt.org/snapshots/targets/ath79/generic/openwrt-imagebuilder-ath79-generic.Linux-x86_64.tar.xz

tar Jxvf openwrt-imagebuilder-ath79-generic.Linux-x86_64.tar.xz

cd openwrt-imagebuilder*-ath79-generic.Linux-x86_64/
make clean
```

Prepare firmware packages
```
mkdir -p packages
cp ../openwrt/bin/packages/mips_24kc/dlan/{dlan-fw-pro-1200plus-ac_1.2-1_mips_24kc.ipk,dlan-plc_1.2-1_mips_24kc.ipk} packages

ls packages/
dlan-fw-pro-1200plus-ac_1.2-1_mips_24kc.ipk  dlan-plc_1.2-1_mips_24kc.ipk

```

Add PLC files as custom
```
sed -i 's/^# src custom.*$/src\/gz custom file:packages/' repositories.conf
```

Make image
```
make image \
PROFILE=devolo_dlan-pro-1200plus-ac \
PACKAGES="-ppp -ppp-mod-pppoe dlan-fw-pro-1200plus-ac dlan-plc hostapd-utils luci-base luci-app-firewall luci-app-commands luci-app-opkg luci-mod-admin-full luci-theme-openwrt-2020 nano open-plc-utils-modpib open-plc-utils-plcstat sysstat uhttpd" \
EXTRA_IMAGE_NAME="dlan-plc"

ls  bin/targets/ath79/generic/
openwrt-dlan-plc-ath79-generic-devolo_dlan-pro-1200plus-ac.manifest                 profiles.json
openwrt-dlan-plc-ath79-generic-devolo_dlan-pro-1200plus-ac-squashfs-sysupgrade.bin  sha256sums
```

More the instructions in README.md inside the dlan-fw package of this repository about using the PLC interface.

## License

See individual packages.

Firmware NVM/PIB files :
Copyright (c) Qualcomm Atheros, Inc.
See LICENSE in qca/*

## References 

https://openwrt.org/docs/guide-developer/quickstart-build-images
https://openwrt.org/toh/devolo/dlan_pro_1200_wifi_ac
https://bitbucket.org/Rjevski/dlan-1200-ac-firmware-downloader/src/master/
https://github.com/openwrt/openwrt/pull/3892
https://github.com/openwrt/openwrt/commit/624b85e646328c5f4ff271e6cf0edc5923d40c67
