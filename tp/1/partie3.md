🌞 **Afficher l'état actuel de LVM**

```
[antna@node1 ~]$ sudo pvscan
  PV /dev/sda1   VG rl_vbox   lvm2 [26.00 GiB / 4.00 MiB free]
  Total: 1 [26.00 GiB] / in use: 1 [26.00 GiB] / in no VG: 0 [0   ]
[antna@node1 ~]$ sudo vgscan
  Found volume group "rl_vbox" using metadata type lvm2
[antna@node1 ~]$ sudo lvscan
  ACTIVE            '/dev/rl_vbox/home' [15.00 GiB] inherit
  ACTIVE            '/dev/rl_vbox/root' [10.00 GiB] inherit
  ACTIVE            '/dev/rl_vbox/swap' [1.00 GiB] inherit
[antna@node1 ~]$
```

🌞 **Déterminer le type de système de fichiers**

```
[antna@node1 ~]$ sudo blkid /dev/rl_vbox/home
/dev/rl_vbox/home: UUID="a02a63dc-03ee-4704-9646-31de891bba0e" TYPE="ext4
"
[antna@node1 ~]$ sudo blkid /dev/rl_vbox/
home  root  swap
[antna@node1 ~]$ sudo blkid /dev/rl_vbox/root
/dev/rl_vbox/root: UUID="3ee1e31c-be42-4a06-a969-d405a5c46fbb" TYPE="ext4"
[antna@node1 ~]$ sudo blkid /dev/rl_vbox/swap
/dev/rl_vbox/swap: UUID="f91086d7-e520-4ff8-b2bb-cf36d6e17bad" TYPE="swap"
[antna@node1 ~]$
```

🌞 **Remplissez votre partition `/home`**

```
[antna@node1 ~]$ dd if=/dev/zero of=/home/antna/bigfile bs=4M count=3000
dd: error writing '/home/antna/bigfile': No space left on device
2166+0 records in
2165+0 records out
15892086784 bytes (16 GB, 15 GiB) copied, 17.6371 s, 901 MB/s
```

🌞 **Constater que la partition est pleine**

```
[antna@node1 ~]$ df -h
Filesystem                Size  Used Avail Use% Mounted on
devtmpfs                  4.0M     0  4.0M   0% /dev
tmpfs                     1.8G     0  1.8G   0% /dev/shm
tmpfs                     732M  8.6M  724M   2% /run
/dev/mapper/rl_vbox-root  9.8G  1.5G  7.8G  16% /
/dev/sda2                 974M  272M  635M  30% /boot
/dev/mapper/rl_vbox-home   15G   15G  1.2M 100% /home
tmpfs
```


🌞 **Agrandir la partition**

```
[antna@node1 ~]$ sudo vgs
  VG      #PV #LV #SN Attr   VSize  VFree
  rl_vbox   1   3   0 wz--n- 26.00g 4.00m
[antna@node1 ~]$ sudo fdisk /dev/sda

Welcome to fdisk (util-linux 2.37.4).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

This disk is currently in use - repartitioning is probably a bad idea.
It's recommended to umount all file systems, and swapoff all swap
partitions on this disk.


Command (m for help): p

Disk /dev/sda: 30 GiB, 32212254720 bytes, 62914560 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xc489354b

Device     Boot    Start      End  Sectors Size Id Type
/dev/sda1           2048 54544383 54542336  26G 8e Linux LVM
/dev/sda2  *    54544384 56641535  2097152   1G 83 Linux

Command (m for help): n
Partition type
   p   primary (2 primary, 0 extended, 2 free)
   e   extended (container for logical partitions)
Select (default p):

Using default response p.
Partition number (3,4, default 3):
First sector (56641536-62914559, default 56641536):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (56641536-62914559, default 62914559):

Created a new partition 3 of type 'Linux' and of size 3 GiB.

Command (m for help): w
The partition table has been altered.
Syncing disks.

[antna@node1 ~]$ lsblk
NAME             MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                8:0    0   30G  0 disk
├─sda1             8:1    0   26G  0 part
│ ├─rl_vbox-root 253:0    0   10G  0 lvm  /
│ ├─rl_vbox-swap 253:1    0    1G  0 lvm  [SWAP]
│ └─rl_vbox-home 253:2    0   15G  0 lvm  /home
├─sda2             8:2    0    1G  0 part /boot
└─sda3             8:3    0    3G  0 part
sr0               11:0    1 1024M  0 rom
[antna@node1 ~]$ sudo pvcreate /dev/sda3
  Physical volume "/dev/sda3" successfully created.
[antna@node1 ~]$ sudo pvs
  PV         VG      Fmt  Attr PSize  PFree
  /dev/sda1  rl_vbox lvm2 a--  26.00g 4.00m
  /dev/sda3          lvm2 ---   2.99g 2.99g
[antna@node1 ~]$ sudo vgs
  VG      #PV #LV #SN Attr   VSize  VFree
  rl_vbox   1   3   0 wz--n- 26.00g 4.00m
```

```
[antna@node1 ~]$ sudo vgextend rl_vbox /dev/sda3
  Volume group "rl_vbox" successfully extended
[antna@node1 ~]$ sudo lvextend -L +1G /dev/rl_vbox/home
  Size of logical volume rl_vbox/home changed from 15.00 GiB (3840 extents) to 16.00 GiB (4096 extents).
  Logical volume rl_vbox/home successfully resized.
[antna@node1 ~]$ sudo xfs_growfs /dev/rl_vbox/home
meta-data=/dev/mapper/rl_vbox-home isize=512    agcount=4, agsize=983040 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=0
data     =                       bsize=4096   blocks=3932160, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=16384, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 3932160 to 4194304
[antna@node1 ~]$ df -h
Filesystem                Size  Used Avail Use% Mounted on
devtmpfs                  4.0M     0  4.0M   0% /dev
tmpfs                     1.8G     0  1.8G   0% /dev/shm
tmpfs                     732M  8.6M  724M   2% /run
/dev/mapper/rl_vbox-root  9.8G  1.5G  7.8G  16% /
/dev/sda2                 974M  272M  635M  30% /boot
/dev/mapper/rl_vbox-home   16G   15G 1018M  94% /home
tmpfs                     366M     0  366M   0% /run/user/1000
```

🌞 **Remplissez votre partition `/home`**

- on va simuler encore avec un truc bourrin :

```
[antna@node1 ~]$ dd if=/dev/zero of=/home/antna/bigfile-extend bs=4M count=2500
dd: error writing '/home/antna/bigfile-extend': No space left on device
255+0 records in
254+0 records out
1065549824 bytes (1.1 GB, 1016 MiB) copied, 1.29712 s, 821 MB/s
```

➜ **Eteignez la VM et ajoutez lui un disque de 40G**

🌞 **Utiliser ce nouveau disque pour étendre la partition `/home` de 20G**

```
[antna@node1 ~]$ sudo fdisk /dev/sdb

Welcome to fdisk (util-linux 2.37.4).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p):

Using default response p.
Partition number (1-4, default 1):
First sector (2048-83886079, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-83886079, default 83886079): +20G

Created a new partition 1 of type 'Linux' and of size 20 GiB.
Partition #1 contains a LVM2_member signature.

Do you want to remove the signature? [Y]es/[N]o: N

Command (m for help): w

The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

[antna@node1 ~]$ sudo pvcreate /dev/sdb1
  Physical volume "/dev/sdb1" successfully created.

[antna@node1 ~]$ sudo vgextend rl_vbox /dev/sdb1
  Volume group "rl_vbox" successfully extended

[antna@node1 ~]$ sudo lvextend -L +20G /dev/rl_vbox/home
  Size of logical volume rl_vbox/home changed from 16.00 GiB (4096 extents) to 36.00 GiB (9216 extents).
  Logical volume rl_vbox/home successfully resized.

[antna@node1 ~]$ sudo xfs_growfs /dev/rl_vbox/home
meta-data=/dev/mapper/rl_vbox-home isize=512    agcount=5, agsize=983040 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=0
data     =                       bsize=4096   blocks=4194304, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=16384, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 4194304 to 9437184

[antna@node1 ~]$ df -h
Filesystem                Size  Used Avail Use% Mounted on
devtmpfs                  4.0M     0  4.0M   0% /dev
tmpfs                     1.8G     0  1.8G   0% /dev/shm
tmpfs                     732M  9.4M  723M   2% /run
/dev/mapper/rl_vbox-root  9.8G  1.5G  7.8G  16% /
/dev/sda2                 974M  272M  635M  30% /boot
/dev/mapper/rl_vbox-home   36G   17G   20G  45% /home
tmpfs                     366M     0  366M   0% /run/user/1000
```

🌞 **Créez une nouvelle partition**

```
[antna@node1 ~]$ sudo lvcreate -L 20G -n web rl_vbox
  Logical volume "web" created.

[antna@node1 ~]$ sudo mkfs.ext4 /dev/sdb2
mke2fs 1.46.5 (30-Dec-2021)
/dev/sdb2 is apparently in use by the system; will not make a filesystem here!
[antna@node1 ~]$ sudo mkfs.ext4 /dev/rl_vbox/web
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 5242880 4k blocks and 1310720 inodes
Filesystem UUID: 9ac3662a-95cc-4fbb-ad5c-9126b4b0bc82
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000

Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done

[antna@node1 ~]$ sudo mount /dev/rl_vbox/web /var/www
[antna@node1 ~]$ df -h
Filesystem                Size  Used Avail Use% Mounted on
devtmpfs                  4.0M     0  4.0M   0% /dev
tmpfs                     1.8G     0  1.8G   0% /dev/shm
tmpfs                     732M  9.5M  723M   2% /run
/dev/mapper/rl_vbox-root  9.8G  1.5G  7.8G  16% /
/dev/sda2                 974M  272M  635M  30% /boot
/dev/mapper/rl_vbox-home   36G   17G   20G  45% /home
tmpfs                     366M     0  366M   0% /run/user/1000
/dev/mapper/rl_vbox-web    20G   24K   19G   1% /var/www
```

🌞 **Proposez au moins une option de montage**

```
[antna@node1 ~]$ sudo mount -o noexec /dev/rl_vbox/web /var/www
```