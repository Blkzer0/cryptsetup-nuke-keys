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

root@kali:~# apt install cryptsetup-nuke-password
root@kali:~# dpkg-reconfigure cryptsetup-nuke-password
