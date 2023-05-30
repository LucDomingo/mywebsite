---
title: 'How can I cipher disk on my centos 8 ?'
date: 2023-05-27
permalink: /posts/2023/05/ciphering-disk-linux/
tags:
  - LUKS
  - Ciphering
  - CentOS 8
---

Everything is in the title, simple question and concise answer in this post

Introduction
======
To cipher disk on CentOS 8, we can use the LUKS (Linux Unified Key Setup) wich is the standard disk encryption system for Linux (it's widely supported in CentOS and other linux distributions).

Step-by-step guide
======
**Warning** : Before starting, be aware that the encryption process involves modifying disk partitions, whivh can lead to data loss if something goes wrong. It's crucial to have a backup of your important data before proceeding.

First install the necessary packages :
<pre>
[root@centos8 ~]# ls yum install cryptsetup
</pre>

Indentify the disk to cipher, for example /dev/sdb.
If the disk contains any mounted partitions, we need to use **umount** to unmount them :
<pre>
[root@centos8 ~]# umount /dev/sdb1 /dev/sdb2 ...
</pre>

Once it's done, we can initiate the encryption process: 
<pre>
[root@centos8 ~]# cryptsetup luksFormat /dev/sdb

WARNING!
========
Cette action écrasera définitivement les données sur /dev/sdb.

Are you sure? (Type 'yes' in capital letters): YES
Saisissez la phrase secrète pour /dev/sdb : 
Vérifiez la phrase secrète : 
</pre>

The command prompt you to enter and confirm a passphrase. **We will need it to unlock the disk later**.
Now our disk is encrypted with LUKS.

But we need to create a mapping to the encrypted disk. LUKS provides a layer of abstraction between the encrypted disk and the decrypted data, allowing the operating system and applications to interact with the disk as if it were unencrypted.

When you encrypt a disk with LUKS, the encrypted data is stored on the physical disk, and the encryption key is securely stored within the LUKS header. To access the data on the encrypted disk, you need to provide the encryption key, which is typically a passphrase. 

Creating a mapping with `cryptsetup luksOpen` command establishes a connection between the encrypted disk and a virtual device in the `/dev/mapper/` directory. This virtual device represents the decrypted view of the encrypted disk. It allows you to perform operations such as creating a file system, mounting the disk, and accessing the data. That's why only the owner of the passphrase can do this kind of operation.

By creating this mapping, the LUKS system handles the decryption process transparently whenever data is read from or written to the encrypted disk. The mapping ensures that the data remains encrypted on the physical disk, and it is decrypted on-the-fly when accessed through the virtual device. 

Let's create the mapping device:
<pre>
[root@centos8 ~]# cryptsetup luksOpen /dev/sdb encrypted_sdb
Saisissez la phrase secrète pour /dev/sdb :
[root@centos8 ~]# lsblk /dev/sdb
NAME            MAJ:MIN RM SIZE RO TYPE  MOUNTPOINT
sdb               8:16   0   8G  0 disk  
└─encrypted_sdb 253:3    0   8G  0 crypt 
</pre>

Now we have unlock the encrypted disk we can create a filsesystem and mount itmkd:
<pre>
[root@centos8 ~]# mkfs.ext4 /dev/mapper/encrypted_sdb 
mke2fs 1.45.6 (20-Mar-2020)
En train de créer un système de fichiers avec 2093056 4k blocs et 523264 i-noeuds.
UUID de système de fichiers=985c7581-195e-47d3-babb-9365fcabec03
Superblocs de secours stockés sur les blocs : 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Allocation des tables de groupe : complété                        
Écriture des tables d'i-noeuds : complété                        
Création du journal (16384 blocs) : complété
Écriture des superblocs et de l'information de comptabilité du système de
fichiers : complété

[root@centos8 ~]# mkdir /mnt/my_encrypted_fs

[root@centos8 ~]# mount /dev/mapper/encrypted_sdb /mnt/my_encrypted_fs/

[root@centos8 ~]# cryptsetup status encrypted_sdb 
/dev/mapper/encrypted_sdb is active and is in use.
  type:    LUKS2
  cipher:  aes-xts-plain64
  keysize: 512 bits
  key location: keyring
  device:  /dev/sdb
  sector size:  512
  offset:  32768 sectors
  size:    16744448 sectors
  mode:    read/write
</pre>

We can see the default cipher used is aes-xts-plain64 and the default key size is 512 bits.

LUKS under the hood
======


Resources
======
- [https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/security_hardening/encrypting-block-devices-using-luks_security-hardening](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/security_hardening/encrypting-block-devices-using-luks_security-hardening)
