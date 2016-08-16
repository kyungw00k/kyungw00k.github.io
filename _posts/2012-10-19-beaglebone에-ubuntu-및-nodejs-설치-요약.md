---
layout: post
title: BeagleBone에 Ubuntu 및 Node.js 설치 요약
date: 2012-10-19 15:06:56.000000000 +09:00
type: post
published: true
status: publish
categories:
- BeagleBone
tags:
- Node.js
- ubuntu
---

생각날때마다 찾기 귀찮아 블로그로 남겨본다.

아래 OS 설치 및 설정이 귀찮으면
[Angstrom 배포본](http://downloads.angstrom-distribution.org/demo/beaglebone/)을 사용하는게 맘 편할듯

Angstrom 관련 boot-loader 및 소스는 
[https://github.com/beagleboard](https://github.com/beagleboard) 참고

### I. OS 설치 및 설정

#### 1. Ubuntu ARM 설치

시작에 앞서, 비글본은 
[armhf](http://wiki.debian.org/ArmHardFloatPort) architecture 를 사용한다.

따라서, 다운 받을때도 armhf 로 되어있는 preinstalled image를 다운받아야 한다.

설치는 Ubuntu에서 진행하는 것으로 한다.


Mac에서 작업하려고 했으나 좀 불편해서.. Vmware 하나 띄워서 작업했다.

- http://rcn-ee.net/deb/rootfs/ 를 들어가면 Preinstalled Image를 나름 최소 용량으로 받을 수 있다.


(http://rcn-ee.net/deb/rootfs/precise/ubuntu-12.04-r8-minimal-armhf-2012-10-19.tar.xz)

- SD 카드를 넣고, 찾아본다.

sudo ./setup_sdcard.sh --probe-mmc

- 필요한 패키지를 설치한다.

sudo apt-get install wget pv dosfstools parted git-core

- SD 카드에 이미지를 심는다.

sudo ./setup_sdcard.sh --mmc /dev/sdb --uboot bone

#### 2. OS 설치 후 터미널 접속해본다.

screen `ls /dev/{tty.usb*B,beaglebone-serial}` 115200

#### 3. Kernel Upgrade


단순 설치만으로는 BeagleBone의 Pin을 사용하기 어려우니, 관련해서 Kernel을 업데이트 해야한다.

URL : http://rcn-ee.net/deb/precise-armhf/LATEST-omap-psp

ABI:1 EXPERIMENTAL http://rcn-ee.net/deb/precise-armhf/v3.2.0-rc2-d0/install-me.sh
ABI:1 TESTING http://rcn-ee.net/deb/precise-armhf/v3.6.2-bone0/install-me.sh
ABI:1 STABLE http://rcn-ee.net/deb/precise-armhf/v3.2.32-psp25/install-me.sh

STABLE 용 URL을 내려받아 root 권한으로 install-me.sh를 실행해 커널을 업데이트 한다.

#### 4. USB Ethernet Gadget로 Internet 잡으려면? udhcpc를 설치한다.
Mac에서는 Internet 공유를 통해 쉽게 설정이 가능하기 떄문에, USB Ethernet Gadget에 공유를 이미 걸어놨다.)

##### A. g_ether 모듈이 로드 되고 있는지 확인한다.

```sh
$ dmesg | grep g_ether
[    2.126892]  gadget: g_ether ready
```

##### B. 로드되고 있다면 `/etc/network/interfaces` 를 수정한다.
`/etc/network/interfaces` 에 아래 내용을 추가한다.

```
auto usb0

iface usb0 inet dhcp
```

#### 5. analog input, PWM, I2C, SPI 확인

**Updated: 12.04 r8 부터는 analog input, PWM, I2C, SPI를 바로 사용할 수 있다.**

```
$  ls /dev/i2c*
 /dev/i2c-1 /dev/i2c-3

$  ls /dev/spi*
 /dev/spidev2.0

$ ls -Fd /sys/devices/platform/omap/e*
 /sys/devices/platform/omap/ecap.0/ /sys/devices/platform/omap/ehrpwm.0/
 /sys/devices/platform/omap/ecap.1/ /sys/devices/platform/omap/ehrpwm.1/
 /sys/devices/platform/omap/ecap.2/ /sys/devices/platform/omap/ehrpwm.2/
 /sys/devices/platform/omap/edma.0/

$ ls /sys/devices/platform/omap/tsc/a*
 /sys/devices/platform/omap/tsc/ain1 /sys/devices/platform/omap/tsc/ain5
 /sys/devices/platform/omap/tsc/ain2 /sys/devices/platform/omap/tsc/ain6
 /sys/devices/platform/omap/tsc/ain3 /sys/devices/platform/omap/tsc/ain7
 /sys/devices/platform/omap/tsc/ain4 /sys/devices/platform/omap/tsc/ain8
```

만약 위와 같이 나오지 않으면 모듈이 로드가 되지 않은 것이니 아래 명령을 실행하고, 다시 한번 확인해보기 바란다.

```sh
$ sudo modprobe ti_tscadc
```

부팅시에 항상 모듈을 불러오기 위해 `/etc/modules`에 `ti_tscadc`를 추가해두자.

```
# /etc/modules: kernel modules to load at boot time.
#
# This file contains the names of kernel modules that should be loaded
# at boot time, one per line. Lines beginning with "#" are ignored.

ti_tscadc
```

### II. NodeJS 0.8.x 설치하기
Updated : v0.8.14 설치

```sh
$ ./configure
$ CXXFLAGS="-fno-rtti -fno-exceptions -march=armv7-a" make -j2
$ CXXFLAGS="-fno-rtti -fno-exceptions -march=armv7-a" make install
```

p.s. 막 글을 저장하려는 찰나에 Raspberry Pi 512M 모델이 도착했다. =_=;
