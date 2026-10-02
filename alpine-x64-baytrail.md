# Prepearing x64 ISO with 32-bit UEFI support
1. Download alpine x86 extended ISO
2. Download alpine x64 standard ISO
3. Put (replace) `boot/grub/efi.img`, `efi/boot/bootia32.efi` and boot partition from x86 ISO into x64 ISO via PowerISO.
4. Extract `grub-efi` apk from x86 ISO and put it's dir `usr/lib/grub/i386-efi` inside the root of the x64 ISO.
