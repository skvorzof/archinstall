## Partition
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
