// TODO:
1. Private key setup described below
2. Remove passphrase from private key with `passwd`
3. Cert Key expiration date hinzufügen - 3 Monate
4. Subkey auf 3 Monate

# Yubi-Key setup

## Setup Air Gaped Machine

https://archlinux.org/download/

set keyboard layout to german: 



### GPG Private key Setup


#### create partition on storage device


Get devices `lsblk`, the we will use `sda` in the following example
https://wiki.archlinux.org/title/Dm-crypt/Device_encryption


1. Run:
`udisksctl unmount --block-device=/dev/sda1`

Desired output (not mounted):
```bash
❯ lsblk                                     
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda           8:0    1   7,5G  0 disk 
└─sda1        8:1    1   7,5G  0 part 

``` 

1. Create Partition of 1 Gb

```bash
sudo fdisk /dev/sda 
[sudo] password for eduard:         

Welcome to fdisk (util-linux 2.37.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

The device contains 'iso9660' signature and it will be removed by a write command. See fdisk(8) man page and --wipe option for more details.

Command (m for help): m

Help:

  DOS (MBR)
   a   toggle a bootable flag
   b   edit nested BSD disklabel
   c   toggle the dos compatibility flag

  Generic
   d   delete a partition
   F   list free unpartitioned space
   l   list known partition types
   n   add a new partition
   p   print the partition table
   t   change a partition type
   v   verify the partition table
   i   print information about a partition

  Misc
   m   print this menu
   u   change display/entry units
   x   extra functionality (experts only)

  Script
   I   load disk layout from sfdisk script file
   O   dump disk layout to sfdisk script file

  Save & Exit
   w   write table to disk and exit
   q   quit without saving changes

  Create a new label
   g   create a new empty GPT partition table
   G   create a new empty SGI (IRIX) partition table
   o   create a new empty DOS partition table
   s   create a new empty Sun partition table


Command (m for help): g

Created a new GPT disklabel (GUID: BD1FAE45-0164-734C-92ED-F0A188D76453).
The device contains 'iso9660' signature and it will be removed by a write command. See fdisk(8) man page and --wipe option for more details.

Command (m for help): n
Partition number (1-128, default 1): 1
First sector (2048-15728606, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-15728606, default 15728606): +1G

Created a new partition 1 of type 'Linux filesystem' and of size 1 GiB.
Partition #1 contains a vfat signature.

Do you want to remove the signature? [Y]es/[N]o: Y

The signature will be removed by a write command.

Command (m for help): p
Disk /dev/sda: 7,5 GiB, 8053063680 bytes, 15728640 sectors
Disk model: USB Flash Disk  
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: BD1FAE45-0164-734C-92ED-F0A188D76453

Device     Start     End Sectors Size Type
/dev/sda1   2048 2099199 2097152   1G Linux filesystem

Filesystem/RAID signature on partition 1 will be wiped.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

``` 

2. fill the 1Gb with Bullsh*t:
```bash
sudo dd bs=32MiB conv=fsync if=/dev/urandom  oflag=direct status=progress of=/dev/sda1

1073741824 bytes (1,1 GB, 1,0 GiB) copied, 147 s, 7,3 MB/s
dd: error writing '/dev/sda1': No space left on device
33+0 records in
32+0 records out
1073741824 bytes (1,1 GB, 1,0 GiB) copied, 147,596 s, 7,3 MB/s

```


3. use command to encrypt your storage: 

--pbkdf-memory 4194304 # 4Gb
--pbkdf-parallel 8 # 8 Threads

Generate as long ass password e.g. 128 characters to use it for the next step:

```bash
$ sudo cryptsetup --type luks2 --cipher aes-xts-plain64 --hash sha512 --integrity hmac-sha512 --iter-time 10000 --key-size 512 --pbkdf argon2id  --pbkdf-memory 4194304  --pbkdf-parallel 8 --use-urandom luksFormat /dev/sda1

WARNING!
========
This will overwrite data on /dev/sda1 irrevocably.

Are you sure? (Type 'yes' in capital letters): YES
Enter passphrase for /dev/sda1: 
Verify passphrase: 

```

AK6x5paqsiGL2A^ePpf!x$QXSEHYJ@6Cax*PHhdwdaRj@hbYKgkC79P5@*imqBnSRwaZvZE#Dd6QycUmx%JB7*&WgRc5e6FhCnrMc9nEpypmMp#tt!xNuE46XvjHgUzJ


5. Configure your yubikey as part of the password to encrypt your data storage

- Generate a strong random password to store that in your yubikey 

- 1 = ShortPress
- 2 = LongPress

as layout use your prefered keyboard layout

```bash
ykman otp static --keyboard-layout DE --generate --length 38 -- 2
Slot 2 is already configured. Overwrite configuration? [y/N]: y
```
Pass-1: banane
Pass-2: 8DZkn/?ZCkÖ!JoZySEyq9UVpb;1xXjPlH)&7U%

Combined: banane8DZkn/?ZCkÖ!JoZySEyq9UVpb;1xXjPlH)&7U%


get the password by pressing your yubikey (if used 1 short press, if used 2 longpress)

generate your complete password with a simple prefix: e.g.

<span style="color: red">banana</span><span style="color: blue">Uy6jRkh7Ced2p!g6YqyPGj%$ryHurV?o</span>




6. generate another keyslot

```bash
sudo cryptsetup luksAddKey /dev/sda1 

```

```bash
sudo cryptsetup open -- /dev/sda1 GnuPG-home
Enter passphrase for /dev/sda1: 
```



```bash
lsblk 
NAME           MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINTS
sda              8:0    1   7,5G  0 disk  
└─sda1           8:1    1     1G  0 part  
  └─GnuPG-home 252:0    0  1008M  0 crypt 

```

btrfs with checksums against bitrot and snapshots as first-line backups 
```bash
sudo mkfs.btrfs --checksum=blake2 --label=GnuPG-home -- /dev/mapper/GnuPG-home
```

```bash
sudo cryptsetup close -- GnuPG-home
```


#### Store your private key onto the encrypted 

!! AIR GAPPED MACHINE !!

Run all commands below as root.

set keyboard layout with;

```bash
localectl --no-convert set-keymap de-latin1-nodeadkeys
```

Now you can put the gpg private key onto your storage device `/mnt`

```bash
cryptsetup open -- /dev/sda1 GnuPG-home
Enter passphrase for /dev/sda1: 
```


```bash
mount -- /dev/mapper/GnuPG-home /mnt/
```

```bash
export GNUPGHOME=/mnt 

```bash
gpgconf --list-dirs --verbose -- homedir
/mnt
```

---
Key management
always use `sudo GNUPGHOME=/mnt ...`
or use `sudo su` afterwards `export -- GNUPGHOME=/mnt`
---

```bash
sudo GNUPGHOME=/mnt gpgconf --kill --verbose -- all
```

```bash
sudo umount /mnt
```

```bash
cryptsetup close -- GnuPG-home
```




