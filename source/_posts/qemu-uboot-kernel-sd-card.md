---
title: QEMU 模拟 uboot 通过 SD 卡启动 Linux kernel
date: 2025-04-12 18:11:33
tags: [linux, uboot, kernel, ARM, qemu]
categories:
  - [Linux_kernel, ARM, kernel]
  - [Linux_kernel, ARM, Boot]
---


整个环境是基于 `Windows 11 + Virtualbox` 来安装的 Ubuntu 24.04 系统中完成的

`Ubuntu 24.04` 系统信息如下

```shell
## Ubuntu 版本
w101@dev-pc:~/workplace/vexpress_env$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 24.04.2 LTS
Release:	24.04
Codename:	noble

## Ubuntu Linux Kernel
w101@dev-pc:~/workplace/vexpress_env$ uname -a
Linux dev-pc 6.8.0-57-generic #59-Ubuntu SMP PREEMPT_DYNAMIC Sat Mar 15 17:40:59 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux
```

<!--more-->

## 安装 ARM 交叉编译器

```bash
sudo apt install gcc-arm-linux-gnueabi
```

安装完成后， 对应的交叉编译工具如下

```shell
w101@dev-pc:~/workplace/vexpress_env$ arm-linux-gnueabi-
arm-linux-gnueabi-addr2line      arm-linux-gnueabi-gcc-ar         arm-linux-gnueabi-gcov-tool      arm-linux-gnueabi-objcopy
arm-linux-gnueabi-ar             arm-linux-gnueabi-gcc-ar-13      arm-linux-gnueabi-gcov-tool-13   arm-linux-gnueabi-objdump
arm-linux-gnueabi-as             arm-linux-gnueabi-gcc-nm         arm-linux-gnueabi-gold           arm-linux-gnueabi-ranlib
arm-linux-gnueabi-c++filt        arm-linux-gnueabi-gcc-nm-13      arm-linux-gnueabi-gprof          arm-linux-gnueabi-readelf
arm-linux-gnueabi-cpp            arm-linux-gnueabi-gcc-ranlib     arm-linux-gnueabi-ld             arm-linux-gnueabi-size
arm-linux-gnueabi-cpp-13         arm-linux-gnueabi-gcc-ranlib-13  arm-linux-gnueabi-ld.bfd         arm-linux-gnueabi-strings
arm-linux-gnueabi-dwp            arm-linux-gnueabi-gcov           arm-linux-gnueabi-ld.gold        arm-linux-gnueabi-strip
arm-linux-gnueabi-elfedit        arm-linux-gnueabi-gcov-13        arm-linux-gnueabi-lto-dump
arm-linux-gnueabi-gcc            arm-linux-gnueabi-gcov-dump      arm-linux-gnueabi-lto-dump-13
arm-linux-gnueabi-gcc-13         arm-linux-gnueabi-gcov-dump-13   arm-linux-gnueabi-nm
```

## 下载&编译 u-boot

### 下载 u-boot

```bash
git clone git@source.denx.de:u-boot/u-boot.git
```

对应的 `u-boot` 版本为 `2025.01-rc1` , 见对应的 `Makefile` 中的信息如下

```text
# SPDX-License-Identifier: GPL-2.0+

VERSION = 2025
PATCHLEVEL = 01
SUBLEVEL =
EXTRAVERSION = -rc1
NAME =

# *DOCUMENTATION*
# To see a list of typical targets execute "make help"
# More info can be located in ./README
# Comments in this file are targeted only to the developer, do not
# expect to learn how to build the kernel reading this file.

# Do not use make's built-in rules and variables
# (this increases performance and avoids hard-to-debug behaviour)
MAKEFLAGS += -rR

```

### 配置 & 编译

```bash
## 进入 u-boot 目录
cd u-boot

## 配置
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- vexpress_ca9x4_defconfig

## 编译
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi-
```

测试 u-boot

```bash
qemu-system-arm -M vexpress-a9 \
     -kernel u-boot \
     -nographic \
     -m 512M
```

出现如下打印信息，表示一切OK
```
w101@dev-pc:~/workplace/vexpress_env/u-boot$ qemu-system-arm -M vexpress-a9 \
     -kernel u-boot \
     -nographic \
     -m 512M

U-Boot 2025.01-rc1-00090-ge61ea9f2e5d2 (Apr 06 2025 - 13:46:20 +0800)

DRAM:  512 MiB
WARNING: Caches not enabled
Core:  23 devices, 11 uclasses, devicetree: embed
Flash: 128 MiB
MMC:   mmci@5000: 0
Loading Environment from Flash... *** Warning - bad CRC, using default environment

In:    uart@9000
Out:   uart@9000
Err:   uart@9000
Net:   eth0: ethernet@3,02000000
Hit any key to stop autoboot:  0
MMC Device 1 not found
no mmc device at slot 1
Card did not respond to voltage select! : -110
smc911x: detected LAN9118 controller
smc911x: phy initialized
smc911x: MAC 52:54:00:12:34:56
BOOTP broadcast 1
BOOTP broadcast 2
...
```

## 下载&编译 kernel

### 下载 kernel

```bash
wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.14.tar.xz
```

### 解压 kernel

```bash
tar Jxvf linux-6.14.tar.xz
```

### 配置 & 编译

```bash
## 进入 linux kernel 源码
cd linux-6.14

## 配置
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- vexpress_defconfig O=./build
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- menuconfig O=./build

## 编译
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- -j8 O=./build

# 或者单独编译 zImage 和 dts, 命令如下:
## 编译 zImage
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- zImage -j8
## 编译 dts
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- dtbs
```

生成的 `zImage` 和 `dts` 文件在目录 `build/arch/arm/boot/` 和 `build/arch/arm/boot/dts/arm/`

```bash
## zImage
w101@dev-pc:~/workplace/vexpress_env/linux-6.14$ ls build/arch/arm/boot/
compressed  dts  Image  zImage

## dts
w101@dev-pc:~/workplace/vexpress_env/linux-6.14$ ls build/arch/arm/boot/dts/arm/*.dtb
build/arch/arm/boot/dts/arm/vexpress-v2p-ca15_a7.dtb   build/arch/arm/boot/dts/arm/vexpress-v2p-ca5s.dtb
build/arch/arm/boot/dts/arm/vexpress-v2p-ca15-tc1.dtb  build/arch/arm/boot/dts/arm/vexpress-v2p-ca9.dtb
```

## 制作根文件系统

以下基于 `busybox` 制作根文件系统 (也可以基于 `buildroot` , 后续在介绍)

### 下载 busybox

```bash
wget https://www.busybox.net/downloads/busybox-1.36.1.tar.bz2
```

### 解压 busybox

```bash
tar jxvf busybox-1.36.1.tar.bz2
```

### 配置 busybox

```bash
## 进入 busybox 目录
cd busybox-1.36.1/

## 配置
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- menuconfig
```

选中如下配置

```config
Setting --->
	--- Build Optioins
	[*] Build static binary (no shared libs)
	[ ] Force NOMMU build (NEW)
	(arm-linux-gnueabi-) Cross compiler prefix
```

### 编译 busybox

```bash
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi-
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- install
```

`make install` 后，在busybox目录下生成 `_install` 目录，该目录下的程序 就是根文件系统中的程序。

然后使用如下命令生成根文件系统

```bash
#!/bin/bash
sudo mkdir rootfs
sudo cp busybox-1.36.1/_install/*  rootfs/ -raf

sudo mkdir -p rootfs/proc/
sudo mkdir -p rootfs/sys/
sudo mkdir -p rootfs/tmp/
sudo mkdir -p rootfs/root/
sudo mkdir -p rootfs/var/
sudo mkdir -p rootfs/mnt/

sudo cp etc rootfs/ -arf

sudo cp -arf /usr/arm-linux-gnueabi/lib rootfs/

sudo rm rootfs/lib/*.a
sudo arm-linux-gnueabi-strip rootfs/lib/*

sudo mkdir -p rootfs/dev/
sudo mknod rootfs/dev/tty1 c 4 1
sudo mknod rootfs/dev/tty2 c 4 2
sudo mknod rootfs/dev/tty3 c 4 3
sudo mknod rootfs/dev/tty4 c 4 4
sudo mknod rootfs/dev/console c 5 1
sudo mknod rootfs/dev/null c 1 3
```

以上 命令 执行完成后， 则生成根文件系统 `rootfs`

## 制作 SD 卡

通过以上步骤，已编译出 `u-boot` 、 `zImage` 和 `dts`, 以及制作好 根文件系统 `rootfs`

以下制作 `SD` 卡， 并把编译的文件  和 根文件系统保存到 `SD` 卡

```bash
## 1. 生成一个空的SD卡镜像
w101@dev-pc:~/workplace/vexpress_env$ dd if=/dev/zero of=sd.img bs=1M count=512
512+0 records in
512+0 records out
536870912 bytes (537 MB, 512 MiB) copied, 0.989246 s, 543 MB/s


## 2. 创建GPT分区，
### 创建了两个分区，
### 一个用来存放 zImage 和 dts，
### 另一个存放根文件系统
w101@dev-pc:~/workplace/vexpress_env$ sgdisk -n 0:0:+64M -c 0:kernel sd.img
Creating new GPT entries in memory.
Warning: The kernel is still using the old partition table.
The new table will be used at the next reboot or after you
run partprobe(8) or kpartx(8)
The operation has completed successfully.

w101@dev-pc:~/workplace/vexpress_env$ sgdisk -n 0:0:0 -c 0:rootfs sd.img
Warning: The kernel is still using the old partition table.
The new table will be used at the next reboot or after you
run partprobe(8) or kpartx(8)
The operation has completed successfully.

## 3. 查看分区：
w101@dev-pc:~/workplace/vexpress_env$ sgdisk -p sd.img
Disk sd.img: 1048576 sectors, 512.0 MiB
Sector size (logical): 512 bytes
Disk identifier (GUID): D43E571C-071F-45BA-BA99-18C918623F9A
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 1048542
Partitions will be aligned on 2048-sector boundaries
Total free space is 2014 sectors (1007.0 KiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048          133119   64.0 MiB    8300  kernel
   2          133120         1048542   447.0 MiB   8300  rootfs

## 4. 查找空闲的loop设备
w101@dev-pc:~/workplace/vexpress_env$ losetup -f
/dev/loop19

## 5. 将SD卡镜像映射到loop设备上
w101@dev-pc:~/workplace/vexpress_env$ sudo losetup /dev/loop19 sd.img
w101@dev-pc:~/workplace/vexpress_env$ sudo partprobe /dev/loop19

### 此时会看到 /dev/loop19p1 和 /dev/loop19p2 两个节点

## 6. 格式化
w101@dev-pc:~/workplace/vexpress_env$ sudo mkfs.ext4 /dev/loop19p1
w101@dev-pc:~/workplace/vexpress_env$ sudo mkfs.ext4 /dev/loop19p2

## 7. 挂载
mkdir p1
mkdir p2
sudo mount -t ext4 /dev/loop19p1 p1/
sudo mount -t ext4 /dev/loop19p2 p2/

## 8. 拷贝文件
sudo cp linux-6.14/build/arch/arm/boot/zImage p1/
sudo cp linux-6.14/build/arch/arm/boot/dts/arm/*.dtb p1/
sudo cp -raf rootfs/* p2/

## 9.umount
sudo umount p1 p2
sudo losetup -d /dev/loop19
```

## QEMU 运行 SD 卡

```bash
w101@dev-pc:~/workplace/vexpress_env$ qemu-system-arm -M vexpress-a9 -m 512M -kernel u-boot/u-boot -nographic -sd sd.img -append "root=/dev/mmcblk0p2 console=ttyAMA0"
```

当运行到 `Hit any key to stop autoboot:  3 `  倒计时时， 按下 `Enter` 键， 并进行从SD加载Linux kernel和 dts的设置

u-boot 命令行下的相关命令如下：

```bash
=> mmc dev 0
=> mmc info
=> part list mmc 0
=> ls mmc 0:1
=> ls mmc 0:2
=> load mmc 0:1 0x60008000 zImage
=> load mmc 0:1 0x61000000 vexpress-v2p-ca9.dtb
=> setenv bootargs 'root=/dev/mmcblk0p2 rw rootfstype=ext4 rootwait earlycon
=> bootz 0x60008000 - 0x61000000
```

对应的命令输出如下所示

```bash
w101@dev-pc:~/workplace/vexpress_env$ qemu-system-arm -M vexpress-a9 -m 512M -kernel u-boot/u-boot -nographic -sd sd.img -append "root=/dev/mmcblk0p2 console=ttyAMA0"
WARNING: Image format was not specified for 'sd.img' and probing guessed raw.
         Automatically detecting the format is dangerous for raw images, write operations on block 0 will be restricted.
         Specify the 'raw' format explicitly to remove the restrictions.
pulseaudio: set_sink_input_volume() failed
pulseaudio: Reason: Invalid argument
pulseaudio: set_sink_input_mute() failed
pulseaudio: Reason: Invalid argument


U-Boot 2025.01-rc1-00090-ge61ea9f2e5d2 (Apr 06 2025 - 13:46:20 +0800)

DRAM:  512 MiB
WARNING: Caches not enabled
Core:  23 devices, 11 uclasses, devicetree: embed
Flash: 128 MiB
MMC:   mmci@5000: 0
Loading Environment from Flash... *** Warning - bad CRC, using default environment

In:    uart@9000
Out:   uart@9000
Err:   uart@9000
Net:   eth0: ethernet@3,02000000
Hit any key to stop autoboot:  0
=> mmc dev 0
switch to partitions #0, OK
mmc0 is current device
=> mmc info
Device: mmci@5000
Manufacturer ID: aa
OEM: 5859
Name: QEMU!
Bus Speed: 12000000
Mode: MMC legacy
Rd Block Len: 512
SD version 3.0
High Capacity: No
Capacity: 512 MiB
Bus Width: 1-bit
Erase Group Size: 512 Bytes
=> part list mmc 0

Partition Map for mmc device 0  --   Partition Type: EFI

Part	Start LBA	End LBA		Name
	Attributes
	Type GUID
	Partition GUID
  1	0x00000800	0x000207ff	"kernel"
	attrs:	0x0000000000000000
	type:	0fc63daf-8483-4772-8e79-3d69d8477de4
	guid:	90ac0a30-6c3a-44f7-8c4e-df4517b4b31e
  2	0x00020800	0x000fffde	"rootfs"
	attrs:	0x0000000000000000
	type:	0fc63daf-8483-4772-8e79-3d69d8477de4
	guid:	a1619328-dbbd-42cf-8eee-3dbce2309a83
=> ls mmc 0:1
<DIR>       4096 .
<DIR>       4096 ..
<DIR>      16384 lost+found
         5912432 zImage
           18711 vexpress-v2p-ca15_a7.dtb
           13104 vexpress-v2p-ca15-tc1.dtb
           12704 vexpress-v2p-ca5s.dtb
           14329 vexpress-v2p-ca9.dtb
=> ls mmc 0:2
<DIR>       4096 .
<DIR>       4096 ..
<DIR>      16384 lost+found
<DIR>       4096 bin
<DIR>       4096 dev
<DIR>       4096 etc
<DIR>       4096 lib
<SYM>         11 linuxrc
<DIR>       4096 mnt
<DIR>       4096 proc
<DIR>       4096 root
<DIR>       4096 sbin
<DIR>       4096 sys
<DIR>       4096 tmp
<DIR>       4096 usr
<DIR>       4096 var
=> load mmc 0:1 0x60008000 zImage
5912432 bytes read in 9094 ms (634.8 KiB/s)
=> load mmc 0:1 0x61000000 vexpress-v2p-ca9.dtb
14329 bytes read in 32 ms (436.5 KiB/s)
=> setenv bootargs 'root=/dev/mmcblk0p2 rw rootfstype=ext4 rootwait earlycon console=tty0 console=ttyAMA0 init=/linuxrc ignore_loglevel'
=> bootz 0x60008000 - 0x61000000
Kernel image @ 0x60008000 [ 0x000000 - 0x5a3770 ]
## Flattened Device Tree blob at 61000000
   Booting using the fdt blob at 0x61000000
Working FDT set to 61000000
   Loading Device Tree to 7eb0c000, end 7eb127f8 ... OK
Working FDT set to 7eb0c000

Starting kernel ...

Booting Linux on physical CPU 0x0
Linux version 6.14.0 (w512@dev-pc) (arm-linux-gnueabi-gcc (Ubuntu 13.3.0-6ubuntu2~24.04) 13.3.0, GNU ld (GNU Binutils for Ubuntu) 2.42) #1 SMP Sun Apr  6 14:02:45 CST 2025
CPU: ARMv7 Processor [410fc090] revision 0 (ARMv7), cr=10c5387d
CPU: PIPT / VIPT nonaliasing data cache, VIPT nonaliasing instruction cache
OF: fdt: Machine model: V2P-CA9
OF: fdt: Ignoring memory block 0x80000000 - 0x80000004
earlycon: pl11 at MMIO 0x10009000 (options '')
printk: legacy bootconsole [pl11] enabled
printk: debug: ignoring loglevel setting.
Memory policy: Data cache writeback
cma: Reserved 16 MiB at 0x7f000000 on node -1
Zone ranges:
  Normal   [mem 0x0000000060000000-0x000000007fffffff]
Movable zone start for each node
Early memory node ranges
  node   0: [mem 0x0000000060000000-0x000000007fffffff]
Initmem setup node 0 [mem 0x0000000060000000-0x000000007fffffff]
Reserved memory: created DMA memory pool at 0x4c000000, size 8 MiB
OF: reserved mem: initialized node vram@4c000000, compatible id shared-dma-pool
OF: reserved mem: 0x4c000000..0x4c7fffff (8192 KiB) nomap non-reusable vram@4c000000
CPU: All CPU(s) started in SVC mode.
percpu: Embedded 17 pages/cpu s38412 r8192 d23028 u69632
pcpu-alloc: s38412 r8192 d23028 u69632 alloc=17*4096
pcpu-alloc: [0] 0 [0] 1 [0] 2 [0] 3
Kernel command line: root=/dev/mmcblk0p2 rw rootfstype=ext4 rootwait earlycon console=tty0 console=ttyAMA0 init=/linuxrc ignore_loglevel
printk: log_buf_len individual max cpu contribution: 4096 bytes
printk: log_buf_len total cpu_extra contributions: 12288 bytes
printk: log_buf_len min size: 16384 bytes
printk: log buffer data + meta data: 32768 + 102400 = 135168 bytes
printk: early log buf free: 14652(89%)
Dentry cache hash table entries: 65536 (order: 6, 262144 bytes, linear)
Inode-cache hash table entries: 32768 (order: 5, 131072 bytes, linear)
Built 1 zonelists, mobility grouping on.  Total pages: 131072
mem auto-init: stack:all(zero), heap alloc:off, heap free:off
SLUB: HWalign=64, Order=0-3, MinObjects=0, CPUs=4, Nodes=1
rcu: Hierarchical RCU implementation.
rcu: 	RCU event tracing is enabled.
rcu: 	RCU restricting CPUs from NR_CPUS=8 to nr_cpu_ids=4.
	Tracing variant of Tasks RCU enabled.
rcu: RCU calculated value of scheduler-enlistment delay is 10 jiffies.
rcu: Adjusting geometry for rcu_fanout_leaf=16, nr_cpu_ids=4
RCU Tasks Trace: Setting shift to 2 and lim to 1 rcu_task_cb_adjust=1 rcu_task_cpu_ids=4.
NR_IRQS: 16, nr_irqs: 16, preallocated irqs: 16
GIC CPU mask not found - kernel will fail to boot.
GIC CPU mask not found - kernel will fail to boot.
L2C: platform modifies aux control register: 0x02020000 -> 0x02420000
L2C: DT/platform modifies aux control register: 0x02020000 -> 0x02420000
L2C-310 enabling early BRESP for Cortex-A9
L2C-310 full line of zeros enabled for Cortex-A9
L2C-310 dynamic clock gating disabled, standby mode disabled
L2C-310 cache controller enabled, 8 ways, 128 kB
L2C-310: CACHE_ID 0x410000c8, AUX_CTRL 0x46420001
rcu: srcu_init: Setting srcu_struct sizes based on contention.
sched_clock: 32 bits at 24MHz, resolution 41ns, wraps every 89478484971ns
clocksource: arm,sp804: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 1911260446275 ns
smp_twd: clock not found -2
Console: colour dummy device 80x30
printk: legacy console [tty0] enabled
Calibrating local timer... 101.32MHz.
Calibrating delay loop... 2845.90 BogoMIPS (lpj=14229504)
CPU: Testing write buffer coherency: ok
CPU0: Spectre v2: using BPIALL workaround
pid_max: default: 32768 minimum: 301
Mount-cache hash table entries: 1024 (order: 0, 4096 bytes, linear)
Mountpoint-cache hash table entries: 1024 (order: 0, 4096 bytes, linear)
CPU0: thread -1, cpu 0, socket 0, mpidr 80000000
Setting up static identity map for 0x60100000 - 0x60100060
rcu: Hierarchical SRCU implementation.
rcu: 	Max phase no-delay instances is 1000.
Timer migration: 1 hierarchy levels; 8 children per group; 1 crossnode level
smp: Bringing up secondary CPUs ...
smp: Brought up 1 node, 1 CPU
SMP: Total of 1 processors activated (2845.90 BogoMIPS).
CPU: All CPU(s) started in SVC mode.
Memory: 486388K/524288K available (9216K kernel code, 747K rwdata, 2156K rodata, 1024K init, 187K bss, 19264K reserved, 16384K cma-reserved)
devtmpfs: initialized
VFP support v0.3: implementor 41 architecture 3 part 30 variant 9 rev 0
clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 19112604462750000 ns
futex hash table entries: 1024 (order: 4, 65536 bytes, linear)
NET: Registered PF_NETLINK/PF_ROUTE protocol family
DMA: preallocated 256 KiB pool for atomic coherent allocations
cpuidle: using governor ladder
hw-breakpoint: debug architecture 0x4 unsupported.
Serial: AMBA PL011 UART driver
/bus@40000000/motherboard-bus@40000000/iofpga@7,00000000/i2c@16000/dvi-transmitter@39: Fixed dependency cycle(s) with /bus@40000000/motherboard-bus@40000000/iofpga@7,00000000/clcd@1f000
/bus@40000000/motherboard-bus@40000000/iofpga@7,00000000/clcd@1f000: Fixed dependency cycle(s) with /bus@40000000/motherboard-bus@40000000/iofpga@7,00000000/i2c@16000/dvi-transmitter@39
/bus@40000000/motherboard-bus@40000000/iofpga@7,00000000/i2c@16000/dvi-transmitter@39: Fixed dependency cycle(s) with /bus@40000000/motherboard-bus@40000000/iofpga@7,00000000/clcd@1f000
/bus@40000000/motherboard-bus@40000000/iofpga@7,00000000/clcd@1f000: Fixed dependency cycle(s) with /bus@40000000/motherboard-bus@40000000/iofpga@7,00000000/i2c@16000/dvi-transmitter@39
/bus@40000000/motherboard-bus@40000000/iofpga@7,00000000/i2c@16000/dvi-transmitter@39: Fixed dependency cycle(s) with /bus@40000000/motherboard-bus@40000000/iofpga@7,00000000/clcd@1f000
/bus@40000000/motherboard-bus@40000000/iofpga@7,00000000/clcd@1f000: Fixed dependency cycle(s) with /bus@40000000/motherboard-bus@40000000/iofpga@7,00000000/i2c@16000/dvi-transmitter@39
/bus@40000000/motherboard-bus@40000000/iofpga@7,00000000/i2c@16000/dvi-transmitter@39: Fixed dependency cycle(s) with /bus@40000000/motherboard-bus@40000000/iofpga@7,00000000/clcd@1f000
/bus@40000000/motherboard-bus@40000000/iofpga@7,00000000/i2c@16000/dvi-transmitter@39: Fixed dependency cycle(s) with /bus@40000000/motherboard-bus@40000000/iofpga@7,00000000/clcd@1f000
/bus@40000000/motherboard-bus@40000000/iofpga@7,00000000/clcd@1f000: Fixed dependency cycle(s) with /bus@40000000/motherboard-bus@40000000/iofpga@7,00000000/i2c@16000/dvi-transmitter@39
/bus@40000000/motherboard-bus@40000000/iofpga@7,00000000/i2c@16000/dvi-transmitter@39: Fixed dependency cycle(s) with /clcd@10020000
/clcd@10020000: Fixed dependency cycle(s) with /bus@40000000/motherboard-bus@40000000/iofpga@7,00000000/i2c@16000/dvi-transmitter@39
SCSI subsystem initialized
libata version 3.00 loaded.
usbcore: registered new interface driver usbfs
usbcore: registered new interface driver hub
usbcore: registered new device driver usb
/clcd@10020000: Fixed dependency cycle(s) with /bus@40000000/motherboard-bus@40000000/iofpga@7,00000000/i2c@16000/dvi-transmitter@39
/bus@40000000/motherboard-bus@40000000/iofpga@7,00000000/clcd@1f000: Fixed dependency cycle(s) with /bus@40000000/motherboard-bus@40000000/iofpga@7,00000000/i2c@16000/dvi-transmitter@39
/bus@40000000/motherboard-bus@40000000/iofpga@7,00000000/i2c@16000/dvi-transmitter@39: Fixed dependency cycle(s) with /bus@40000000/motherboard-bus@40000000/iofpga@7,00000000/clcd@1f000
/bus@40000000/motherboard-bus@40000000/iofpga@7,00000000/i2c@16000/dvi-transmitter@39: Fixed dependency cycle(s) with /clcd@10020000
pps_core: LinuxPPS API ver. 1 registered
pps_core: Software ver. 5.3.6 - Copyright 2005-2007 Rodolfo Giometti <giometti@linux.it>
PTP clock support registered
Advanced Linux Sound Architecture Driver Initialized.
clocksource: Switched to clocksource arm,sp804
NET: Registered PF_INET protocol family
IP idents hash table entries: 8192 (order: 4, 65536 bytes, linear)
tcp_listen_portaddr_hash hash table entries: 512 (order: 0, 4096 bytes, linear)
Table-perturb hash table entries: 65536 (order: 6, 262144 bytes, linear)
TCP established hash table entries: 4096 (order: 2, 16384 bytes, linear)
TCP bind hash table entries: 4096 (order: 4, 65536 bytes, linear)
TCP: Hash tables configured (established 4096 bind 4096)
UDP hash table entries: 256 (order: 1, 14336 bytes, linear)
UDP-Lite hash table entries: 256 (order: 1, 14336 bytes, linear)
NET: Registered PF_UNIX/PF_LOCAL protocol family
RPC: Registered named UNIX socket transport module.
RPC: Registered udp transport module.
RPC: Registered tcp transport module.
RPC: Registered tcp-with-tls transport module.
RPC: Registered tcp NFSv4.1 backchannel transport module.
workingset: timestamp_bits=30 max_order=17 bucket_order=0
squashfs: version 4.0 (2009/01/31) Phillip Lougher
jffs2: version 2.2. (NAND) © 2001-2006 Red Hat, Inc.
9p: Installing v9fs 9p2000 file system support
io scheduler mq-deadline registered
io scheduler kyber registered
io scheduler bfq registered
ledtrig-cpu: registered to indicate activity on CPUs
sii902x 0-0060: supply iovcc not found, using dummy regulator
sii902x 0-0060: supply cvcc12 not found, using dummy regulator
simple-pm-bus bus@40000000:motherboard-bus@40000000:iofpga@7,00000000: Failed to create device link (0x180) with supplier dcc:clock-controller-2 for /bus@40000000/motherboard-bus@40000000/iofpga@7,00000000/sysctl@1000
simple-pm-bus bus@40000000:motherboard-bus@40000000:iofpga@7,00000000: Failed to create device link (0x180) with supplier dcc:clock-controller-2 for /bus@40000000/motherboard-bus@40000000/iofpga@7,00000000/sysctl@1000
physmap-flash 40000000.flash: physmap platform flash device: [mem 0x40000000-0x43ffffff]
40000000.flash: Found 2 x16 devices at 0x0 in 32-bit bank. Manufacturer ID 0x000000 Chip ID 0x000000
Intel/Sharp Extended Query Table at 0x0031
Using buffer write method
erase region 0: offset=0x0,size=0x40000,blocks=256
physmap-flash 40000000.flash: physmap platform flash device: [mem 0x44000000-0x47ffffff]
40000000.flash: Found 2 x16 devices at 0x0 in 32-bit bank. Manufacturer ID 0x000000 Chip ID 0x000000
Intel/Sharp Extended Query Table at 0x0031
Using buffer write method
erase region 0: offset=0x0,size=0x40000,blocks=256
Concatenating MTD devices:
(0): "40000000.flash"
(1): "40000000.flash"
into device "40000000.flash"
physmap-flash 48000000.psram: physmap platform flash device: [mem 0x48000000-0x49ffffff]
smsc911x 4e000000.ethernet eth0: MAC Address: 52:54:00:12:34:56
isp1760 4f000000.usb: isp1760 bus width: 32, oc: digital
isp1760 4f000000.usb: NXP ISP1760 USB Host Controller
isp1760 4f000000.usb: new USB bus registered, assigned bus number 1
isp1760 4f000000.usb: Scratch test failed. 0x00000000
isp1760 4f000000.usb: can't setup: -19
isp1760 4f000000.usb: USB bus 1 deregistered
usbcore: registered new interface driver usb-storage
rtc-pl031 10017000.rtc: registered as rtc0
rtc-pl031 10017000.rtc: setting system clock to 2025-04-06T09:09:16 UTC (1743930556)
mmci-pl18x 10005000.mmci: Got CD GPIO
mmci-pl18x 10005000.mmci: Got WP GPIO
input: AT Raw Set 2 keyboard as /devices/platform/bus@40000000/bus@40000000:motherboard-bus@40000000/bus@40000000:motherboard-bus@40000000:iofpga@7,00000000/10006000.kmi/serio0/input/input0
mmci-pl18x 10005000.mmci: mmc0: PL181 manf 41 rev0 at 0x10005000 irq 31,32 (pio)
usbcore: registered new interface driver usbhid
usbhid: USB HID core driver
hw perfevents: enabled with armv7_cortex_a9 PMU driver, 7 (8000003f) counters available
mmc0: new SD card at address 3ba3
mmcblk0: mmc0:3ba3 QEMU! 512 MiB
aaci-pl041 10004000.aaci: ARM AC'97 Interface PL041 rev0 at 0x10004000, irq 37
aaci-pl041 10004000.aaci: FIFO 512 entries
NET: Registered PF_PACKET protocol family
9pnet: Installing 9P2000 support
Registering SWP/SWPB emulation handler
input: ImExPS/2 Generic Explorer Mouse as /devices/platform/bus@40000000/bus@40000000:motherboard-bus@40000000/bus@40000000:motherboard-bus@40000000:iofpga@7,00000000/10007000.kmi/serio1/input/input2
10009000.serial: ttyAMA0 at MMIO 0x10009000 (irq = 38, base_baud = 0) is a PL011 rev1
printk: legacy console [ttyAMA0] enabled
printk: legacy console [ttyAMA0] enabled
printk: legacy bootconsole [pl11] disabled
printk: legacy bootconsole [pl11] disabled
 mmcblk0: p1 p2
1000a000.serial: ttyAMA1 at MMIO 0x1000a000 (irq = 39, base_baud = 0) is a PL011 rev1
1000b000.serial: ttyAMA2 at MMIO 0x1000b000 (irq = 40, base_baud = 0) is a PL011 rev1
1000c000.serial: ttyAMA3 at MMIO 0x1000c000 (irq = 41, base_baud = 0) is a PL011 rev1
drm-clcd-pl111 1001f000.clcd: assigned reserved memory node vram@4c000000
drm-clcd-pl111 1001f000.clcd: using device-specific reserved memory
drm-clcd-pl111 1001f000.clcd: core tile graphics present
drm-clcd-pl111 1001f000.clcd: this device will be deactivated
drm-clcd-pl111 1001f000.clcd: Versatile Express init failed - -19
drm-clcd-pl111 10020000.clcd: DVI muxed to daughterboard 1 (core tile) CLCD
drm-clcd-pl111 10020000.clcd: initializing Versatile Express PL111
drm-clcd-pl111 10020000.clcd: DVI muxed to daughterboard 1 (core tile) CLCD
drm-clcd-pl111 10020000.clcd: initializing Versatile Express PL111
drm-clcd-pl111 10020000.clcd: DVI muxed to daughterboard 1 (core tile) CLCD
drm-clcd-pl111 10020000.clcd: initializing Versatile Express PL111
clk: Disabling unused clocks
ALSA device list:
  #0: ARM AC'97 Interface PL041 rev0 at 0x10004000, irq 37
EXT4-fs (mmcblk0p2): recovery complete
EXT4-fs (mmcblk0p2): mounted filesystem 6afd9b97-bfc0-4b42-a55a-0511064a0620 r/w with ordered data mode. Quota mode: disabled.
VFS: Mounted root (ext4 filesystem) on device 179:2.
Freeing unused kernel image (initmem) memory: 1024K
Run /linuxrc as init process
  with arguments:
    /linuxrc
  with environment:
    HOME=/
    TERM=linux
random: crng init done
mount: mounting debugfs on /sys/kernel/debug failed: No such file or directory
/etc/init.d/rcS: line 14: can't create /proc/sys/kernel/hotplug: nonexistent directory

Please press Enter to activate this console. drm-clcd-pl111 10020000.clcd: DVI muxed to daughterboard 1 (core tile) CLCD
drm-clcd-pl111 10020000.clcd: initializing Versatile Express PL111
i2c 0-0039: deferred probe pending: sii902x: Failed to find remote bridge
amba 1000f000.wdt: deferred probe pending: (reason unknown)
amba 10020000.clcd: deferred probe pending: (reason unknown)
amba 100e0000.memory-controller: deferred probe pending: (reason unknown)
amba 100e1000.memory-controller: deferred probe pending: (reason unknown)
amba 100e5000.watchdog: deferred probe pending: (reason unknown)

[root@vexpress ]#
```

## 参考

- [用Qemu模拟vexpress-a9 （一） --- 搭建Linux kernel调试环境 - dolinux - 博客园](https://www.cnblogs.com/pengdonglin137/p/5023342.html)
- [用Qemu模拟vexpress-a9 （二） --- 搭建u-boot调试环境 - dolinux - 博客园](https://www.cnblogs.com/pengdonglin137/p/5023608.html)
- [用Qemu模拟vexpress-a9 （三）--- 实现用u-boot引导Linux内核 - dolinux - 博客园](https://www.cnblogs.com/pengdonglin137/p/5023704.html)
- [用QEMU模拟运行uboot从SD卡启动Linux - dolinux - 博客园](https://www.cnblogs.com/pengdonglin137/p/12194548.html)
- [Linux利器：QEMU！用它模拟开发板能替代真开发板？-CSDN博客](https://blog.csdn.net/weiqifa0/article/details/108878062)


