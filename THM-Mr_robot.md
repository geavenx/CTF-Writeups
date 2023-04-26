# Mr Robot - TryHackMe (Pawn)

## Introduction
Medium tier CTF, with Mr. Robot series theme.

## Enumeration
### Nmap and Gobuster
Starting with **nmap** scanning the target address in order to find open ports with the command:<br>
`nmap $IP -p- -T5 -sV`


Discovered two ports: SSH (22) was closed, but HTTP (80) was opened, so I went to the web server and ran gobuster on them with the command:<br>
`gobuster dir -u http://$IP/ -w /usr/share/wordlists/dirb/common.txt`

Found a lot of directories, but only two of them really caught my attention: **/robots and /wp-admin.php**

### robots.txt
Going to the **/robots** directory I found interesting files, fsociety.dic and **key-1-of-3.txt**, going to the key file (/key-1-of-3.txt) I got my first flag!<br>
`073403c8a58a1f80d943455fb30724b9`

## Exploitation
### fsociety.dic and wp-admin
In the fsociety directory you can download a dictionary of words, that is obviously a wordlist, and the wp-admin is a wordpress login page, so I decided to try to use that wordlist to crack the wordpress credentials, started using **burpsuite** to find the correct username, with the *cluster bomb* attack and the wordlist I noticed a single request a bigger length than the others, was the username **Elliot**, trying on the wp-admin with any password, it confirms to me that is the correct username. 

### wpscan
With the username in hands I could use wpscan (tried with hydra and burpsuite, but I think wpscan is the easier) and the wordlist to brute force the password, so then I tried. Using the command:<br>
`wpscan -v -U Elliot -P fsocity.dic --url 10.10.38.245/wp-login`<br>
After some waiting I got success with the credential **Elliot:ER28-0652** and head into the admin section of the wordpress.

### Reverse shell
In the admin page, after some searching I found an editor tool on the appearance section, there is an archive.php on this page, so I used the pentestmonkey php reverse shell on here (https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php) and loaded the page `10.10.38.245/wp-content/themes/twentyfifteen/archive.php` with ncat listening to my chosen port (4444 in this case) and worked! I got a reverse shell on the target.<br>
In order to get a better shell, I used the following commands: `python -c 'import pty;pty.spawn("/bin/bash")'` and `export TERM=xterm`, but you dont really have to do this. <br>

### /home/robot
Searching through the files I found inside /home/robot two interesting files: `key-2-of-3.txt` and `password.raw-md5`, I do not have access to the `key-2-of-3.txt` file, but I have access to the `password.raw-md5` obviously is a md5 hash that can be cracked, copied the hash to crackstation.net and got the password: **abcdefghijklmnopqrstuvwxyz**. Using the password to login in the robot user and got success. Tried to read `key-2-of-3.txt` logged in as robot and obtained the second flag!

## Privilege escalation
Looking for vulnerabilities I ran `find / -perm -4000 -type f 2>/dev/null` to look for SUID files that can be exploited, and then I realized that **nmap** was listed, and with some researching I discovered that is easy to exploit this SUID

### Nmap SUID
Following the vk9-sec tutorial (https://vk9-sec.com/nmap-privilege-escalation/) <br>
`$nmap --interactive`<br>
`$bash`<br>
`$exit`<br>
`$!sh`<br>
`#`<br>
a