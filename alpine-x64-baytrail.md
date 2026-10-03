# Prepearing x64 ISO with 32-bit UEFI support
1. Download alpine x86 extended ISO
2. Download alpine x64 standard ISO
3. Put (replace) `boot/grub/efi.img`, `efi/boot/bootia32.efi` and boot partition from x86 ISO into x64 ISO via PowerISO.
4. Extract `grub-efi` apk from x86 ISO and put it's dir `usr/lib/grub/i386-efi` inside the root of the x64 ISO.
# Installing
1. Boot ISO (Ventoy is not supported), login as `root`
2. Run `setup-alpine` till the disk setup, then press `Ctrl`+`C`
```sh
apk add cfdisk e2fsprogs
# Assuming system disk is /dev/sda
# Select GPT. Create 256M "EFI System", other for "Linux filesystem" (I don't like swap). Write and quit.
cfdisk /dev/sda
modprobe fat
modprobe vfat
modprobe ext4
# Assuming sda1 is EFI partition, sda2 is linux partition
mkfs.vfat /dev/sda1
mkfs.ext4 /edv/sda2
# Now mount the disk
mount /dev/sda2 /mnt
mkdir -p /mnt/boot/efi
mount /dev/sda1 /mnt/boot/efi
# Install Alpine
setup-disk /mnt -m sys
# Let's fix EFI GRUB
for dir in dev proc sys; do mount --bind /$dir /mnt/$dir; done
chroot /mnt /bin/sh
# Mount the installation media into /tmp (sr0 for me)
mount -o ro /dev/sr0 /tmp
# Copy i386-efi dir we prepeared earlier
cp -r /tmp/i386-efi /usr/lib/grub
# Use nano to configure /etc/default/grub for you (I'll add vmxnet3 module)
apk add nano
nano /etc/default/grub
# Continue fixing GRUB
apk add grub-efi efibootmgr
grub-install --target=i386-efi --efi-directory=/boot/efi --bootloader-id=alpine -—removable
grub-mkconfig -o /boot/grub/grub.cfg
# Clean up and reboot
umount /tmp
exit
for dir in dev proc sys; do umount /mnt/$dir; done
umount /mnt/boot/efi
umount /mnt
reboot
```
# Configuring
Repositories, sudo
```sh
su
# Uncomment community repo
nano /etc/apk/repositories
apk update
apk upgrade
apk add sudo bash
EDITOR=nano visudo
exit
```
Install newer kernel
```sh
sudo apk add linux-stable
sudo reboot
```
