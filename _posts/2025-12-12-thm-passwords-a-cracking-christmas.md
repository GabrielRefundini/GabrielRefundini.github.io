---
layout: post
title: "TryHackMe: Passwords - A Cracking Christmas"
date: 2025-12-12 17:33:44 -0300
categories: TryHackMe CTF
tags: crack password
author: yaza
media_subpath:
image:
  path: /5fc2847e1bbebc03aa89fbf2-1763007383191.png
---


Damn, its a rainy cloudy start of the night.. perfect vibe.
Learn how to crack password-based encrypted files.

[Room Link](https://tryhackme.com/room/attacks-on-ecrypted-files-aoc2025-asdfghj123)

command ```file name``` point whats the file type

```bash
$ pdf2john flag.pdf > flagpdf_hash

```
I tried to identify the hash type on Hash ID, but actually it makes no sense :P

Lets make it simple
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt flagpdf_hash

gpdf_hash 
Using default input encoding: UTF-8
Loaded 1 password hash (PDF, PDF encrypted document [MD5-RC4 / SHA2-AES 32/64])
Cost 1 (revision) is 4 for all loaded hashes
Cost 2 (key length) is 128 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, 'h' for help, almost any other key for status



naughtylist      (flag.pdf)  


   
1g 0:00:00:00 DONE (2025-12-12 21:45) 16.67g/s 40533p/s 40533c/s 40533C/s avatar..tootie
Use the "--show --format=PDF" options to display all of the cracked passwords reliably
Session completed

```

Found it, ezpz... turned on some random [dub techno](https://youtu.be/or4zmXBAvIU)and let's move on.
Just realized I don't know how to open a open on terminal, lol... gemini time.

```bash
$ pdftotext flag.pdf 
Command Line Error: Incorrect password
```

```bash
$ pdftotext --help

  -opw <string>        : owner password (for encrypted files)
  -upw <string>        : user password (for encrypted files)

```
User Password is the opening pass, reading and limited pass.
Owner Password is the permission pass, full pass.

```bash
$ pdftotext flag.pdf -opw naughtylist
$
```

Got no answer...
On linux the options/flags should come before the file name.
```bash
ubuntu@tryhackme:~/Desktop$ pdftotext -opw naughtylist flag.pdf
ubuntu@tryhackme:~/Desktop$ ls
flag.pdf  flag.txt  flag.zip  flagpdf_hash  john  mate-terminal.desktop

```
Oh actually it worked and created flag.txt, lets open it!
```bash
ubuntu@tryhackme:~/Desktop$ cat flag.txt
THM{Cr4ck1ng_PDFs_1s_34$y}
```

Now lets go for the zip file. 
```bash
ubuntu@tryhackme:~/Desktop$ unzip flag.zip 
Archive:  flag.zip
   skipping: flag.txt                unsupported compression method 99
```

unzip doesnt support zip files with AES, lets try 7z

```bash
$ 7z x flag.zip

7-Zip 23.01 (x64) : Copyright (c) 1999-2023 Igor Pavlov : 2023-06-20
 64-bit locale=C.UTF-8 Threads:2 OPEN_MAX:1024

Scanning the drive for archives:
1 file, 245 bytes (1 KiB)

Extracting archive: flag.zip
--
Path = flag.zip
Type = zip
Physical Size = 245

    
Enter password (will not be echoed):

```

Ok we need to find out the password.
```bash
$ zip2john flag.zip > flagzip_hash.txt
```

```bash
$ john --wordlist=/usr/share/wordlists/rockyou.txt flagzip_hash.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (ZIP, WinZip [PBKDF2-SHA1 256/256 AVX2 8x])
Cost 1 (HMAC size [KiB]) is 1 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, 'h' for help, almost any other key for status


winter4ever      (flag.zip/flag.txt)   

 
1g 0:00:00:00 DONE (2025-12-12 22:15) 2.222g/s 9102p/s 9102c/s 9102C/s friend..sahara
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

```bash
$ 7z x flag.zip 

7-Zip 23.01 (x64) : Copyright (c) 1999-2023 Igor Pavlov : 2023-06-20
 64-bit locale=C.UTF-8 Threads:2 OPEN_MAX:1024

Scanning the drive for archives:
1 file, 245 bytes (1 KiB)

Extracting archive: flag.zip
--
Path = flag.zip
Type = zip
Physical Size = 245

    
Would you like to replace the existing file:
  Path:     ./flag.txt
  Size:     0 bytes
  Modified: 2025-10-03 11:56:03
with the file from archive:
  Path:     flag.txt
  Size:     29 bytes (1 KiB)
  Modified: 2025-10-03 11:56:03
? (Y)es / (N)o / (A)lways / (S)kip all / A(u)to rename all / (Q)uit? y

               
Enter password (will not be echoed):
Everything is Ok

Size:       29
Compressed: 245
ubuntu@tryhackme:~/Desktop$ cat flag.txt

THM{Cr4ck1n6_z1p$_1s_34$yyyy}
```

GG!
Took me around 60m to setup everything, do the room and write. Was a easy one!

OOoh wait, theres a hidden key for a side quest, lets try to find it! Looks really hard to find... I'll skip that for now since even veterans are having difficulty to find it, ggs for now.
![[/aoc-house.webp]]

----

Other information:
- PDF: `pdfcrack`, `john` (via `pdf2john`)
- ZIP: `fcrackzip`, `john` (via `zip2john`)
- General: `john` (very flexible) and `hashcat` (GPU acceleration, more advanced)
- Jump boxes = jump servers
- It's worth noting that on Windows systems, Sysmon Event ID 1 captures process creation with full command line properties, while on Linux, `auditd`, `execve`, or EDR sensors capture binaries and arguments
- GPU cracking is loud.
	- - `nvidia-smi` shows long‑running processes named `hashcat` or `john`.
	- High, steady GPU utilisation and power draw while the fan curve spikes.
	- Libraries loaded: `nvcuda.dll`, `OpenCL.dll`, `libcuda.so`, `amdocl64.dll`.
- Downloads of large text files named `rockyou.txt`, or Git clones of popular wordlist repos.
- Package installs, for example `apt install john hashcat`, detected by EDR package telemetry.
- Tool updates and driver fetches for GPU runtimes.
- Repeated reads of files such as wordlists or encrypted files would need analysis.
- Capture triage artefacts such as process list, process memory dump, `nvidia-smi` sample output, open files, and the encrypted file.
- Preserve the working directory, wordlists, hash files, and shell history.
- Review which files were decrypted. Search for follow‑on access, lateral movement or exfiltration.

  


**Detections**

Below are some examples of detection rules and hunting queries we can put to use across various environments.

_Sysmon_:

```EventID=1
(ProcessName="C:\Program Files\john\john.exe" OR
 ProcessName="C:\Tools\hashcat\hashcat.exe" OR
 CommandLine="*pdf2john.pl*" OR
 CommandLine="*zip2john*")
```

_Linux audit rules, temporary for an investigation:_

```bash
auditctl -w /usr/share/wordlists/rockyou.txt -p r -k wordlists_read
auditctl -a always,exit -F arch=b64 -S execve -F exe=/usr/bin/john -k crack_exec
auditctl -a always,exit -F arch=b64 -S execve -F exe=/usr/bin/hashcat -k crack_exec
```

_Sigma style rule, Windows process create for cracking tools:_

```yaml
title: Password Cracking Tools Execution
id: 9f2f4d3e-4c16-4b0a-bb3a-7b1c6c001234
status: experimental
logsource:
  product: windows
  category: process_creation
detection:
  selection_name:
    Image|endswith:
      - '\john.exe'
      - '\hashcat.exe'
      - '\fcrackzip.exe'
      - '\pdfcrack.exe'
      - '\7z.exe'
      - '\qpdf.exe'
  selection_cmd:
    CommandLine|contains:
      - '--wordlist'
      - 'rockyou.txt'
      - 'zip2john'
      - 'pdf2john'
      - '--mask'
      - ' -a 3'
  condition: selection_name or selection_cmd
level: medium
```