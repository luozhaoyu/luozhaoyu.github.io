---
layout: post
title: "Fix UEFI Ubuntu's Grub in Boot Manager"
categories: blog
---

I bought a HP laptop equipped with Windows 8.1, but it would jump over Ubuntu's Grub after I install it.

Since I could still use BIOS to enter into the disk boot manually and I have /boot/efi in my Ubuntu, it demonstrates that my Ubuntu 14 is fine with the EFI. So I decide to cheat the evil and stupid UEFI by replacing Microsoft's .efi file with Grub .efi file.

1. `sudo cp /boot/efi/EFI/Microsoft/Boot/bootmgfw.efi /boot/efi/EFI/Microsoft/Boot/bootmgfw.efi.old`
- `sudo cp /boot/efi/EFI/ubuntu/grubx64.efi /boot/efi/EFI/Microsoft/Boot/bootmgfw.efi`
- add these codes to `/etc/grub.d/40_custom`, attention, should copy resource id from `/boot/grub/grub.cfg`


        menuentry 'Modified Origin Windows Boot Manager (on /dev/sda2)' --class windows --class os $menuentry_id_option 'osprober-efi-F282-E2AF' {
            insmod part_gpt
            insmod fat
            set root='hd0,gpt2'
            if [ x$feature_platform_search_hint = xy ]; then
              search --no-floppy --fs-uuid --set=root --hint-bios=hd0,gpt2 --hint-efi=hd0,gpt2 --hint-baremetal=ahci0,gpt2  F282-E2AF
            else
              search --no-floppy --fs-uuid --set=root F282-E2AF
            fi
            chainloader /EFI/Microsoft/Boot/bootmgfw.efi.old
        }


- `update-grub2`

Another more suitable way is to use "efibootmgr" `sudo efibootmgr -o 0005,0004` to specify the order explicitly
