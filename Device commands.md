#linux 


Find devices in `/dev`, running `ls -l` shows different types of devices: 

- Block devices: Devices that transfer data in blocks (chunks of data) like hard drives, SSDs, and USB drives.
- Character Devices: Devices that transfer data character by character, such as keyboards, mice, serial ports, and terminals
- Other devices: socket devices, pipe devices, etc. 


Before the date we can see major and minor device numbers. Similar devices have the same major number, for ex sda1 and sda2: 
- **Major Number**: This number identifies the driver associated with the device. It's like a category that tells the kernel which driver to use for a particular device.
- **Minor Number**: This number is used by the driver to identify a specific device it controls within that category. For example, multiple hard drives or partitions managed by the same driver.


In addition to the file like description in `/dev`, we have the `sysfs` interface  in `/sys/devices`. 

Find information about a device, including its path in `/sys/` with `udevadm info --query=all --name=/dev/nvme0n1` (`udevadm` queries the `systemctl-udevd` daemon)
Path will look like `/devices/pci0000:00/0000:00:01.2/0000:02:00.0/nvme/nvme0/nvme0n1` (`/sys` is ommitted). Go into the path and do `cat dev` to see major and minor device numbers

- `mount` allows you to see all currently mounted devices
- you can also use `journalctl -k` to find a device
- when unsure how to find a service: `systemctl list-units --type=service | grep -i udev`
- `cat /proc/devices` shows block and character devices for which system has drivers. 

The motherboard is not a "device" in the sense that it doesn’t have a major number. It's a platform that hosts various devices (like the CPU, RAM, USB controllers, etc.).
Specific devices like your NVIDIA GPU are not represented in `/proc/devices` by their specific names. Instead, the GPU is managed by a specialized driver (e.g., the NVIDIA driver) that would have its major and minor numbers for the various components it interacts with, like the framebuffer or rendering pipeline.
### dd command

- useful when working with block devices
- Read from an input file/stream and writes to output file/stream, possibly with encofing conversion. 
- For ex you can process a chunk of data, ignore the middle and process the end

```bash
dd if=/dev/zero of=new_file bs=1026 count=1
# if = input file
# of = output file
# bs = block size
# count = nb of blocks to copy

# for ex, when copying the arch iso media: 
dd bs=4M \ 
if=path/to/archlinux-version-x86_64.iso \ 
of=/dev/disk/by-id/usb-My_flash_drive 
conv=fsync \ 
oflag=direct \ 
status=progress \
```
