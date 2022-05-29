## Partitioning Strategy
Three parts are necessary for this to work: the UEFI 
firmware, the EFI partition (boot partition `/boot`), and 
the actual Linux system (root filesystem `/`).

Because the Raspberry Pi does not support sane UEFI like 
`x86_64` platform does out of the box, we need to place the 
UEFI firmware inside the SD card, for the Raspberry Pi 
bootloader to load up the UEFI firmware first, before 
letting the UEFI to boot the actual operating system.

### Raspberry Pi 3B+ (No GPT Support)
On older broads like Raspberry Pi 3B+, the builtin 
bootloader is too stupid to recognize GPT (GUID Partition 
Table), and only supports MBR (Master Boot Record).
Trying to boot from GPT would result in absolutely nothing 
happening (red light would light on to indicate power, but 
green light would not; indicating no disk read).
Therefore, a hack named "hybrid MBR" is used to allow the 
co-existence of both MBR and GPT on the same disk (not to 
be confused with "protective MBR", which merely protects 
GPT from MBR-only programs overwriting it).

The idea here is to create three partitions in GPT + 
hybrid MBR.
1. UEFI firmware `MBR: W95 FAT16 (LBA)` 
`GPT: Microsoft basic data`
2. EFI partition `GPT: EFI system partition`
3. Linux system `GPT: Linux filesystem`

We register only the first partition in hybrid MBR for the 
stupid bootloader to understand and load the UEFI firmware, 
while other partitions stay untouched.

### Raspberry Pi 4B (Supports GPT w/ new EEPROM)
On newer broads like Raspberry Pi 4B that uses EEPROM to 
store the system's bootloader, it is possible to update the 
bootloader to understand GPT, so the above hack is not 
needed and the firmware can directly reside in the 
partition. However, the latest EEPROM is necessary for GPT 
support, otherwise the bootloader will remain stupid and 
exhibit nothing.

The idea is to create two partitions in GPT.
1. EFI partition (with UEFI firmware)
`GPT: EFI system partition`
2. Linux system `GPT: Linux filesystem`
