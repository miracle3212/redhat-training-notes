# Unit 10: Partitions and File Systems

## Partitions

### Commands
    fdisk
    partx
    mkfs.ext4
    blkid
    mount
    umount
    dumpe2fs

### Files
    /etc/fstab

### Examples
    # create a new partition
    # (import fdisk commands: m, p, n, w, t)
    fdisk -cu /dev/vda # -l to list
        n, p, 3, <enter>, +64MB, w
    # add the partition to the partition table
    partx -a /dev/vda
    # verify it's added
    ls /dev/vda3
    # format it to ext4
    mkfs.ext4 /dev/vda3
    # verify it has been formated to ext4 
    blkid
    # mount it
    mkdir /data && mount /dev/vda3 /data
    # to mount it persistently, add an entry to /etc/fstab:
    # /dev/vda3   /data  ext4  defaults   1 2
    # always after modifying fstab
    mount -a
    # to unmount it
    umount /data
    # file system information
    dumpe2fs /dev/vda3 | less

### Notes
* see example config in [config/partitions](config/partitions)
* in ```fstab```, put ```0``` as the last parameter on the exam, never in production

## Encryption

### Commands
    cryptsetup

### Files
    /etc/crypttab

### Examples
    cryptsetup luksFormat /dev/vda5 # to encrypt the fs
    cryptsetup luksOpen /dev/vda5 secret # to open it in /dev/mapper/secret
    cryptsetup luksClose secret

### Notes
* NOT on the exam
* to create a fs on an encrypted partition, do it in /dev/mapper/secret: ```mkfs.ext4 /dev/mapper/secret```
* to mount an encrypted fs, put an entry in ```/etc/fstab```
* ```/etc/crypttab``` is used to tell the system to unencrypt system automatically on boot (so that it can be mounted)

## Swap

### Commands
    mkswap
    swapon
    swapoff

### Examples
    # to see current amount of swap space
    top
    # add a new partition
    fdisk -cu /dev/vda
        n, p, <enter>, +1G, t, 4, 82, w
    partx -a /dev/vda
    # make swap
    mkswap /dev/vda4
    # add it to fstab:
    # UUID=cf8c009c-3ec0-468f-b092-924cc340d2a6   swap  swap	defaults  0  0
    # see current swap partitions
    swapon -s
    # add new swap partition
    swapon -a
    # check again with top
    top
    # remove it
    swapoff /dev/vda4

### Notes
* swap space: extension of memory on disk
* swapping pages from memory that haven't been used in a long time to disk

# Unit 11: File Access

### Notes
* sticky bit (```t```) for directories: only the file owner or the directory owner can rename or delete files (default: any user with write and execute permissions for the directory can rename or delete contained files, regardless of owner)
* suid bit (```s```) for binaries: binary is run as the user owning it (default: as the user executing it); e.g. ```/usr/bin/passwd```
* sgid bit (```s```) for binaries: same, but applies to groups
* sgid bit (```s```) for directories: for newly created files in a dir, they inherit group from the dir (default: the group they get is the primary group of the user who created the file)

## ACLs

### Commands
    getfacl
    setfacl

### Examples
    setfacl -m u:clair:rwx . # grant clair access to dir
    setfacl -m d:u:clair:rwx . # grant clair access to newly created files in dir
    setfacl -m u:faraday:- date # remove access to date for faraday

### Notes
* to mount the filesystem with ACL enabled, in ```fstab```, put ```acl``` instead of ```defaults```
* use case: user faraday wants to grant access to a directory to user clair. but they don't share any groups. and he can't create a common group for them as he is not root. how to do this? he can use ACLs on his files to explicitly grant access to specific user.

# Unit 12: LVM

### Commands
    pvcreate
    vgcreate
    lvcreate
    pvs
    pvscan
    pvdisplay
    vgs
    vgscan
    vgdisplay
    lvs
    lvscan
    lvdisplay
    lvextend
    lvreduce
    lvresize

### Files
    /etc/fstab

### Examples
    # first create a new partition
    fdisk -cu /dev/vda
        n, +256M
        t, 8e
        w
    partx -a /dev/vda
    # it is not necessary to run pvcreate, we can run vgcreate right away, it will do pvcreate as well
    vgcreate -s 8M myvg /dev/vda7
    # display information
    blkid
    pvs
    pvscan
    pvdisplay
    vgs
    vgscan
    vgdisplay
    lvs
    lvscan
    lvdisplay
    ls -l /dev/vgsrv/
    ls -l /dev/mapper/
    # let's create an LV
    lvcreate -L 17M -n mylv myvg
    # format it
    mkdir /lvm
    mkfs.ext4 /dev/mapper/myvg-mylv
    # put an entry in /etc/fstab
    # mount it
    mount -a
    # let's extend it
    lvresize -l +150%LV -n -r /dev/mapper/myvg-mylv # always use -n -r
    # shrink it to 40M
    lvresize -L 40M -n -r /dev/mapper/myvg-mylv
    # add additional PV to the VG
    fdisk -cu /dev/vda
        n, +256M
        t, 8e
    partx -a /dev/vda
    vgextend myvg /dev/vda8
    vgs
    pvs
    # to remove PV from VG
    # first remove extends from PV
    pvmove /dev/vda7
    pvs # it's empty
    # now remove it
    vgreduce myvg /dev/vda7
    # check
    pvs
    # create a snapshot
    lvcreate -s -L 16M -n snap /dev/mapper/myvg-mylv
    # check it's created    
    ls /dev/mapper/myvg-snap
    mkdir /snap
    # add it to fstab
    mount -a
    # change /lvm/hosts
    # check /lvm/hosts, /snap/hosts

### Notes
* see example fstab config in [config/lvm](config/lvm)
* use case: how to extend partitions
* LVM has 3 layers: 1) linux block devices, 2) physical volumes, 3) volume groups
* PVs are built on top of block devices, and PVs are united into a single pool - VG
* PE (physical extent) size is the smallest allocatable size within a volume group
* LV (logical volume) can be created from VG
* LVs can be extended, snapshots can be created from it

# Unit 13: Boot

### Files
    /boot/grub/grub.conf # bootloader
    /etc/inittab # default runlevel
    /proc/cmdline # what parameters the kernal was booted with

### Notes
* to change the root password: boot into runlevel 1, you will be logged in as root, run passwd to change password, init 3 to go to runlevel 3
* to see how many kernels there are: ```rpm -q kernel```
