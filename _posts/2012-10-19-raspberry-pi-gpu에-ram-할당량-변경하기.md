---
layout: post
title: Raspberry Pi GPU에 RAM 할당량 변경하기
date: 2012-10-19 18:57:28.000000000 +09:00
type: post
published: true
status: publish
categories:
- Raspberry Pi
---
**Updated : 파일 교체가 아니라 `/boot/config.txt` 에서 설정할 수 있도록 바뀌었습니다.**

Raspberry Pi용 기본 OS에서는 아무런 변경을 하지 않는 경우에는 GPU에 기본으로 64M을 할당하게 됩니다.

256MB 모델의 경우, 192MB/64MB 형태로 부팅이 된다는 것이죠.

만약 GPU를 3D를 돌리는게 아닌 최소한의 용도로 사용한다고 할 경우에는

불필요하게 GPU에 RAM을 줄 필요가 없는 것이죠.

이를 변경하기 위해서는 [이 곳](https://github.com/raspberrypi/firmware/tree/master/boot)에 방문해서 적절한 elf를 내려받아서 적용하시면 될 것 같습니다.

예를 들면, 다음과 같습니다.

> arm128_start.elf - 128MB을 제외한 나머지는 GPU에 할당함

256MB 모델의 경우, GPU에 64MB만 할당하고 싶다면 아래 파일을 사용하시면 되겠습니다.

```
$ sudo cp /boot/arm192_start.elf /boot/start.elf
```

커널 업그레이드 하니 아래와 같은 문구를 발견했습니다.

> ARM/GPU split is now defined in /boot/config.txt using the gpu_mem option!

Github에 가봐도 [예전처럼 별도의 파일](https://github.com/raspberrypi/firmware/commit/c57ea9dd367f12bf4fb41b7b86806a2dc6281176)이 따로 있는게 아니더군요.

`/boot/config.txt`에 `gpu_mem option`을 설정하는 형태로 바뀌었습니다.

훨신 더 간단해진 것 같네요 :D

[http://elinux.org/RPi_config.txt](http://elinux.org/RPi_config.txt) 에 보면 아래와 같은 항목을 찾아볼 수 있습니다.

```
# Memory
disable_l2cache disable ARM access to GPU's L2 cache. Needs corresponding L2 disabled kernel. Default 0

**gpu_mem**
GPU memory in megabyte. Sets the memory split between the ARM and GPU. ARM gets the remaining memory.
**Min 16. Default 64**

gpu_mem_256 GPU memory in megabyte for the 256MB Raspberry Pi. Ignored by the 512MB RP. Overrides gpu_mem. Max 192. Default not set

gpu_mem_512 GPU memory in megabyte for the 512MB Raspberry Pi. Ignored by the 256MB RP. Overrides gpu_mem. Max 448. Default not set
```

설정하지 않으면 기본은 64M을 잡는 것 같습니다.

실제로 dmesg로 확인해보니 그렇네요

```
...

[ 0.000000] Memory: 192MB = 192MB total
[ 0.000000] Memory: 188976k/188976k available, 7632k reserved, 0K highmem
[ 0.000000] Virtual kernel memory layout:
[ 0.000000] vector : 0xffff0000 - 0xffff1000 ( 4 kB)
[ 0.000000] fixmap : 0xfff00000 - 0xfffe0000 ( 896 kB)
[ 0.000000] vmalloc : 0xcc800000 - 0xe8000000 ( 440 MB)
[ 0.000000] lowmem : 0xc0000000 - 0xcc000000 ( 192 MB)
[ 0.000000] modules : 0xbf000000 - 0xc0000000 ( 16 MB)
[ 0.000000] .text : 0xc0008000 - 0xc04c0e78 (4836 kB)
[ 0.000000] .init : 0xc04c1000 - 0xc04e0b10 ( 127 kB)
[ 0.000000] .data : 0xc04e2000 - 0xc050e1c0 ( 177 kB)
[ 0.000000] .bss : 0xc050e1e4 - 0xc05b5128 ( 668 kB)
[ 0.000000] NR_IRQS:330
...
```

입맛에 맛게 설정하시면 될 것 같습니다.

만약 16M으로 설정하고 싶다면 아래와 같이 /boot/config.txt를 설정하시면 될 것 같습니다.

```
gpu_mem=16
```

[이 글](http://raspberrypi.stackexchange.com/questions/673/what-is-the-optimum-split-of-main-versus-gpu-memory)을 보면 이런 이야기도 있네요.

>240MB RAM and 16 VRAM for almost zero graphical power. There is enough GPU memory to render the screen, but not much else. Use this when you need a further non GUI performance boost.

용도가 GUI 환경이 아니라면 GPU에 16MB 정도 주면 된다는 이야기가 있습니다.

실제로 콘솔만 쓴다면 당연하겠죠? :D
