## Internet
1. iwctl
2. station wln0 connect gate
3. ping ya.ru

## Partition
- lsblk, cfdisk
- nvme0n1p4 /efi (Windows boot EFI) 
- nvme0n1p5 /boot
- nvme0n1p6 /

1. mkfs.ext4 -L boot nvme0n1p5
2. mkfs.btrfs -L arch nvme0n1p6

#### Mount
1. mount /dev/disk/by-label/arch /mnt
2. btrfs su cr /mnt/@
3. btrfs su cr /mnt/@home
4. umount /mnt
5. mount -o noatime,compress=lzo,ssd,discard=async,space_cache=v2,subvol=@ /dev/nvme0n1p6 /mnt
6. mkdir /mnt/{boot,home}
7. mount -o noatime,compress=lzo,ssd,discard=async,space_cache=v2,subvol=@home  /dev/nvme0n1p6 /mnt/home
8. mount  /dev/nvme0n1p5 /mnt/boot
9. mkdir /mnt/boot/efi
10. mount /dev/nvme0n1p4 /mnt/boot/efi

## Install
1. pacman -Sy reflector
2. cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.bak
3. reflector --latest 6 --sort rate --download-timeout 100 --save /etc/pacman.d/mirrorlist
4. pacstrap -K /mnt base base-devel linux linux-firmware intel-ucode nano
5. genfstab -U /mnt >> /mnt/etc/fstab

#### Setup
1. arch-chroot /mnt
2. ln -sf /usr/share/zoneinfo/Europe/Moscow /etc/localtime
3. hwclock --systohc
4. nano /etc/locale.gen
5. locale-gen
6. echo "LANG=en_US.UTF-8" >> /etc/locale.conf
7. echo arch > /etc/hostname
8. echo "127.0.0.1 localhost" >> /etc/hosts
9. passwd
10. nano /etc/pacman.conf ([multilib], ParallelDownloads)
11. pacman -Sy
12. pacman -S grub grub-btrfs efibootmgr os-prober networkmanager git bluez bluez-utils gdm gnome timeshift
13. mkinitcpio -P
14. nano /etc/sudoers (%wheel ALL=(ALL:ALL) ALL)
15. useradd -mg users -G wheel oglu
16. passwd oglu
17. systemctl enable NetworkManager
18. systemctl enable bluetooth
19. systemctl enable gdm
20. grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=ARCH
21. grub-mkconfig -o /boot/grub/grub.cfg
22. Ctrl + D
23. umount -R /mnt && reboot

## Nvidia
1. sudo pacman -S nvidia
2. Remove kms from the HOOKS array in /etc/mkinitcpio.conf
3. Add MODULES=(nvidia, nvidia_modeset, nvidia_uvm nvidia_drm)
4. sudo echo "options nvidia NVreg_PreserveVideoMemoryAllocations=1" >> /etc/modprobe.d/nvidia.conf
5. sudo nano /etc/default/grub -> GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet page_alloc.shuffle=1 split_lock_detect=off nvidia_drm.fbdev=1 nvidia_drm.modeset=1"
6. sudo mkinitcpio -P
7. sudo grub-mkconfig -o /boot/grub/grub.cfg
8. systemctl status -> sudo systemctl enable nvidia-suspend.service
9. systemctl status -> sudo systemctl enable nvidia-hibernate.service
10. systemctl status -> sudo systemctl enable nvidia-resume.service
11. reboot
