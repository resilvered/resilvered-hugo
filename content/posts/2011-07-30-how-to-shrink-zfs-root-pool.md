---
title: "How to Shrink zfs root pool."
date: 2011-07-30T00:00:00+00:00
draft: false
categories: [blog, zfs, work, Solaris 11]
tags: [zfs, work, Solaris 11]
---

This exercise for me is to replace my current single disk rpool, the second disk was removed for another project, on to two slightly smaller disks. Doing so restores my root mirror but as the disks are slightly smaller some work is required.

For reference my current root pool name is rpool, the current disk in this pool is c5t1d0, the first replacement disk is install at c5t0d0 and is empty.

Use fdisk create a Solaris patition.

```
# fdisk /dev/rdsk/c5t0d0p0
No fdisk table exists. The default partition for the disk is:

a 100% "SOLARIS System" partition

Type "y" to accept the default partition,  otherwise type "n" to edit the
partition table.
n

        Total disk size is 7788 cylinders
        Cylinder size is 12544 (512 byte) blocks

                                          Cylinders
 Partition   Status    Type          Start   End   Length    %
 =========   ======    ============  =====   ===   ======   ===
     1       Active    Solaris2          1  4672    4672     60
     2                 Win95 FAT32    4673  7787    3115     40

SELECT ONE OF THE FOLLOWING:
1. Create a partition
2. Specify the active partition
3. Delete a partition
4. Change between Solaris and Solaris2 Partition IDs
5. Edit/View extended partitions
6. Exit (update disk configuration and exit)
7. Cancel (exit without updating disk configuration)
Enter Selection: 6

```

Use format and create Silce 0 using the whole partition

Create temporary new pool on new disk

```
# zpool create -f tpool c5t0d0s0
# zfs set compression=on tpool

```

Snaphot and copy to my new tpool

```
# zfs snapshot -r rpool@shrink
# zfs send -vR rpool@shrink | zfs receive -vFd tpool

```

Prepare for booting

```
# rm /tpool/boot/grub/bootsign/pool_rpool
# touch /tpool/boot/grub/bootsign/pool_tpool
# zpool set bootfs=tpool/ROOT/Solaris_11_SRU4 tpool
# cd /boot/grub/
# /sbin/installgrub stage1 stage2 /dev/rdsk/c5t0d0s0
stage2 written to partition 0, 277 sectors starting at 50 (abs 12594)
stage1 written to partition 0 sector 0 (abs 12544)
# vi /tpool/boot/grub/menu.lst

```

Change all the references to tpool to rpool in the menu.1st file.

At this point shutdown, on power up go into the BIOS and make sure you are booting off the right disk. If you need to be sure remove the current boot disk

Once confrimed I'm running off my new root pool, tpool, I shutdown install the second new drive as c5t1d0 and restart

At this stage I could just attach my new disk to the mirror and we are in buisness, but I'd like to keep my root pool name as rpool. So I perform pretty much the same proceedure as before.

Use fdisk to partition the disk

```
# fdisk /dev/rdsk/c5t0d0p0

```

Copy over the vtoc to my new disk

```
# prtvtoc  /dev/rdsk/c5t0d0s2 | fmthard -s - /dev/rdsk/c5t1d0s2

# zpool create -f rpool c5t1d0s0
# zfs set compression=on rpool

```

Clean up my shrink snapshots from the first run

```
# zfs list -t snapshot | grep shrink
tpool@shrink                                      23K      -    80K  -
tpool/ROOT@shrink                                   0      -    21K  -
tpool/ROOT/Solaris_11_SRU3@shrink                   0      -  3.11G  -
tpool/ROOT/Solaris_11_SRU4@shrink               96.2M      -  3.15G  -
tpool/ROOT/opensolaris@shrink                       0      -  2.04G  -
tpool/ROOT/opensolaris-134b@shrink                  0      -  2.75G  -
tpool/ROOT/opensolaris-134b-1@shrink                0      -  2.92G  -
tpool/dump@shrink                                   0      -  1019M  -
tpool/swap@shrink                                   0      -   353M  -

# zfs destroy tpool@shrink
# zfs destroy tpool/ROOT@shrink
# zfs destroy tpool/ROOT/Solaris_11_SRU3@shrink
# zfs destroy tpool/ROOT/Solaris_11_SRU4@shrink
# zfs destroy tpool/ROOT/opensolaris-134b@shrink
# zfs destroy tpool/ROOT/opensolaris-134b-1@shrink
# zfs destroy tpool/dump@shrink
# zfs destroy tpool/swap@shrink
# zfs destroy tpool/ROOT/opensolaris@shrink

# zfs snapshot -r tpool@shrink
# zfs send -vR tpool@shrink | zfs receive -vFd rpool

# rm /rpool/boot/grub/bootsign/pool_tpool
# touch /rpool/boot/grub/bootsign/pool_rpool
# zpool set bootfs=rpool/ROOT/Solaris_11_SRU4 rpool
# cd /boot/grub/
# /sbin/installgrub stage1 stage2 /dev/rdsk/c5t1d0s0
stage2 written to partition 0, 277 sectors starting at 50 (abs 12594)
stage1 written to partition 0 sector 0 (abs 12544)
# vi /rpool/boot/grub/menu.lst
# reboot -p

```

Change all references of tpool to rpool in menu.lst

Reboot go back to BIOS and make sure I'm now booting of my new root pool called rpool

```
# zpool list
NAME    SIZE  ALLOC   FREE    CAP  DEDUP  HEALTH  ALTROOT
rpool  27.8G  6.96G  20.8G    25%  1.00x  ONLINE  -
tpool  27.8G  6.90G  20.8G    24%  1.00x  ONLINE  -
# zpool destroy tpool
# zpool attach -f rpool c5t1d0s0 c5t0d0s0
Make sure to wait until resilver is done before rebooting.
# zpool status rpool
pool: rpool
state: ONLINE
status: One or more devices is currently being resilvered.  The pool will
   continue to function, possibly in a degraded state.
action: Wait for the resilver to complete.
scan: resilver in progress since Sat Jul 30 15:07:54 2011
1.20G scanned out of 6.96G at 42.3M/s, 0h2m to go
1.20G resilvered, 17.21% done
config:

   NAME          STATE     READ WRITE CKSUM
   rpool         ONLINE       0     0     0
     mirror-0    ONLINE       0     0     0
       c5t1d0s0  ONLINE       0     0     0
       c5t0d0s0  ONLINE       0     0     0  (resilvering)

# cd /boot/grub/
# /sbin/installgrub stage1 stage2 /dev/rdsk/c5t0d0s0
stage2 written to partition 0, 277 sectors starting at 50 (abs 12594)
stage1 written to partition 0 sector 0 (abs 12544)

```

Once the resilver has finish boot reboot back to the BIOS and ensure I can actually boot of either of the disks in the rpool.

```
# reboot -p

```
