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

root@kali: apt install cryptsetup-nuke-password
root@kali: dpkg-reconfigure cryptsetup-nuke-password

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
