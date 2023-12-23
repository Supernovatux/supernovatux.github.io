---
layout: post
title: On Dualboot + TPM + LUKS + Legion 5 Pro 16ACH6H
lead: A very quick guide.
---

Hello,

This is a short note on how I set up my Legion 5 pro with Archlinux + Windows 11 Dualboot.

# Procedure 
- Read all the instructions and the files provided before proceeding
- My partition structure look like. Other than the once below I have my windows partition, windows recovery partition, and Linux swap partition.

```bash
> sudo inxi -p
Partition:
  ID-1: / size: 742.84 GiB used: 550.05 GiB (74.0%) fs: btrfs dev: /dev/dm-1
  ID-2: /boot size: 1022 MiB used: 13.3 MiB (1.3%) fs: vfat
    dev: /dev/nvme0n1p5
  ID-3: /efi size: 256 MiB used: 207.1 MiB (80.9%) fs: vfat
    dev: /dev/nvme0n1p1
  ID-4: /home size: 742.84 GiB used: 550.05 GiB (74.0%) fs: btrfs
    dev: /dev/dm-1
  ID-5: swap-1 size: 9.75 GiB used: 0 KiB (0.0%) fs: swap dev: /dev/dm-0
```
-  Install [Archlinux with luks](https://wiki.archlinux.org/title/dm-crypt/Encrypting_an_entire_system) on the `root` partition. Use a [systemd based initramfs](https://wiki.archlinux.org/title/Dm-crypt/System_configuration#mkinitcpio).

```bash
> cat /etc/mkinitcpio.conf
# Note this a sample. Modify your parameters accordingly don't use this file.
MODULES=(amdgpu amd_pstate)
HOOKS=(base systemd autodetect modconf sd-encrypt block filesystems keyboard fsck)
```

- Have a large (at least 256MB)  `efi` partition mounted at `/efi` . We will place our kernel in `efi` partition so make sure its big enough. You can use a separate `boot partition` but it makes this slightly more complicated
- Use [Unified kernel Image](https://wiki.archlinux.org/title/Unified_kernel_image)
- Make sure to modify your `/etc/mkinitcpio.d/linux-zen.preset`  so the the UKI is generated to the `/efi/EFI/Linux/archlinux-linux-zen.efi` .

``` bash
> cat /etc/mkinitcpio.d/linux-zen.preset 
# mkinitcpio preset file for the 'linux-zen' package

ALL_config="/etc/mkinitcpio.conf"
ALL_kver="/boot/vmlinuz-linux-zen"

PRESETS=('default' 'fallback' 'vfio')

default_config="/etc/mkinitcpio.conf"
#default_image="/boot/initramfs-linux-zen.img"
default_uki="/efi/EFI/Linux/archlinux-linux-zen.efi"
default_options="--splash=/usr/share/systemd/bootctl/splash-arch.bmp --cmdline /etc/kernel/cmdline"


fallback_config="/etc/mkinitcpio_fallback.conf"
#fallback_image="/boot/initramfs-linux-zen-fallback.img"
fallback_uki="/efi/EFI/Linux/archlinux-linux-zen-fallback.efi"
fallback_options="-S autodetect --cmdline /etc/kernel/cmdline_fallback"
```

- Use [systemd-boot](https://wiki.archlinux.org/title/systemd-boot)
- Try to boot and see if you can unlock the root partition. Note that you must use systemd based initramfs. You can use `/etc/crypttab.initramfs` to make kernel command line more compact

```bash
> cat /etc/crypttab.initramfs 
CRYPT_ROOT /dev/nvme0n1p6 none discard,timeout=180
LUKS_SWAP /dev/nvme0n1p7 none discard
> cat /etc/kernel/cmdline
root=UUID=90f8214f-7vc3-408a-a00a-2ec2c47ffcdd rw rootflags=subvol=@ nowatchdog nmi_watchdog=0 mitigations=off resume=/dev/mapper/LUKS_SWAP quiet bgrt_disable
```

- Use [sbctl](https://wiki.archlinux.org/title/Unified_Extensible_Firmware_Interface/Secure_Boot#Implementing_Secure_Boot)  to create and sign the efi binaries. Enroll with Microsoft's keys `sudo sbctl enroll-keys -m` .I use [sbctl-initcpio-post-hook ]() for automatic signing after each kernel update.
- Now turn on secure boot in bios and see if everything works.
- If everything works enroll `tpm` state into the `LUKS root volume` . I use a file as in below for this purpose

```bash
#!/bin/bash
echo "This script will enroll latest TPM2 PCR values into the LUKS2 header of the root and swap partition. You will need to rerun this script everytime you upgrade/modify your kernel"

#/etc/cryptsetup-keys.d/CRYPT_ROOT.key contains the secret to unlock the LUKS volume
#That file can be created using
#print "<password>" | sudo tee /etc/cryptsetup-keys.d/CRYPT_ROOT.key
# else you can remove the --unlock-key-file=/etc/cryptsetup-keys.d/CRYPT_ROOT.key part and let the program pront for the passphrase
echo "Wiping nvme0n1p7 tpm2 slots"
sudo systemd-cryptenroll /dev/nvme0n1p7 --wipe-slot=tpm2 
echo "Wiping nvme0n1p6 tpm2 slots"
sudo systemd-cryptenroll /dev/nvme0n1p6 --wipe-slot=tpm2
echo "Enrolling nvme0n1p6 with tpm2 pcr 4+7+14"
sudo systemd-cryptenroll --tpm2-device=auto --tpm2-pcrs=4+7+14 /dev/nvme0n1p6 --unlock-key-file=/etc/cryptsetup-keys.d/CRYPT_ROOT.key
echo "Enrolling nvme0n1p7 with tpm2 pcr 4+7+14"
sudo systemd-cryptenroll --tpm2-device=auto --tpm2-pcrs=4+7+14 /dev/nvme0n1p7 --unlock-key-file=/etc/cryptsetup-keys.d/CRYPT_ROOT.key 
```

- Everything should work in theory :)

