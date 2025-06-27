---
layout: post
title: "Automatically decrypt LUKS devices during boot via network using initramfs hooks by Clevis and Tang on UEFI Arch Linux with LVM2"
categories: [IT]
image: images/2025-06-27-automatically-decrypt-luks-devices-during-boot-via-network-using-initramfs-hooks-by-clevis-and-tang-on-uefi-arch-linux-with-lvm2/cover.jpg
excerpt: "sdsdsd"
---

![Cover]({{ site.baseurl }}/images/2025-06-27-automatically-decrypt-luks-devices-during-boot-via-network-using-initramfs-hooks-by-clevis-and-tang-on-uefi-arch-linux-with-lvm2/cover.jpg)

During boot, [Clevis](https://github.com/latchset/clevis) runs on your computer and connects to a [Tang](https://github.com/latchset/tang) server on your network to perform the unlocking. Most of the configuration is done on the Clevis side, as Tang servers are fairly standard. For a Tang server, I recommend [Padhi's Docker container](https://github.com/padhi-homelab/docker_tang). Just spawn it up, take note of your port and IP, and make sure to persist (and take care) the directory used as its database, as it serves as the key for your LUKS system.

For this setup to work, it's important to install the encrypted system without encrypting the `/boot` partition. In this example, I install UEFI Arch Linux on the entire `/dev/nvme0n1` (256GB) device, reserving 1GB for an unencrypted `/boot` partition. Then, I create an encrypted LUKS container with 8GB allocated for swap and the remaining space for the root (`/`) partition, without separating `/home`. We will divide the LUKS container into different partitions using an LVM2 setup.

At the end, it will look like this:
```
NAME           MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINTS
nvme0n1        259:0    0 223.6G  0 disk  
├─nvme0n1p1    259:1    0  1000M  0 part  /boot
└─nvme0n1p2    259:2    0 222.6G  0 part  
  └─cryptroot  253:0    0 222.6G  0 crypt 
    ├─vg0-swap 253:1    0     8G  0 lvm   [SWAP]
    └─vg0-root 253:2    0 214.6G  0 lvm   /
```

Here are the basic installation commands:
```bash
# 1. Verify UEFI mode
ls /sys/firmware/efi && echo "UEFI boot!"

# 2. Wipe the disk
wipefs -a /dev/nvme0n1
sgdisk --zap-all /dev/nvme0n1

# 3. Partition the disk
parted /dev/nvme0n1 -- mklabel gpt
parted /dev/nvme0n1 -- mkpart ESP fat32 1MiB 1001MiB
parted /dev/nvme0n1 -- set 1 esp on
parted /dev/nvme0n1 -- mkpart primary 1001MiB 100%

# 4. Format the ESP as FAT32
mkfs.fat -F32 /dev/nvme0n1p1

# 5. Create and open LUKS2 container
cryptsetup luksFormat --type luks2 /dev/nvme0n1p2
cryptsetup open /dev/nvme0n1p2 cryptroot

# 6. Create LVM inside the LUKS device
pvcreate /dev/mapper/cryptroot
vgcreate vg0 /dev/mapper/cryptroot
lvcreate -L 8G vg0 -n swap
lvcreate -l 100%FREE vg0 -n root

# 7. Format and enable swap
mkfs.ext4 /dev/vg0/root
mkswap /dev/vg0/swap

# 8. Mount the filesystems
mount /dev/vg0/root /mnt
mkdir /mnt/boot
mount /dev/nvme0n1p1 /mnt/boot
swapon /dev/vg0/swap

# 9. Install base system
pacstrap /mnt base linux linux-firmware lvm2 grub efibootmgr

# 10. Generate fstab
genfstab -U /mnt >> /mnt/etc/fstab

# 11. Chroot into the system
arch-chroot /mnt

# 12. Set timezone and locale (example: São Paulo)
ln -sf /usr/share/zoneinfo/America/Sao_Paulo /etc/localtime
hwclock --systohc
echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
echo "KEYMAP=br-abnt2" > /etc/vconsole.conf

# 13. Set hostname
echo myhostname > /etc/hostname
echo "127.0.0.1 localhost" >> /etc/hosts
echo "::1       localhost" >> /etc/hosts
echo "127.0.1.1 myhostname.localdomain myhostname" >> /etc/hosts

# 14. Set root password
passwd
```

Then, **before the first reboot**, we start setting up the initramfs with networking, Clevis, LUKS, and LVM2. After that, we configure GRUB to work with this setup.

Install the required packages:
```bash
pacman -S --needed clevis networkmanager curl git base-devel
```

Install `yay`:
```bash
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
cd ..
rm -rf yay
```

Install the mkinitcpio Clevis hook:
```bash
yay -S mkinitcpio-clevis-hook
```

Update `/etc/mkinitcpio.conf` for a Clevis + LUKS + LVM setup. Add `curl` and `ip` to `BINARIES`, and use the following order in `HOOKS`, highlighting `net`, `clevis`, `encrypt`, and `lvm2`:
```
[...]
BINARIES=(/usr/bin/curl /usr/bin/ip)
[...]
HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block net clevis encrypt lvm2 filesystems fsck)
[...]
```

The `net` hook is responsible for setting up networking during early boot.

**Note:** As pointed out in Jelle's post (URL at the bottom), there's no timeout in the `net` hook, so it won't fall back to the `encrypt` hook if it fails. He recommends editing the `/usr/lib/initcpio/hooks/net` file and replacing the `ipconfig` call with `ipconfig -t 30 "ip=${ip}"`.

Then, rebuild the initramfs:
```bash
mkinitcpio -P
```

Get the UUID of the encrypted partition:
```bash
blkid -s UUID -o value /dev/nvme0n1p2
```

You'll need this UUID to configure GRUB. Edit `/etc/default/grub` so your `GRUB_CMDLINE_LINUX` looks like this (with your UUID, of course):
```
GRUB_CMDLINE_LINUX="cryptdevice=UUID=c6d3b27d-049d-40aa-8982-91e0e6d1959a:cryptroot root=/dev/vg0/root ip=:::::eth0:dhcp"
```

Note the `ip=:::::eth0:dhcp` part. While your network interface may have a different name, `eth0` is commonly used during early boot. This setting is important to ensure network availability during the unlock process.

Now, install and configure GRUB for UEFI:
```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

Finally, bind Clevis to the Tang server:
```bash
clevis luks bind -d /dev/nvme0n1p2 tang '{"url":"http://YOUR_TANG_SERVER_IP:YOUR_TANG_SERVER_PORT"}'
```

It will connect to the Tang server and ask for your LUKS password. If everything goes well, you can now reboot, and Clevis/Tang should handle unlocking your device automatically.

To exit and reboot:
```bash
exit
umount -R /mnt
swapoff -a
reboot
```

- Thanks to Jelle van der Waa for [his blog post](https://vdwaa.nl/arch-clevis-tang.html), which pointed to the importance of the `net` initramfs hook, his post may also serve as a useful reference. Also, thanks to the author of [this blog post](https://www.ogselfhosting.com/index.php/2023/12/25/tang-clevis-for-a-luks-encrypted-debian-server/), which I also used as a reference. Unfortunately, the website is now offline and I couldn't get his name.
- *Cover picture generated using Midjourney.*
