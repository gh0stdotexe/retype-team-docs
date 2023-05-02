Original Guide: \
[How to disable your firewall in rescue mode ?](https://datacadamia.com/os/linux/firewalld_rescue_mode "How to disable your firewall in rescue mode ?")

---

When working with firewalls such as [UFW](https://app.nuclino.com/t/b/d132b0ee-ff34-42e5-916e-641cb90cd045 "Nuclino"), the unexpected can happen and you can be locked out of your server.

Many providers provide a rescue mode that permits you to get access back to your disk, usually called a "rescue mode".

This how-to shows you how to disable your firewall (but you may use it to perform a number of other maintenance operations).

---

## Reboot Server in Rescue Mode

To reboot your VPS in rescue mode, you should go to the administration website of your VPS. After doing so, we'll be able to boot into our system as a `root` user (usually, using a password that we are prompted to set before booting into recovery).

The rescue mode is just:

- a new machine that boots on a minimal disk with a minimal OS
- and attach your disk

You get then access to your file and disk. You can perform administrative tasks such as:

- deactivating your firewall
- backup or data recovery
- update your network configuration files
- etc.

They should send you via email or via their dashboard the root and password credentials of the new virtual machine created.

## Login to the rescue VPS and check the disks

Once you have logged on to your machine, the prompt should indicate that it's in rescue mode.

```shell
[RESCUE] root@vps-427a1b7c:/ $
```

You can list the [disk partitions](https://datacadamia.com/os/linux/disk/partition "Linux - Disk (storage devices) can be divided into one or more logical disks called File System - Partition / Volume (Logical Disk Partition). This division is described in the File System - Partition table (Sector 0) found in sector 0 of the disk.  Articles Related Syntax Name The partition is the last part of the name.  For example,") with the lsblk command.

```shell
lsblk
```

```swift
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0  2.5G  0 disk
└─sda1   8:1    0  2.5G  0 part /
sdb      8:16   0   80G  0 disk
└─sdb1   8:17   0   80G  0 part
```

The above output shows two disks device:

- sda1 of 2.5 Gb [mounted](https://datacadamia.com/os/linux/disk/mount "...  All files accessible in a Unix system are arranged in one big tree, the file hierarchy, rooted at /. These files can be spread out over several Linux - Device (Disk, CDROM, ). The mount command serves to attach the file system found on some device to the big file tree. Conversely, the umount command will detach it again.") at the root, the new VPS
- sdb1 of 80 Gb, not mounted, the disk of your machine that contains all your data.

In a non-rescue mode, you would see only your disk.

```shell
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
sda      8:0    0  80G  0 disk
└─sda1   8:1    0  80G  0 part /
```

## Mount your disk to access your data

To get access to the data on your disk, you need to [mount it](https://datacadamia.com/os/linux/disk/mount "...  All files accessible in a Unix system are arranged in one big tree, the file hierarchy, rooted at /. These files can be spread out over several Linux - Device (Disk, CDROM, ). The mount command serves to attach the file system found on some device to the big file tree. Conversely, the umount command will detach it again.").

- Create a mount point if necessary (ie directory where your data will be available)

```shell
# /mnt may be already created
mkdir /mnt
```

- Mount your disk into this directory

```shell
mount /dev/sdb1 /mnt
```

- Check that you have access to your data
- Modify the root of the file system. It's not always needed but all process and file system will think logically that the root of the file system / is now /mnt.

```shell
chroot /mnt
```

At this stage, you have access to your disk, you can search file

- by name

```shell
find . -name myfile.myextension
```

- or by content with a pattern

```shell
grep -rnw . -e 'how to disable ?'
```

---

## Bypass 'Connection Refused' Errors

Coming soon.

UFW Location
