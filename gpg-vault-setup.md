
# GnuPG Vault Setup

## Prerequisites

### Software

- cryptsetup
- fdisk
- udisksctl
- ykman

### Hardware

- yubikey
- At least 2 storage devices 2Gb each (recommended 3)
  - 1 as GnuPG home
  - 1 as backup locally
  - 1 as backup in a different physical location

## Create encrypted vault


### Important Note

- In the process all data will be wiped from the storage device.
- We recommend to use the storage device for the single purpose it was setup (GnuPG home or backup), nothing else!
- After the vault setup, you should only use these vaults on an air-gapped machine
- Store all generated passwords during this process in your preferred password manager


### Prepare storage device


Get devices `lsblk`, the we will use `sda` in the following example
https://wiki.archlinux.org/title/Dm-crypt/Device_encryption


Insert your storage device. If it is mounted automatically, you need to run:

```bash
udisksctl unmount --block-device=/dev/sda1
```

Desired output (not mounted):

```bash
❯ lsblk                                     
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda           8:0    1   7,5G  0 disk 
└─sda1        8:1    1   7,5G  0 part 
...
```

#### Create partition - fdisk

start `fdisk` as root:

```bash
sudo fdisk /dev/sda 
```

create a new empty GPT partition table:

```bash
Command (m for help): g
```

add a new partition with at least `1G` of storage:

```bash
Command (m for help): n
Partition number (1-128, default 1): 1
First sector (2048-15728606, default 2048): 2048
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-15728606, default 15728606): +1G

Do you want to remove the signature? [Y]es/[N]o: Y
```

save changes on the disk
```bash
Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
``` 

####  overwrite partition with random data - dd

fill the partition with bullshit:

```bash
sudo dd bs=32MiB conv=fsync if=/dev/urandom  oflag=direct status=progress of=/dev/sda1
```

#### Encrypt partition - cryptsetup

First generate a long password e.g. 128 characters as the first passphrase.
Store this long passphrase in your password manager e.g. as `GnuGP home vault master password`.

Encrypt the partition:

```bash
$ sudo cryptsetup --type luks2 --cipher aes-xts-plain64 --hash sha512 --integrity hmac-sha512 --iter-time 10000 --key-size 512 --pbkdf argon2id  --pbkdf-memory 4194304  --pbkdf-parallel 8 --use-urandom luksFormat /dev/sda1

WARNING!
========
This will overwrite data on /dev/sda1 irrevocably.

Are you sure? (Type 'yes' in capital letters): YES
```

Use your GnuGP Vault Master Password

```bash
Enter passphrase for /dev/sda1: 
Verify passphrase: 

```

#### Add another key (optional) - ykman

This step is optional, but highly recommended. Typcially you won't have access to your password manager on your air gapped machine, this is why we will setup a two component password.
The yubikey will provide strong password and you can remember a simple of the password.

**Important Node**: The layout you will use to generate the password, will be set up later on the air gapped machine.

We will use the german layout here:

```bash
ykman otp static --keyboard-layout DE --generate --length 38 -- 2
Slot 2 is already configured. Overwrite configuration? [y/N]: y
```

This command will store a random string of 38 characters on the second slot of your yubikey.
Which means you can access it by long-pressing your yubikey.

Now as for the two component password:

Your easy to type password: banana
Random generated password on the yubikey: tATXo4rB$$55dT2a#mJLnM9cQcT%yEgaZezXaB

<span style="color: red">banana</span><span style="color: blue">tATXo4rB$$55dT2a#mJLnM9cQcT%yEgaZezXaB</span>

Store the complete password in your password manager as `GnuGP Vault 


Generate another keyslot with the two component password:

```bash
sudo cryptsetup luksAddKey /dev/sda1 
```

### Test and protect

Open the encrypted partition. Use your vault master password or the two component password:

```bash
sudo cryptsetup open -- /dev/sda1 GnuPG-home
Enter passphrase for /dev/sda1: 
```

Desired output of `lsblk`:

```bash
lsblk 
NAME           MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINTS
sda              8:0    1   7,5G  0 disk  
└─sda1           8:1    1     1G  0 part  
  └─GnuPG-home 252:0    0  1008M  0 crypt 

```

### Protection against bitrot - mkfs.btrfs

Use `mkfs.btrfs` with checksums against bitrot and snapshots as first-line backups:

```bash
sudo mkfs.btrfs --checksum=blake2 --label=GnuPG-home -- /dev/mapper/GnuPG-home
```


```bash
sudo cryptsetup close -- GnuPG-home
```
