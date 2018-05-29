---
title:  "Part 3: A Fresh-Baked Holiday Pi"
search: true
excerpt: "SANS Holiday Hack 2016 Writeup - The Challenges - Part 3"
header:
  #image: /assets/images/foo-bar-identity.jpg
  teaser: /assets/images/sans-2016.png
categories:
  - SANS
  - Holday Hack
  - writeup
classes: wide
last_modified_at: 2016-03-06T19:56:50+01:00
---

Question 5: What is the password for the 'cranpi' account on the Cranberry Pi system?
{: .notice--danger}
Question 6: How did you open each terminal door and where had the villain imprisoned Santa?
{: .notice--danger}

After gathering all the pieces to the Cranberry Pi system from the North Pole (a [Quest](https://quest2016.holidayhackchallenge.com) objective), you are given a [link](https://www.northpolewonderland.com/cranbian.img.zip) to a ZIP file containing a "Cranbian" image.

For this section, I transferred the cranbian.img.zip file over to a [Kali Linux](https://www.kali.org/) virtual machine and continued there.

The SANS blog post titled [Mount a Raspberry Pi File System Image](https://pen-testing.sans.org/blog/2016/12/07/mount-a-raspberry-pi-file-system-image) walks you through the steps needed to get going. However, I'll still show the exact commands I used.

Extracting the image...

```yaml
root@gh0st1:/# unzip cranbian.img.zip 
Archive:  cranbian.img.zip
  inflating: cranbian-jessie.img
```

Using `fdisk` to get important information about the image (partition and sector sizes/types/offsets)...

```yaml 
root@gh0st1:/# fdisk -l cranbian-jessie.img
Disk cranbian-jessie.img: 1.3 GiB, 1389363200 bytes, 2713600 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x5a7089a1

Device               Boot  Start     End Sectors  Size Id Type
cranbian-jessie.img1        8192  137215  129024   63M  c W95 FAT32 (LBA)
cranbian-jessie.img2      137216 2713599 2576384  1.2G 83 Linux
```

Based on the `fdisk` output, cranbian-jessie.img2 is our primary target.  It's the largest partition (1.2G) and uses the Linux file system.

Before mounting it, I need to know where the file system begins.  This is done by multiplying the sector size (512 bytes) and the start sector for the Linux file system (137216) I come up with the total file system size in bytes (70254592).

```yaml 
root@gh0st1:/# echo $((512*137216))
70254592
``` 
Now to make a directory to mount the image...

```yaml 
root@gh0st1:/# mkdir /root/cranbian
```

Mounting the image to that directory...

```yaml 
root@gh0st1:/# mount -v -o offset=70254592 -t ext4 /cranbian-jessie.img /root/cranbian
mount: /dev/loop1 mounted on /root/cranbian.
```

You can navigate the directory structure as you would normally.

```yaml
root@gh0st1:/# cd /root/cranbian/
root@gh0st1:~/cranbian# ls -la
total 132
drwxr-xr-x 21 root root 36864 Dec  5 16:09 .
drwxr-xr-x 33 root root  4096 Dec 20 05:01 ..
drwxr-xr-x  2 root root  4096 Nov 23 10:11 bin
drwxr-xr-x  2 root root  4096 Sep 22 23:52 boot
drwxr-xr-x  4 root root  4096 Sep 22 22:23 dev
drwxr-xr-x 77 root root  4096 Dec  5 11:25 etc
drwxr-xr-x  3 root root  4096 Nov 21 10:25 home
drwxr-xr-x 17 root root  4096 Nov 23 10:07 lib
drwx------  2 root root 16384 Sep 22 23:52 lost+found
drwxr-xr-x  2 root root  4096 Sep 22 22:20 media
drwxr-xr-x  2 root root  4096 Sep 22 22:20 mnt
drwxr-xr-x  3 root root  4096 Sep 22 22:27 opt
drwxr-xr-x  2 root root  4096 Jan  6  2015 proc
drwx------  2 root root  4096 Nov 23 10:14 root
drwxr-xr-x  5 root root  4096 Sep 22 22:28 run
drwxr-xr-x  2 root root  4096 Sep 22 22:39 sbin
drwxr-xr-x  2 root root  4096 Sep 22 22:20 srv
drwxr-xr-x  2 root root  4096 Apr 12  2015 sys
drwxrwxrwt  7 root root  4096 Nov 17 15:17 tmp
drwxr-xr-x 10 root root  4096 Sep 22 22:20 usr
drwxr-xr-x 11 root root  4096 Nov 23 10:10 var
root@gh0st1:~/cranbian# 
```

To crack the password of a local Linux account with `john` you'll need two things, the /etc/shadow file and the /etc/passwd file.  The `unshadow` tool combines the passwd and shadow files so `john` can attempt to crack them against a wordlist.  Both `unshadow` and `john` are distributed with [John the Ripper](http://www.openwall.com/john/) which comes preconfigured in Kali.  Since I had access to the Cranbian file system now, the rest is pretty straightforward.

Running `unshadow` without any parameters will tell you what it is looking for.

```yaml
root@gh0st1:~/# unshadow 
Usage: unshadow PASSWORD-FILE SHADOW-FILE
```

Specify the file locations...

```yaml
root@gh0st1:~/# unshadow /root/cranbian/etc/passwd /root/cranbian/etc/shadow
root:*:0:0:root:/root:/bin/bash
daemon:*:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:*:2:2:bin:/bin:/usr/sbin/nologin
sys:*:3:3:sys:/dev:/usr/sbin/nologin
sync:*:4:65534:sync:/bin:/bin/sync
games:*:5:60:games:/usr/games:/usr/sbin/nologin
man:*:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:*:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:*:8:8:mail:/var/mail:/usr/sbin/nologin
news:*:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:*:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:*:13:13:proxy:/bin:/usr/sbin/nologin
www-data:*:33:33:www-data:/var/www:/usr/sbin/nologin
backup:*:34:34:backup:/var/backups:/usr/sbin/nologin
list:*:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:*:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:*:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:*:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-timesync:*:100:103:systemd Time Synchronization,,,:/run/systemd:/bin/false
systemd-network:*:101:104:systemd Network Management,,,:/run/systemd/netif:/bin/false
systemd-resolve:*:102:105:systemd Resolver,,,:/run/systemd/resolve:/bin/false
systemd-bus-proxy:*:103:106:systemd Bus Proxy,,,:/run/systemd:/bin/false
messagebus:*:104:109::/var/run/dbus:/bin/false
avahi:*:105:110:Avahi mDNS daemon,,,:/var/run/avahi-daemon:/bin/false
ntp:*:106:111::/home/ntp:/bin/false
sshd:*:107:65534::/var/run/sshd:/usr/sbin/nologin
statd:*:108:65534::/var/lib/nfs:/bin/false
cranpi:$6$2AXLbEoG$zZlWSwrUSD02cm8ncL6pmaYY/39DUai3OGfnBbDNjtx2G99qKbhnidxinanEhahBINm/2YyjFihxg7tgc343b0:1000:1000:,,,:/home/cranpi:/bin/bash
```
There is the output above.  This time I will save it to a text file.

```yaml
root@gh0st1:~/# unshadow /root/cranbian/etc/passwd /root/cranbian/etc/shadow > /root/unshadow.txt
```

Thanks to tips received from talking with the elves during the [Quest](https://quest2016.holidayhackchallenge.com), the popular `rockyou.txt` wordlist will be used to attempt to crack the password.

```yaml
root@gh0st1:~/# john /root/unshadow.txt --wordlist=/usr/share/wordlists/rockyou.txt
Warning: detected hash type "sha512crypt", but the string is also recognized as "crypt"
Use the "--format=crypt" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 128/128 AVX 2x])
Press 'q' or Ctrl-C to abort, almost any other key for status
0g 0:00:05:06 0.94% (ETA: 03:17:10) 0g/s 519.9p/s 519.9c/s 519.9C/s ilovedogs2..hustlers
0g 0:00:05:18 0.98% (ETA: 03:15:48) 0g/s 521.2p/s 521.2c/s 521.2C/s tony89..tina101
0g 0:00:05:19 0.98% (ETA: 03:15:30) 0g/s 521.4p/s 521.4c/s 521.4C/s spiky..solveig
0g 0:00:09:37 1.83% (ETA: 02:57:38) 0g/s 532.7p/s 532.7c/s 532.7C/s hateall..harley143
0g 0:00:10:14 1.95% (ETA: 02:56:13) 0g/s 534.1p/s 534.1c/s 534.1C/s zablan..yuhuu
0g 0:00:10:15 1.96% (ETA: 02:56:09) 0g/s 534.2p/s 534.2c/s 534.2C/s woodpony..winx club
yummycookies     (cranpi)
1g 0:00:14:24 DONE (2016-12-11 18:26) 0.001156g/s 525.2p/s 525.2c/s 525.2C/s yveth..yulyul
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```
After about 14 minutes in my VM with 2GB of RAM, the password was discovered.

Question 5: What is the password for the 'cranpi' account on the Cranberry Pi system?
**yummycookies**
{: .notice}

Question 6: How did you open each terminal door and where had the villain imprisoned Santa?
This feature was so awesome, it deserved a separate section in my write-up.  See the [terminals](/terminals/) section for answers to each challenge.  As for Santa, he was imprisoned in his very own Dungeon For Errant Reindeer (DFER), but he had no memory of what happened or who had imprisoned him.  Finding him required beating the OUTATIME Challenge and traveling back in time to 1978.
{: .notice}
