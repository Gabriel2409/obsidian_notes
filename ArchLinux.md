#linux

### Installation guide
https://wiki.archlinux.org/title/Installation_guide

- Download the mirror, for ex: `wget https://mirror.thekinrar.fr/archlinux/iso/2024.07.01/archlinux-2024.07.01-x86_64.iso`
- Follow the instructions from https://archlinux.org/download/. Most importants are:
	- Download checksums and sig file from the archlinux site, **NOT the mirror**
- Check the b2sum, the sha256sum and the fingerprint with  `gpg --fingerprint pierre@archlinux.org` and make sure they match what you see on the site

- Find location of your usb with `lsblk` and unmount it if necessary, for ex `sudo umount /dev/sda1`, then write with `dd`
