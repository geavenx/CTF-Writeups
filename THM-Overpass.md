# Overpass - TryHackMe (pawn)

## Introduction
Quite easy CTF, good for beginners.

## Enumeration
Started with **nmap** scanning ports with the command: `nmap -T5 -sV -p- 10.10.182.114`  

With this we discovered that the target got 2 open ports: 22-ssh and 80-HTTP.
After I tried connecting to the web server by using the IP address of the target as URL.
Inside the page I found a Download section, there we got some files to download, but after looking at the files I didnt find anything useful, so I decided to run **gobuster** to find any other parent directories with the command:
`gobuster dir -u http://10.10.193.23/ -w /home/vitor/wordlists/KaliLists-master/dirb/common.txt`

Gobuster found 6 directories (**aboutus, admin, css, downloads, img, index.html**), the admin directory catch my attention, so I went there, and after trying the usual credentials (root:root, admin:admin etc.) with no success, I looked at the 	source code of the page, and found a login.js script in the html.

### login.js
By looking at login.js, we notice that every time the value of the *admin* credential are correct, a *SessionToken* cookie is created.

## Exploitation
Now, we can try to manipulate that if statement by creating a fake SessionToken cookie to login the admin section. In order to create the cookie, you need to open the developer tools of your browser on Firefox its `F12` key, now you click on cookies and in the `+` at the corner, rename the cookie to `SessionToken` and path to root `/`. Now I refresh the page and got into the /admin section, and there it is, a RSA key.

### RSA key

To crack this RSA key we can use john the ripper, just copy the entire key and paste in a file named rsa_key with nano, use `chmod 600 rsa_key` to change the access permissions and run ssh2john with the command:
`python3 ssh2john rsa_key > rsa_key.hash`
Now crack this hash with john the ripper with the command:
`john --wordlist=/usr/share/wordlists/rockyou.txt rsa_key.hash`

Okay, now we got the RSA password `james13`
Knowing the username (James) and the password (james13) we can try to secure shell in the target using the command:
`ssh -i rsa_key james@10.10.193.23`
and I got the user flag!

## Privilege escalation

That one was easy, first I sent linpeas.sh from my computer to the target with scp using the command:
`scp -i rsa_key /path/to/linpeas.sh james@10.10.193.23:/dev/shm/linpeas.sh`
Then run linpeas.sh with `chmod +x linpeas.sh` and `./linpeas.sh`.
Target is vulnerable to CVE-2021-4034!
Using arthepsy code (https://github.com/arthepsy/CVE-2021-4034) I got the root access!

So now I can just `cd /root` and `cat root.txt` in order to get the root flag!
