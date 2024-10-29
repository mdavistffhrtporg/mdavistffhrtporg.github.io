---
date: 2024-04-04T10:58:08-04:00
description: "LUKS2 & TPM2"
featured_image: "markus-spiske-unsplash-scaled.jpg"
tags: ["OSS", "Fedora"]
categories: ["OSS", "Fedora"]
title: "How to unlock a LUKS2 disk with a TPM2 chip"
---

[LUKS2](https://en.wikipedia.org/wiki/Linux_Unified_Key_Setup) encrypted partitions can be unlocked in coordination with [systemd-cryptenroll](https://www.freedesktop.org/software/systemd/man/latest/systemd-cryptenroll.html) and a [Trusted Platform Module 2.0](https://en.wikipedia.org/wiki/Trusted_Platform_Module) (TPM2) chip. TPM2 chips offer an alternative to typing in a password every time your non-Atomic and Atomic Desktop, i.e. [Fedora Silverblue](https://fedoraproject.org/atomic-desktops/silverblue/), is booted up. While this is straight forward on a non-Atomic Desktop such as [Fedora Workstation](https://fedoraproject.org/workstation/), there is little documentation on how to do this on an Atomic Desktop. Although not recommended, repeatedly typing passwords can be tedious if you are layering several packages on an a desktop Operating System (OS) like Fedora Silverblue.

# What is a Trusted Platform Module?

A TPM2 chip is a special piece of storage built into most modern PCs that work in conjunction with an [Application Programming Interface (API)](https://en.wikipedia.org/wiki/API) to store encryption secrets. Those secrets, along with secure boot, and the [Unified Extensible Firmware Interface (UEFI)](https://en.wikipedia.org/wiki/UEFI) work to ensure that the user’s machine stays secure from tampering. Along with the OS, the TPM2 chip can supplant the encryption of data on the disk, so that if a computer is cold-booted, data will stay secure, even if it is transferred to another computer. Essentially, the TPM2 chip renders data useless if there is tampering. While all may sound well with this setup, there are still weaknesses, and I prefer to manually enter a password rather than putting all of my faith in the TPM2 chip.

# Tying LUKS2 to the TPM2 chip- Atomic Desktop

The first step in tying LUKS2 to your TPM2 chip is identifying your partition numbers. This can be done using the lsblk command. Here is an example:
```
$ lsblk
NAME MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
zram0 252:0 0 8G 0 disk [SWAP]
nvme0n1 259:0 0 476.9G 0 disk
├─nvme0n1p1 259:1 0 600M 0 part /boot/efi
├─nvme0n1p2 259:2 0 1G 0 part /boot
└─nvme0n1p3 259:3 0 475.4G 0 part
└─luks-e9b6d985-2893-4e0f-8c4f-881b83a24988 253:0 0 475.3G 0 crypt /var/home
/var
/sysroot/ostree/deploy/fedora/var
/usr
/etc
/
/sysroot
```
In this case, it is nvme0n1p3 since it shows that it is tied to LUKS2.

Next, add the tpm2-tss module to your dracut configuration*:
```
$ echo "add_dracutmodules+=\" tpm2-tss \"" | sudo tee /etc/dracut.conf.d/tpm2.conf
add_dracutmodules+=" tpm2-tss "
```
Third, you will use systemd-cryptenroll to tell your system to enroll your TPM2 chip as an alternate method of decrypting your disk on boot:
```
$ sudo systemd-cryptenroll --wipe-slot tpm2 --tpm2-device auto --tpm2-pcrs "0+1+2+3+4+5+7+9" /dev/nvme0n1p3
```
Now, add the following to your _/etc/crypttab_ under your first line*:
```
tpm2-device=auto,tpm2-pcrs=0+1+2+3+4+5+7+9
```
Finally, will need to rebuild your _initramfs_ by performing the following:
```
$ rpm-ostree initramfs --enable --arg=--force-add --arg=tpm2-tss
```
# Considerations & conclusion

If have made it this far, you are probably wondering about the fact that what we are doing is all on an Atomic Desktop and how /etc/* will be affected. While this is an immutable desktop, the file modifications in /etc/*will still be saved since that directory only makes changes when a rpm-ostree rollback is performed or upgrade is performed. During a rollback, the files will be restored to their previous state, meaning that changes will be lost. During an upgrade, there is a 3-way diff/merge. This means that Atomic Desktop treats your files from each deployment as if they are own independent copy. Furthermore, the defaults of the new deployment will be merged with the current copy, so any modification will be kept. So, unless you rollback, changes will be kept.

As previously mentioned, the TPM2 chip along with secure boot do have weaknesses. One way to mitigate those weaknesses is to use a separate pieces of hardware, such as a FIDO2 key, i.e. a YubiKey. This type of token is kept separate from the system, meaning that there is an extra layer of security that requires the user’s presence in order to decrypt the disk. And, FIDO2 keys will block any further input upon a previously determined set amount of failed PIN inputs, thereby preventing a brute force attack. Please look for details on unlocking your LUKS2 disk with a FIDO2 token soon.

_This is part (1) of a (2)-part series._

_Photo by: [Markus Spiske](https://unsplash.com/@markusspiske) on [Unsplash](https://unsplash.com/)_
