---
layout: post
title: "Fix UEFI Ubuntu's Grub in Boot Manager"
---

I bought a HP laptop equipped with Windows 8.1, but it would jump over Ubuntu's Grub after I install it.

Since I could still use BIOS to enter into the disk boot manually and I have /boot/efi in my Ubuntu, it demonstrates that my Ubuntu 14 is fine with the EFI. So I decide to cheat the evil and stupid UEFI by replacing Microsoft's .efi file with Grub .efi file.

1. `sudo cp /boot/efi/EFI/Microsoft/Boot/bootmgfw.efi /boot/efi/EFI/Microsoft/Boot/bootmgfw.efi.old`
- `sudo cp /boot/efi/EFI/ubuntu/grubx64.efi /boot/efi/EFI/Microsoft/Boot/bootmgfw.efi`

Another more suitable way is to use "efibootmgr" `sudo efibootmgr -o 0005,0004` to specify the order explicitly
