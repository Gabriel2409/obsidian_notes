#linux 

## Context

This can happen after any system update where a **DKMS module** (nvidia, xpadneo, virtualbox, etc.) fails to build for the new kernel. The post-install hooks fail silently and the boot entry ends up broken.

**Symptoms:**
- Linux missing from bootloader
- `bootctl status` shows "default boot entry is broken"
- System only boots to firmware or Windows

---

## Identify Your System Layout

Before mounting anything, identify your partitions:

```bash
lsblk -f
fdisk -l
```

`lsblk -f` shows filesystem types. `fdisk -l` shows the **Type** column which explicitly labels partitions as "EFI System" or "Linux filesystem", making identification unambiguous.

Look for:
- **EFI System** / FAT32 (~500MB-1GB) — this is your EFI partition
- **Linux filesystem** / btrfs — this is your root

Then check your fstab to know the correct mount points:

```bash
cat /mnt/etc/fstab   # before chroot
# or
cat /etc/fstab       # inside chroot
```

This tells you whether your EFI partition is mounted on `/boot` or `/boot/efi`. **Do not assume — always check fstab first.**

> **Always match the mount point from fstab — do not assume.**

---

## Recovery Steps

### 1. Boot from Arch Live USB

Press F11/F12 at startup to open the boot menu and select the USB.

### 2. Connect to WiFi

```bash
iwctl
station wlan0 connect "YourWifiName"
exit
```

### 3. Mount and chroot

Replace `<root-partition>` and `<efi-partition>` with what you found in `lsblk -f`. The EFI mount point must match your fstab (`/boot` or `/boot/efi`).

```bash
mount -o subvol=@ /dev/<root-partition> /mnt
mkdir -p /mnt/<efi-mount-point>
mount /dev/<efi-partition> /mnt/<efi-mount-point>
arch-chroot /mnt
```

### 4. Fix the broken DKMS package

```bash
dkms status                      # identify the broken module
pacman -R <broken-dkms-package>  # remove it
pacman -S linux linux-headers    # reinstall kernel
bootctl install                  # reinstall bootloader entries
```

### 5. Verify before rebooting

- **vmlinuz-linux** — the Linux kernel itself, the core of the OS
- **initramfs-linux.img** — a temporary mini filesystem loaded at boot to mount the real root partition

Both must be present where the bootloader can find them (check fstab to know where).

```bash
ls /boot/vmlinuz-linux           # kernel must be here
ls /boot/loader/entries/         # boot entry must exist
dkms status                      # all modules should show "installed"
```

### 6. Reboot

```bash
exit
reboot
```

---

## Prevention

**After every update, check DKMS:**
```bash
dkms status
```

**Take snapshots regularly:**
```bash
sudo btrfs subvolume snapshot / /.snapshots/root/root_snapshot_$(date +%Y-%m-%d-%H%M)
sudo btrfs subvolume snapshot /home /.snapshots/home/home_snapshot_$(date +%Y-%m-%d-%H%M)
```

**Back up the EFI partition** (replace `<efi-partition>` with yours from `lsblk -f`):
```bash
sudo dd if=/dev/<efi-partition> of=~/boot_backup_$(date +%Y-%m-%d).img
```
