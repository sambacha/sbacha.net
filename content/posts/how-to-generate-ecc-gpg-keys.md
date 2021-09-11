---
title: "How to Generate Ecc Gpg Keys"
date: 2021-09-11T16:00:00-07:00
summary: "Generating non-RSA GPG Keys (e.g. Ed25519) is not a straightforward process. This guide outlines the process step by step with best practices."
---


# GPG ECC Key Generation
> [source documentation](https://www.gniibe.org/memo/software/gpg/keygen-25519.html)

## Overview

- [osx dmg installer](https://sourceforge.net/projects/gpgosx/files/)

> PIN Entry (Pinentry) is needed to submit your passphrase!

Here is an explanation of how to create your new ECC keys for GnuPG.

GnuPG 2.1.x supports ECC (Elliptic Curve Cryptography). ECC is generic
term and security of ECC depends on the curve used. Unfortunately, no
one wants to use standardized curve of NIST.

Since GnuPG 2.1.0, we can use Ed25519 for digital signing. Although it
is not yet standardized in OpenPGP WG, it's considered safer.

Ed25519 was introduced to OpenSSH already, so, we can use ssh-agent
feature of gpg-agent using authentication subkey of OpenPGP. However,
there was no encryption support for corresponding curve.

Since GnuPG 2.1.7 of August 2015, encryption by Curve25519 is supported.
It is pretty much experimental, because it requires development version
of libgcrypt (and not standardized yet).

Anyhow, here is the how to.

## Preparation

-   Development version of libgcrypt: git.gnupg.org
    
    You need to clone the master branch and build and install it by
yourself (from source). Here, we assume that it's installed in
/usr/local.
    
-   GnuPG 2.1.7 or later
    
    In Debian experimental, we have gnupg2 package.
    

## Example session log

Here is an example session log to create newer ECC keys for GnuPG.

At first, we should have LD_LIBRARY_PATH defined. And it is better to
kill existing gpg-agent because it doesn't run with LD_LIBRARY_PATH
defined.

```bash
$ export LD_LIBRARY_PATH=/usr/local/lib
$ gpg-connect-agent KILLAGENT /bye
OK closing connection
```
Next, we invoke gpg frontend with --expert and --full-gen-key option.

```bash
$ gpg2 --expert --full-gen-key
gpg (GnuPG) 2.1.8; Copyright (C) 2015 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
```
Then, we input 9 to select ECC primary key and ECC encryption subkey.

```bash
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
Your selection? 9
```
Next is the important selection. We input 1 to select "Curve25519".

```bash
Please select which elliptic curve you want:
   (1) Curve 25519
   (2) NIST P-256
   (3) NIST P-384
   (4) NIST P-521
   (5) Brainpool P-256
   (6) Brainpool P-384
   (7) Brainpool P-512
   (8) secp256k1
Your selection? 1
```
You'll see WARNING, but it is what you want.

```bash
gpg: WARNING: Curve25519 is not yet part of the OpenPGP standard.
Use this curve anyway? (y/N) y
```
It asks about expiration of key.
```bash
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0)
Key does not expire at all
Is this correct? (y/N) y
```
Then, it asks about a user ID.

```bash
GnuPG needs to construct a user ID to identify your key.

Real name: Kunisada Chuji
Email address: chuji@gniibe.org
Comment:
You selected this USER-ID:
    "Kunisada Chuji <chuji@gniibe.org>"

Lastly, it asks confirmation.

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? o
```
Then, it goes like this.

```bash
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
```
It asks the passphrase for keys by pop-up window, and then, finishes.

```bash
gpg: key 7C406DB5 marked as ultimately trusted
public and secret key created and signed.

gpg: checking the trustdb
gpg: 3 marginal(s) needed, 1 complete(s) needed, PGP trust model
gpg: depth: 0  valid:   6  signed:  67  trust: 0-, 0q, 0n, 0m, 0f, 6u
gpg: depth: 1  valid:  67  signed:  40  trust: 67-, 0q, 0n, 0m, 0f, 0u
gpg: next trustdb check due at 2015-12-05
pub   ed25519/7C406DB5 2015-09-11
      Key fingerprint = 608C 9215 646A D2C5 551B  320B 1717 4C1A 7C40
6DB5
uid         [ultimate] Kunisada Chuji <chuji@gniibe.org>
sub   cv25519/DF7B31B1 2015-09-11
```

ed25519/7C406DB5 is the primary key, and 
cv25519/DF7B31B1 is encryption subkey.

Next, we add authentication subkey which can be used with OpenSSH. We
invoke gpg frontend with --edit-key and the key ID.

```bash
$ gpg2 --expert --edit-key 7C406DB5
```
```sh
gpg (GnuPG) 2.1.8; Copyright (C) 2015 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Secret key is available.

sec  ed25519/7C406DB5
     created: 2015-09-11  expires: never       usage: SC
     trust: ultimate      validity: ultimate
ssb  cv25519/DF7B31B1
     created: 2015-09-11  expires: never       usage: E
[ultimate] (1). Kunisada Chuji <chuji@gniibe.org>
```
We invoke addkey subcommand.
```bash
gpg> addkey
```
It asks a kind of key, we input 11 to select ECC for authentication.
```bash
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
Your selection? 11
```
and then, we specify "Authenticate" capability.
```bash
Possible actions for a ECDSA key: Sign Authenticate
Current allowed actions: Sign

   (S) Toggle the sign capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? a

Possible actions for a ECDSA key: Sign Authenticate
Current allowed actions: Sign Authenticate

   (S) Toggle the sign capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? s

Possible actions for a ECDSA key: Sign Authenticate
Current allowed actions: Authenticate

   (S) Toggle the sign capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? q

Then, it asks which curve. We input 1 for "Curve25519".

Please select which elliptic curve you want:
   (1) Curve 25519
   (2) NIST P-256
   (3) NIST P-384
   (4) NIST P-521
   (5) Brainpool P-256
   (6) Brainpool P-384
   (7) Brainpool P-512
   (8) secp256k1
Your selection? 1
```
It asks confirmation. We say y.
```bash
gpg: WARNING: Curve25519 is not yet part of the OpenPGP standard.
Use this curve anyway? (y/N) y

It asks expiration of the key.

Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0)
Key does not expire at all
Is this correct? (y/N) y

And the confirmation.

Really create? (y/N) y
```
It goes.
```bash
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.

It asks the passphrase. And done.

sec  ed25519/7C406DB5
     created: 2015-09-11  expires: never       usage: SC
     trust: ultimate      validity: ultimate
ssb  cv25519/DF7B31B1
     created: 2015-09-11  expires: never       usage: E
ssb  ed25519/8679DF5F
     created: 2015-09-11  expires: never       usage: A
[ultimate] (1). Kunisada Chuji <chuji@gniibe.org>
```
We type save to exit form gpg.
```bash
gpg> save
```

### Note
