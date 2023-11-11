---
title: "Dynamically creating an IMG file with a FAT32 partition"
slug: dynamically-creating-an-img-file-with-a-fat32-partition
aliases:
- /2023/10/27/dynamically-creating-img-file-with-fat32-partition.html
author: "Alex Oswald"
date: 2023-10-27
---

I recently had a need to dynamically create an `img` file containing a FAT32 partition so its contents would be bootable on a PC. I had a heck of a time trying to put all the pieces together from different articles online. Alas, I came up with a working solution, so here it is. Enjoy!

> This solution is for Linux. It does not work on Windows, but will work on Windows Subsystem for Linux (WSL).

This is a multi-step process outlined below:

1. Create the `disk.img` image file
    - Create empty `disk.img` file
    - Create partition table
    - Create partition
    - Format partition as FAT32 file system
2. Create the `partition.img` image file
    - Create empty `partition.img` file
    - Format the partition image file as FAT32 file system
3. Copy files into `partition.img`
4. Copy `partition.img` into the partition in `disk.img`

# Create the `disk.img` image file

First, we want to create a blank `img` file.

The [`dd`](https://www.man7.org/linux/man-pages/man1/dd.1.html) command is used to create a file of a specified size. The `bs` parameter is the block size, and `count` is the number of blocks. So if we want a 1GB file, we would do `bs=1M count=1024`. The `if` parameter is the input file, which in this case is `/dev/zero`. `/dev/zero` is a special file that provides as many null characters as you want. So we can use this to create a blank file. The `of` parameter is the output file, which in this case is `disk.img`.

```bash
$ sudo dd if=/dev/zero of=disk.img bs=1M count=1024
user@example:~/img_test$ sudo dd if=/dev/zero of=disk.img bs=1M count=1024
1024+0 records in
1024+0 records out
1073741824 bytes (1.1 GB, 1.0 GiB) copied, 0.707222 s, 1.5 GB/s
```

Now we need to create a GPT partition table in the `img` file.

We are using the [parted](https://github.com/bcl/parted) tool to manage the disk partitions. The `--script` parameter tells `parted` to not prompt for user input and that we are running in a script for automation. The `disk.img` parameter is the path to the `img` file we created. The `mktable` command creates a new partition table. The `gpt` parameter specifies the type of partition table to create.

```bash
sudo parted --script disk.img mktable gpt
```

Create a FAT32 partition that spans the entire disk.

`--align optimal` ensures the partition is aligned to the optimal sector size for the disk. `mkpart` creates a partition. `primary` is the partition type. `fat32` is the file system type. `2048s` is the starting sector. `100%` is the ending sector.

```bash
sudo parted --script disk.img --align optimal mkpart primary fat32 2048s 100%
```

We can name the partition if we want. This is optional.

`name` sets the name of the partition. `1` is the partition number. `BOOT` is the name.

```bash
sudo parted --script disk.img name 1 BOOT
```

We can print the partition table to verify everything is correct.

```bash
$ sudo parted --script disk.img print
Model:  (file)
Disk /home/user/disk.img: 1074MB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name  Flags
 1      1049kB  1073MB  1072MB               BOOT  msftdata
```

# Create the `partition.img` image file

We will use `dd` to create another `img` file to represent the FAT32 partition. Make it the same size as `disk.img` minus a couple megabytes. This is to make up for the fact that the partition stars 2048 sectors in, and not at the beginning of the disk. If you don't, you will get out of space errors when copying this partition into `disk.img`

```bash
$ sudo dd if=/dev/zero of=partition.img bs=1M count=1022
1022+0 records in
1022+0 records out
1071644672 bytes (1.1 GB, 1022 MiB) copied, 1.00757 s, 1.1 GB/s
```

Now we need to create a FAT32 file system on the partition image file.

We will use the `mkfs.vfat` command to do this. `-F 32` specifies the FAT32 file system. `-v` is verbose output.

```bash
$ sudo mkfs.vfat partition.img -F 32 -v
mkfs.fat 4.2 (2021-01-31)
partition.img has 64 heads and 63 sectors per track,
hidden sectors 0x0000;
logical sector size is 512,
using 0xf8 media descriptor, with 2093049 sectors;
drive number 0x80;
filesystem has 2 32-bit FATs and 8 sectors per cluster.
FAT size is 2040 sectors, and provides 261117 clusters.
There are 32 reserved sectors.
Volume ID is 986efae3, no volume label.
```

Let us print the partition info to verify everything is correct.

```bash
$ sudo parted --script partition.img print
Model:  (file)
Disk /home/user/img_test/partition.img: 1072MB
Sector size (logical/physical): 512B/512B
Partition Table: loop
Disk Flags:

Number  Start  End     Size    File system  Flags
 1      0.00B  1072MB  1072MB  fat32
```

# Copy files into `partition.img`

We can use the [mcopy](https://www.gnu.org/software/mtools/manual/mtools.html#mcopy) tool to copy files into the `partition.img` file.

To test this, I created the directory `files`, with 3 text files, and a directory `inner_dir`, that contains another text file.

```bash

```bash
$ sudo mcopy -i partition.img /home/user/files/* -spQmv ::/
Copying inner_dir
Copying text4.txt
Copying text1.txt
Copying text2.txt
Copying text3.txt
```

`s` indicates a recursive copy and will copy all directories and their contents. `p` preserves the file attributes. `Q` is valid when copying multiple files and will quit as soon as one copy fails. `m` preserves the file modification time. `v` is verbose output.

There is a `b` switch to use *batch* copy mode for large recursive copies. For me, it kept erroring with `Internal error, size too big`.

# Copy `partition.img` into the partition in `disk.img`

Lastly, we need to copy the contents of the `partition.img` file into the partition in `disk.img`. We can do this with the `dd` command after mounting the partitions of the image file.

We will use `kpartx` to attach the partition in `disk.img` to a loop device. `kpartx` will create a loop device for each partition in the image file. We will use the `-a` switch to add the partition to the loop device. `-v` is verbose output.

```bash
$ sudo kpartx -av disk.img
add map loop5p1 (253:0): 0 2093056 linear 7:5 2048
```

We can verify the attachment by listing out block devices.

```bash
$ lsblk
NAME      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0       7:0    0  59.2M  1 loop /snap/core20/1977
loop1       7:1    0  59.3M  1 loop /snap/core20/2019
loop2       7:2    0 109.6M  1 loop /snap/lxd/24326
loop3       7:3    0  35.5M  1 loop /snap/snapd/20102
loop4       7:4    0  35.5M  1 loop /snap/snapd/20298
loop5       7:5    0     1G  0 loop
└─loop5p1 253:0    0  1022M  0 part
sda         8:0    0     1T  0 disk
└─sda1      8:1    0  1024G  0 part /mnt/example
sdb         8:16   0    30G  0 disk
├─sdb1      8:17   0  29.9G  0 part /
└─sdb15     8:31   0    99M  0 part /boot/efi
```

We can see that `loop5` represents the loop device for `disk.img` and `loop5p1` is the partition we created. Now lets copy the contents of `partition.img` into the `loop5p1` partition. Notice the `mapper` location. This is where `kpartx` attached the partition.

```bash
$ sudo dd if=partition.img of=/dev/mapper/loop5p1 bs=1M
1022+0 records in
1022+0 records out
1071644672 bytes (1.1 GB, 1022 MiB) copied, 2.15913 s, 496 MB/s
```

Now lets unmount the partition.

```bash
$ sudo kpartx -dv disk.img
del devmap : loop5p1
loop deleted : /dev/loop5
```

We can delete `partition.img`.

Lets print the partition info for `disk.img` once more to verify we're done.

```bash
$ sudo parted --script disk.img print
Model:  (file)
Disk /home/sync-user/img_test/disk.img: 1074MB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name  Flags
 1      1049kB  1073MB  1072MB  fat32        BOOT  msftdata
```

We have a `gpt` type partition table and 1 partition labeled `BOOT` formatted with the FAT32 file system! `disk.img` is ready to go!

# References

- [Layouting a disk image and copying files into it](https://superuser.com/questions/868117/layouting-a-disk-image-and-copying-files-into-it)