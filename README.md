


## Session requirements

```shell
sudo apt-get install gparted
sudo apt-get install qemu-system
sudo apt-get install device-tree-compiler
```

```shell
git clone git://git.denx.de/u-boot.git
```
----
## Keywords

1. bootloader.
2. ROM.
3. TPL.
4. Device tree.
5. U-Boot.
6. Labels.
7. SRAM
8. DRAM
9. uImage
10. zImage
---
## Actions.
0. Booting Sequence.
1. Emulating RASPI3
2. Compile Device tree.
3. Moving from Bootloader to Kernel.
4. configure bootloader to run over SOC.
5. Booting Sequence.
6. Building U-Boot.
7. installing U-Boot.
8. Using U-Boot.
---
## 1. Understanding Booting Sequence.
---

![[[3. Linux Startup.png](https://github.com/embeddedlinuxworkshop/M2-S2/blob/main/3.%20Linux%20Startup.png)]]

#### 🕐 1.1 Important timepoints.
---

-  t1 ( *Hardware Specific* ) ▶ Power On self test ( state of hardware = undefined state  )
-  t1 &rarr; t2 ( Running inside *ROM code* ( *Vendor specific* ) ).
	- ROM code *copy SPL into SRAM*.
	- ![[4. ROM CODE.png]]

- t3: Running SPL ( Secondary program loader ) code (*Vendor specific*).
	- initializing hardware ( *DRAM* and *DRAM controller* ).

> - SPL can be found in first partition SD-CARD.


- SPL takes control 
	![[5.SPL.png]]

- t4: SPL loads bootloader from /boot partition.

---

## 2. Bootloader Mission.

---

1. Load Kernel to DRAM.
2. Pass DTB to kernel ( Details Hardware ).
3. Location + size initramfs ( optional ).
4. Gives control to kernel.
![[6. Bootloader.png]]
---

## 3. Introduction device trees.
---

#### 3.1: Device Tree for RASPI3 ( BCM2837 ).

[link]
#### 3.2 Operations on Device tree.

1. Device Tree &rarr; Binary.
```shell
dtc <name>.dts -o <name>.dtb # convert it into binary.
```

2. Binary &rarr; Device tree.
```shell
dtc -I dtb -O dts -o output.dts input.dtb
```

---

## 4. U-BOOT.
---

> - Support multiple architecture and boards.

#### Operations
---

1. Downloading u-boot.

```shell
git clone git://git.denx.de/u-boot.git
cd u-boot
```


2. Building u-boot for RASPI3.

```shell
make rpi_3_defconfig
make CROSS_COMPILE=<toolchain prefix>
```

![[7. U-boot.png]]

3. Installing U-boot ( **SD-CARD Version** )

```shell
1. Using gparted tool to create FAT16 partiotion on top of SD-CARD.
2. Installing /boot partiotion.
3. insert SD-CARD into your Hardware.
```

3. Installing U-boot on top of QEMU Emulator ( TARGET ).

source: https://github.com/ARM-software/u-boot/blob/master/doc/README.qemu-arm

```shell
qemu-system-aarch64 \
    -M raspi3b \
    -cpu cortex-a72 \
    -append "rw earlyprintk loglevel=8 console=ttyAMA0,115200" \
    -kernel u-boot.bin \
    -m 1G -smp 4 \
    -serial stdio \
    -usb -device usb-mouse -device usb-kbd \
        -device usb-net,netdev=net0 \
        -netdev user,id=net0,hostfwd=tcp::5555-:22 \
```


4. Using U-BOOT.
##### 4.1 Convert zImage into uImage ( HOST ).

```shell
mkimage -A arm64 -O linux -T kernel -C gzip -a 0x80008000 (load address) \  
-e 0x80008000 (entry point)
-n 'Linux' -d zImage (compressed image) uImage (uBoot Image)
```


#### 4.2 Loading images from flash into RAM ( inside U-BOOT ).

```shell
nand read <RAM address> 280000<nand offset> 400000 <size> # read from nand into RAM.
```

#### 4.3 Booting Kernel.
---

```shell
bootm <Kernel address> <address of ramdisk> <address of dtb>
```

---

