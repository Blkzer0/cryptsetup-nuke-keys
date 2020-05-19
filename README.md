cryptsetup-nuke-keys
====================

A patch for cryptsetup which adds the option to nuke all keyslots given a certain passphrase.
Original cryptsetup patch by Juergen Pabel, found here - http://lxer.com/module/newswire/view/103692/index.html

<pre>
root@kali:~# cryptsetup luksAddNuke /dev/sda5
Enter any existing passphrase: 		(existing password)
Enter new passphrase for key slot:	(set the nuke password)
</pre>

Once the machine is rebooted and you are prompted for the LVM decryption passphrase. 
If you provide the nuke password, all password keyslots get deleted, rendering the encrypted LVM volume inaccessible.

For more details check -  http://www.kali.org/how-to/emergency-self-destruction-luks-kali

Update: As of July 2019, Kali Linux no longer ships this cryptsetup patch, instead we introduced a cryptsetup-nuke-password package that provides a similar feature without modifying cryptsetup.

Instructions copied from Kali's form:

Let’s go through the motions of encrypting, backing up, destroying, and then restoring your data using Kali Linux. Start by downloading and installing Kali Linux 1.0.6 with Full Disk Encryption. Once that is done, you can verify your information as follows:
```
root@kali-crypto:~# cryptsetup luksDump /dev/sda5
LUKS header information for /dev/sda5

" Version:        1
Cipher name:    aes
Cipher mode:    xts-plain64
Hash spec:      sha1
Payload offset: 4096
MK bits:        512
MK digest:      04 cd d0 51 bf 57 10 f5 87 08 07 d5 c8 2a 34 24 7a 89 3b db
MK salt:        27 42 e5 a6 b2 53 7f de 00 26 d3 f8 66 fb 9e 48
                16 a2 b0 a9 2c bb cc f6 ea 66 e6 b1 79 08 69 17
MK iterations:  65750
UUID:           126d0121-05e4-4f1d-94d8-bed88e8c246d

Key Slot 0: ENABLED
    Iterations:             223775
    Salt:                   7b ee 18 9e 46 77 60 2a f6 e2 a6 13 9f 59 0a 88
                            7b b2 db 84 25 98 f3 ae 61 36 3a 7d 96 08 a4 49
    Key material offset:    8
    AF stripes:             4000
Key Slot 1: DISABLED
Key Slot 2: DISABLED
Key Slot 3: DISABLED
Key Slot 4: DISABLED
Key Slot 5: DISABLED
Key Slot 6: DISABLED
Key Slot 7: DISABLED 
```

As you can see, we have slot 0 enabled with slots 1 to 7 unused. At this point, we will add our nuke key.

```
root@kali-crypto:~# apt install cryptsetup-nuke-password
root@kali-crypto:~# dpkg-reconfigure cryptsetup-nuke-password
```

This didn’t change anything to the LUKS container, instead it installed the nuke password and a small hook in the initrd. This hook will detect when you enter your nuke password at boot time and it will call “cryptsetup luksErase” on your LUKS container at that time.
Wonderful. Now we need to back up the encryption keys. This can easily be done with the “luksHeaderBackup” option.

```
root@kali-crypto:~# cryptsetup luksHeaderBackup --header-backup-file luksheader.back /dev/sda5
root@kali-crypto:~# file luksheader.back
luksheader.back: LUKS encrypted file, ver 1 [aes, xts-plain64, sha1] UUID: 126d0121-05e4-4f1d-94d8-bed88e8c246d
root@kali-crypto:~#
```
So, in our case we would like to encrypt this data for storage. There are a number of ways this could be done, however we will use openssl to make the process quick and easy using default tools in Kali.

```
root@kali-crypto:~# openssl enc -aes-256-cbc -salt -in luksheader.back -out luksheader.back.enc
enter aes-256-cbc encryption password:
Verifying - enter aes-256-cbc encryption password:
root@kali-crypto:~# ls -lh luksheader.back*
-r-------- 1 root root 2.0M Jan  9 13:42 luksheader.back
-rw-r--r-- 1 root root 2.0M Jan  9 15:50 luksheader.back.enc
root@kali-crypto:~# file luksheader.back*
luksheader.back:     LUKS encrypted file, ver 1 [aes, xts-plain64, sha1] UUID: 126d0121-05e4-4f1d-94d8-bed88e8c246d
luksheader.back.enc: data
```

Great, now we have the encrypted header ready to be backed up. In this case, we would like to place the header somewhere that it is easily accessible. This could be as simple as on a USB thumb drive that is kept in a safe location. At this point, lets reboot and make use of the Nuke key and see how Kali responds.

So we used the Nuke key, and as expected we can no longer boot into Kali. Let’s see what happened on the actual disk by booting up into a Kali live CD and dumping the LUKS header again.

```
root@kali:~# cryptsetup luksDump /dev/sda5
LUKS header information for /dev/sda5

Version:        1
Cipher name:    aes
Cipher mode:    xts-plain64
Hash spec:      sha1
Payload offset: 4096
MK bits:        512
MK digest:      04 cd d0 51 bf 57 10 f5 87 08 07 d5 c8 2a 34 24 7a 89 3b db
MK salt:        27 42 e5 a6 b2 53 7f de 00 26 d3 f8 66 fb 9e 48
                16 a2 b0 a9 2c bb cc f6 ea 66 e6 b1 79 08 69 17
MK iterations:  65750
UUID:           126d0121-05e4-4f1d-94d8-bed88e8c246d

Key Slot 0: DISABLED
Key Slot 1: DISABLED
Key Slot 2: DISABLED
Key Slot 3: DISABLED
Key Slot 4: DISABLED
Key Slot 5: DISABLED
Key Slot 6: DISABLED
Key Slot 7: DISABLED
```

As we can see, no keyslots are in use. The Nuke worked as expected. To restore the header back in place, it’s a simple matter of retrieving the encrypted header from your USB drive. Once we have that, we can decrypt it and conduct our restore:

```
root@kali:~# openssl enc -d -aes-256-cbc -in luksheader.back.enc -out luksheader.back
enter aes-256-cbc decryption password:
root@kali:~# cryptsetup luksHeaderRestore --header-backup-file luksheader.back /dev/sda5

WARNING!
========
Device /dev/sda5 already contains LUKS header. Replacing header will destroy existing keyslots.

Are you sure? (Type uppercase yes): YES
root@kali:~# cryptsetup luksDump /dev/sda5
LUKS header information for /dev/sda5

Version:        1
Cipher name:    aes
Cipher mode:    xts-plain64
Hash spec:      sha1
Payload offset: 4096
MK bits:        512
MK digest:      04 cd d0 51 bf 57 10 f5 87 08 07 d5 c8 2a 34 24 7a 89 3b db
MK salt:        27 42 e5 a6 b2 53 7f de 00 26 d3 f8 66 fb 9e 48
                16 a2 b0 a9 2c bb cc f6 ea 66 e6 b1 79 08 69 17
MK iterations:  65750
UUID:           126d0121-05e4-4f1d-94d8-bed88e8c246d

Key Slot 0: ENABLED
    Iterations:             223775
    Salt:                   7b ee 18 9e 46 77 60 2a f6 e2 a6 13 9f 59 0a 88
                            7b b2 db 84 25 98 f3 ae 61 36 3a 7d 96 08 a4 49
    Key material offset:    8
    AF stripes:             4000
Key Slot 1: DISABLED
Key Slot 2: DISABLED
Key Slot 3: DISABLED
Key Slot 4: DISABLED
Key Slot 5: DISABLED
Key Slot 6: DISABLED
Key Slot 7: DISABLED
```

Our slots are now restored. All we have to do is simply reboot and provide our normal LUKS password and the system is back to its original state.
