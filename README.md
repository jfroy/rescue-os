# Rescue USI

This is a very simple mkosi configuration to build a rescue Unified System Image[^1][^2][^3][^4] for my Talos nodes. The image contains a barebones [Arch Linux](https://archlinux.org/) system, the [OpenZFS](https://openzfs.github.io/openzfs-docs/index.html) [`zfs-dkms-staging-git`](https://aur.archlinux.org/packages/zfs-dkms-staging-git) and [`zfs-utils`](https://aur.archlinux.org/zfs-utils.git) Arch AUR packages, and the [mstflint](https://aur.archlinux.org/packages/mstflint)[^5] Arch AUR package.

The resulting USI is a bootable UEFI program that will be signing for secure boot. It can be copied to an EFI system partition or loaded from the network.

## How to build

Run `mkosi --secure-boot` as root. This can be done in a user namespace:

```shell
unshare --map-auto --map-current-user --setuid 0 --setgid 0
mkosi --secure-boot
```

The secure boot UKI signing key and certificate must be provided.

## PXE boot

I use the following barebones dnsmasq config to PXE boot the resue USI:

```netboot.conf
enable-tftp
tftp-root=/srv/tftp
pxe-service=X86-64_EFI,"PXELINUX (EFI)",arch-usi.efi
pxe-service=7,"PXELINUX (EFI)",arch-usi.efi
```

On a Unifi gateway system, that configuration can be stored as `/run/dnsmasq.conf.d/netboot.conf`, the USI copied to `/srv/tftp`. `dnsmasq` must be restarted after adding the config file (e.g. `kill $(cat /run/dnsmasq.pid)`). This setup will not survive gateway reboots or system upgrades.

[^1]: <https://overhead.neocities.org/blog/build-usi-mkosi/>
[^2]: <https://swsnr.de/archlinux-rescue-image-with-mkosi/>
[^3]: <https://kairos.io/docs/architecture/trustedboot/>
[^4]: <https://www.mauromorales.com/2024/06/27/a-new-dawn-for-secure-linux-in-untrusted-environments/>
[^5]: <https://github.com/Mellanox/mstflint>
