## Instructions

### Update EEPROM (Raspberry Pi 4B)
Before doing any partitioning to the SD card, update the 
EEPROM first. Download the official EEPROM image 
[here](https://github.com/raspberrypi/rpi-eeprom/releases).

To unzip it and flash the image, run:
```sh
7z x rpi-boot-eeprom-recovery-*-sd.img.zip
cp rpi-boot-eeprom-recovery-*-sd.img /dev/mmcblk1
```

### Download UEFI Firmware
#### EDK2 UEFI Releases
Raspberry Pi 3B+: https://github.com/pftf/RPi3/releases  
Raspberry Pi 4B: https://github.com/pftf/RPi4/releases

#### Note
The popularly recommended tool `rpi-eeprom-update` is 
pitifully useless on any systems other than Raspbian. 
Even if you managed to get it running, it would not 
recognize the board and abort the update anyway.

### Partition the SD Card
We will use `gdisk` to proceed with partitioning.

#### Note
The popular tool `fdisk` defaults to MBR/DOS partitions, 
which is not ideal here; and while the tool `parted` 
defaults to GPT, it does not autocomplete the partition 
size which is a giant pain.
That leaves `gdisk` the only suitable candidate.

#### Step 1: Create New GPT Partition Table
```
GPT fdisk (gdisk) version 1.0.9

...

Command (? for help): o
This option deletes all partitions and creates a new protective MBR.
Proceed? (Y/N): y
```

#### Step 2: Create Standalone FAT16 Partition for UEFI Firmware and Set up Hybrid MBR (if necessary)
```
Command (? for help): n
Partition number (1-128, default 1): <ENTER>
First sector ... : <ENTER>
Last sector ... : +256MiB
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): 0700
Changed type of partition to 'Microsoft basic data'

Command (? for help): r

Recovery/transformation command (? for help): h

WARNING! Hybrid MBRs ...

Type from one to three GPT partition numbers, separated by spaces, to be
added to the hybrid MBR, in sequence: 1
Place EFI GPT (0xEE) partition first in MBR (good for GRUB)? (Y/N): n

Creating entry for GPT partition #1 (MBR partition #1)
Enter an MBR hex code (default 07): 0e
Set the bootable flag? (Y/N): y

Unused partition space(s) found. Use one to protect more partitions? (Y/N): n

Recovery/transformation command (? for help): m
```

#### Step 3: Create Partition for EFI System Partition
```
Command (? for help): n
Partition number (2-128, default 2): <ENTER>
First sector ... : <ENTER>
Last sector ... : +256MiB
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): ef00
Changed type of partition to 'EFI system partition'
```

#### Step 4: Create Partition for Linux Filesystem
```
Command (? for help): n
Partition number (3-128, default 3): <ENTER>
First sector ... : <ENTER>
Last sector ... : <ENTER>
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): <ENTER>
Changed type of partition to 'Linux filesystem'
```

#### Step 5: Write to Disk
```
Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): y
OK; writing new GUID partition table (GPT) to ...
```


### Make Filesystem on Partitions
While doing so, it's a good habit to label the partitions. 
It makes mounting much more easier.

#### Note
Avoid lowercase labels because `mkfs.fat` said they "might 
not work properly on some systems". However, you can always 
change partition labels using `fatlabel` or `e2label`.

#### Raspberry Pi 3B+
```sh
mkfs.fat -F 16 -n PI3_FIRMWARE /dev/mmcblk1p1
mkfs.fat -F 32 -n PI3_BOOT /dev/mmcblk1p2
mkfs.ext4 -L PI3_ROOTFS /dev/mmcblk1p3
```

#### Raspberry Pi 4B
```sh
mkfs.fat -F 32 -n PI4_BOOT /dev/mmcblk1p1
mkfs.ext4 -L PI4_ROOTFS /dev/mmcblk1p2
```

### Extract the UEFI Firmware

#### Raspberry Pi 3B+
```
mount /dev/disk/by-label/PI3_ROOTFS /mnt
mkdir /mnt/boot
mount /dev/disk/by-label/PI3_BOOT /mnt/boot
mkdir /mnt/boot/firmware
mount /dev/disk/by-label/PI3_FIRMWARE /mnt/boot/firmware
7z x RPi3_UEFI_Firmware_v*.zip /mnt/boot/firmware
umount -R /mnt
sync
```

#### Raspberry Pi 4B
```
mount /dev/disk/by-label/PI4_ROOTFS /mnt
mkdir /mnt/boot
mount /dev/disk/by-label/PI4_BOOT /mnt/boot
7z x RPi4_UEFI_Firmware_v*.zip /mnt/boot
umount -R /mnt
sync
```

### All Done!
Now if nothing goes wrong, simply install the system 
following the usual UEFI procedures. Just don't touch the 
pre-configured partitions. If you can, do yourself a flavor 
by building a custom installer ISO with wireless networking 
and sshd enabled. It's a simple pleasure to preform the 
install headlessly.
