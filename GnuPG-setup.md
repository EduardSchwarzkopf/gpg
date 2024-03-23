# GnuPG Setup

> All steps below should be done on an air-gapped machine

This setup needs to be setup only on the initial GnuPG setup.

## Prerequisites

- setup your encrypted vault first.
- Arch linux live system
- `gpg` at least version `2.4`


## Mount encrypted vault

!! AIR GAPPED MACHINE !!

> Run all commands below as root.

set keyboard layout with;

```bash
localectl --no-convert set-keymap de-latin1-nodeadkeys
```

Open the encrypted vault:

```bash
cryptsetup open -- /dev/sda1 GnuPG-home
Enter passphrase for /dev/sda1: 
```


```bash
mkdir -p /mnt/gpg
mount -- /dev/mapper/GnuPG-home /mnt/gpg
```

## GnuPG setup

```bash
export GNUPGHOME=/mnt/gpg
``

```bash
echo require-secmem >> ${GNUPGHOME}/gpg.conf
echo ask-cert-level >> ${GNUPGHOME}/gpg.conf
echo ask-cert-expire >> ${GNUPGHOME}/gpg.conf
echo 'cert-digest-algo SHA512' >> ${GNUPGHOME}/gpg.conf
```


## Save and close

```bash
gpgconf --kill all
sudo umount /mnt
cryptsetup close -- GnuPG-home
```

