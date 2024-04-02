# Key setup

## Prerequisites

- setup your encrypted vault first.
- created Keys in the encrypted vault
- Arch linux live system
- `gpg` at least version `2.4`

## Setup

!! AIR GAPPED MACHINE !!

Run all commands below as root.

set keyboard layout to `de` with:

```bash
localectl --no-convert set-keymap de-latin1-nodeadkeys
```

check your vault with `lsblk`. We will use `sdb` from now on.

Open your GnuPG vault:

```bash
cryptsetup open -- /dev/sdb1 GnuPG-home
Enter passphrase for /dev/sdb1: 
```

mount your vault:

```bash
mkdir -p /mnt/gpg
mount -- /dev/mapper/GnuPG-home /mnt/gpg
```

set the GnuPG home variable:

```bash
export GNUPGHOME=/mnt/gpg
```

Check if the homedir is set correctly:

```bash
gpgconf --list-dirs --verbose -- homedir
/mnt/gpg
```

## Renew key

```bash
gpg --list-keys
```

You can use `Tab` to pick the right key

```bash
gpg --edit-key KEYID
```

### Renew Cert key

```bash
gpg> expire
```

Recommended value is: `3m`

Confirm with `y`

### Renew Subkeys

Pick all other subkeys:

```bash
gpg> key 1
gpg> key 2
gpg> key <n>
```

renew:

```bash
gpg> expire
```

Confirm that you want to change multiple keys with `y`

Recommended value is: `3m`

Confirm with `y`

### Trust

```bash
gpg> key 0
gpg> trust
```

pick any number you feeld comfortable with. Since this is you, you can pick `5`.

### Save

```bash
gpg> save
```

## Export


### Move to yubikey


Insert you yubikey and check if it is recognized:

```bash
gpg --card-status
```

Now move the keys onto your yubikey:

```bash
gpg --edit-key KEYID
```

Signing Key:

```bash
gpg> key 1
gpg> keytocard
```

Your selection: `1` (Signature Key)

Enter the `AdminPin`

If none was set before the default Key is `12345678`, after that you will be asked to provide a new one.

Next the authentication key:

```bash
gpg> key 0
gpg> key 2
gpg> keytocard
```

Enter the `AdminPin`

**Quit without saving:**

```bash
gpg> quit
```

### Export public key

```bash
gpg --export --output /root/public.gpg
```

**!IMPORTANT!** Before inserting a new storage device, first close your vault:

```bash
gpgconf --kill all
umount -- "${GNUPGHOME}"
cryptsetup close gnupgp
```


Now insert a new storage device. List all usb devices with:

```bash
lsblk
```

create a folder for your storage device and mount it:

(replace `sdx` with the correct name)

```bash
mkdir -p /mnt/usb
mount -- /dev/sdx1 /mnt/gpg
```

transfer your public key to your storage device:

```bash
mv /root/public.gpg /mnt/usb
```

unmount it:

```bash
umount /dev/sdx1
```

## Import

Do this step on your day to day machine.

Insert your storage device with the `public.gpg` key and import the material with

```bash
gpg --import /path/to/public.gpg
```

Upload your key to any public keyserver, e.g. <https://keys.openpgp.org/>


## Sign commits

Check out this tutorial on how to [tell Git about your signing key](https://docs.github.com/en/authentication/managing-commit-signature-verification/telling-git-about-your-signing-key)