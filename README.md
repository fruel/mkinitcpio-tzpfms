# mkinitcpio-tzpfms
Archlinux mkinitcpio hook to unlock encrypted ZFS filesystems using keys stored in TPM2 chips.

# Dependencies
Requires `tzpfms`:
 * https://git.sr.ht/~nabijaczleweli/tzpfms
 * https://aur.archlinux.org/packages/tzpfms

# Setup:

* Prerequisites:
  * Secure boot is enabled - see for example:
    * https://wiki.archlinux.org/title/Unified_Extensible_Firmware_Interface/Secure_Boot
    * https://wiki.cachyos.org/configuration/secure_boot_setup/
  * (optional, recommended) UKI are used to boot Linux directly without bootloader
    * With a UKI the kernel, initramfs and cmdline are signed as one unit that gets verified by the UEFI during secure boot.
    * https://wiki.archlinux.org/title/Unified_kernel_image

* Install `tzpfms`:
  ```
  yay tzpfms
  yay mkinitcpio-tzpfms
  ```
* Add the `tzpfms` hook to your mkinitcpio configuration in `/etc/mkinitcpio.conf` after the `zfs` hook:
  ```
  HOOKS=(base udev autodetect microcode kms modconf block keyboard keymap consolefont zfs tzpfms filesystems fsck)
  ```
* Bind the ZFS encryption key to the TPM2 PCRs.
  ```
  zfs-tpm2-change-key -b ./recovery.key -P sha256:7 zpcachyos
  ```
  * This replaces the previous encryption key/password. To unlock the disk without TPM in the future, you will need the `./recovery.key` file. Store it in a secure location. To unlock the ZFS filesystem if you loose access to the TPM use `cat ./recovery.key | zfs load-key zpcachyos`.
  * The example above just uses PCR register 7 which means a valid secure boot state is required for unlocking the disk. There are many other PCRs that can be used. For more information, check:
    * https://wiki.archlinux.org/title/Systemd-cryptenroll#Trusted_Platform_Module
    * https://wiki.archlinux.org/title/Trusted_Platform_Module#Accessing_PCR_registers
