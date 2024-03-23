# Key setup

## Prerequisites

- setup your encrypted vault first.
- Arch linux live system
- `gpg` at least version `2.4`

## Setup

!! AIR GAPPED MACHINE !!

Run all commands below as root.

set keyboard layout with;

```bash
localectl --no-convert set-keymap de-latin1-nodeadkeys
```

check your vault with `lsblk`. We will use `sda` from now on.

Open your GnuPG vault:

```bash
cryptsetup open -- /dev/sda1 GnuPG-home
Enter passphrase for /dev/sda1: 
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

## Generate key

```bash
gpg --full-generate-key --expert
```

```bash
gpg --full-generate-key --expert
gpg (GnuPG) 2.2.27; Copyright (C) 2021 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

gpg: keybox '/mnt/pubring.kbx' created
Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
   (9) ECC and ECC
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
  (13) Existing key
  (14) Existing key from card
Your selection? 11

Possible actions for a ECDSA/EdDSA key: Sign Certify Authenticate 
Current allowed actions: Sign Certify 

   (S) Toggle the sign capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? S

Possible actions for a ECDSA/EdDSA key: Sign Certify Authenticate 
Current allowed actions: Certify 

   (S) Toggle the sign capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? Q
Please select which elliptic curve you want:
   (1) Curve 25519
   (3) NIST P-256
   (4) NIST P-384
   (5) NIST P-521
   (6) Brainpool P-256
   (7) Brainpool P-384
   (8) Brainpool P-512
   (9) secp256k1
Your selection? 1
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 2024-03-28
Key expires at Thu 28 Mar 2024 12:00:00 PM CET
Is this correct? (y/N) y

GnuPG needs to construct a user ID to identify your key.

Real name: Eduard Schwarzkopf
Email address: eschwarzkopf@evoila.de
Comment: key for evoila
You selected this USER-ID:
    "Eduard Schwarzkopf (key for evoila) <eschwarzkopf@evoila.de>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
Please enter the passphrase to
protect your new key
Passphrase: 
Repeat: 
You have not entered a passphrase - this is in general a bad idea!
Please confirm that you do not want to have any protection on your key.
  Yes, protection is not needed
  Enter new passphrase
[ye]? y
gpg: /mnt/trustdb.gpg: trustdb created
gpg: key 7F95CD115AEBC96E marked as ultimately trusted
gpg: directory '/mnt/openpgp-revocs.d' created
gpg: revocation certificate stored as '/mnt/openpgp-revocs.d/2A80B6BC905635B411B7D5127F95CD115AEBC96E.rev'
public and secret key created and signed.

pub   ed25519 2024-03-22 [C] [expires: 2024-03-28]
      2A80B6BC905635B411B7D5127F95CD115AEBC96E
uid                      Eduard Schwarzkopf (key for evoila) <eschwarzkopf@evoila.de>

```

### Generate Subkeys

```bash
gpg --edit-key --expert -- ${fingerprint_of_new_personal_key}
gpg (GnuPG) 2.2.27; Copyright (C) 2021 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Secret key is available.

sec  ed25519/7F95CD115AEBC96E
     created: 2024-03-22  expires: 2024-03-28  usage: C   
     trust: ultimate      validity: ultimate
[ultimate] (1). Eduard Schwarzkopf (key for evoila) <eschwarzkopf@evoila.de>

gpg> addkey 
Please select what kind of key you want:
   (3) DSA (sign only)
   (4) RSA (sign only)
   (5) Elgamal (encrypt only)
   (6) RSA (encrypt only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
  (12) ECC (encrypt only)
  (13) Existing key
  (14) Existing key from card
Your selection? 10
Please select which elliptic curve you want:
   (1) Curve 25519
   (3) NIST P-256
   (4) NIST P-384
   (5) NIST P-521
   (6) Brainpool P-256
   (7) Brainpool P-384
   (8) Brainpool P-512
   (9) secp256k1
Your selection? 1
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 2024-03-28
Key expires at Thu 28 Mar 2024 12:00:00 PM CET
Is this correct? (y/N) y
Really create? (y/N) y
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
Please enter the passphrase to
protect your new key
Passphrase: 
Repeat: 
You have not entered a passphrase - this is in general a bad idea!
Please confirm that you do not want to have any protection on your key.
  Yes, protection is not needed
  Enter new passphrase
[ye]? y

sec  ed25519/7F95CD115AEBC96E
     created: 2024-03-22  expires: 2024-03-28  usage: C   
     trust: ultimate      validity: ultimate
ssb  ed25519/22B32A977C3E91FA
     created: 2024-03-22  expires: 2024-03-28  usage: S   
[ultimate] (1). Eduard Schwarzkopf (key for evoila) <eschwarzkopf@evoila.de>

```

#### Authentication Key

```bash
gpg> addkey 
Please select what kind of key you want:
   (3) DSA (sign only)
   (4) RSA (sign only)
   (5) Elgamal (encrypt only)
   (6) RSA (encrypt only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
  (12) ECC (encrypt only)
  (13) Existing key
  (14) Existing key from card
Your selection? 11

Possible actions for a ECDSA/EdDSA key: Sign Authenticate 
Current allowed actions: Sign 

   (S) Toggle the sign capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? S

Possible actions for a ECDSA/EdDSA key: Sign Authenticate 
Current allowed actions: 

   (S) Toggle the sign capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? A

Possible actions for a ECDSA/EdDSA key: Sign Authenticate 
Current allowed actions: Authenticate 

   (S) Toggle the sign capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? Q
Please select which elliptic curve you want:
   (1) Curve 25519
   (3) NIST P-256
   (4) NIST P-384
   (5) NIST P-521
   (6) Brainpool P-256
   (7) Brainpool P-384
   (8) Brainpool P-512
   (9) secp256k1
Your selection? 1
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 2024-03-28
Key expires at Thu 28 Mar 2024 12:00:00 PM CET
Is this correct? (y/N) y
Really create? (y/N) y
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
Please enter the passphrase to
protect your new key
Passphrase: 
Repeat: 
You have not entered a passphrase - this is in general a bad idea!
Please confirm that you do not want to have any protection on your key.
  Yes, protection is not needed
  Enter new passphrase
[ye]? y

sec  ed25519/7F95CD115AEBC96E
     created: 2024-03-22  expires: 2024-03-28  usage: C   
     trust: ultimate      validity: ultimate
ssb  ed25519/22B32A977C3E91FA
     created: 2024-03-22  expires: 2024-03-28  usage: S   
ssb  ed25519/12D57EAFDEF2AB2D
     created: 2024-03-22  expires: 2024-03-28  usage: A   
[ultimate] (1). Eduard Schwarzkopf (key for evoila) <eschwarzkopf@evoila.de>
```

#### Send Keys to card

Signing Key:

```bash
gpg> key 1
gpg> keytocard
```

Next the authentication key:

```bash
gpg> key 0
gpg> key 2
gpg> keytocard
```

Quit without saving
```bash
gpg> quit
```

#### export

```bash
gpg --export --output /tmp/public_keys.gpg
```

Move `public_keys.gpg` to different storage device to make it public on an internet connected device.


#### import 

```bash
gpg --import /tmp/public_keys.gpg
```

Import on your day to day computer. Not on your air gapped device!


## Backup 

!! AIR GAPPED !!

- Different storage type
- Different geo location

look for 

```bash
tar --create --directory=${GNUPGHOME} --file=GnuPG_home.tar --verbose -- .
./
```

### finish operation

```bash
gpgconf --kill all
umount -- "${GNUPGHOME}"
cryptsetup close gnupgp
```
