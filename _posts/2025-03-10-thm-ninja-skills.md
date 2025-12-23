---
layout: post
title: "TryHackMe: Ninja Skills"
date: 2025-03-10 17:33:44 -0300
categories: TryHackMe CTF
tags: shell script
author: yaza
# description: My first CTF, Basic Pentesting was a straightforward room...
media_subpath:
image:
  path: /debd97fb93bffb8eb23d0687aa382bb0.png
---

Heyo!
Practise your Linux skills and complete the challenges.

[Room Link](https://tryhackme.com/room/ninjaskills)

Bash Script:

```bash
#!/bin/bash
files=("8V2L" "bny0" "c4ZX" "D8B3" "FHl1" "oiMO" "PFbD" "rmfX" "SRSq" "uqyw" "v2Vb" "X1Uy")
for file in "${files[@]}"; do
	echo "Searching for...$file"
	find / -name "$file" -type f 2>/dev/null
done
```

Tried to send via SCP but didnt work, so i sent by creating a python server on my machine and using wget on the target:
My machine:

```shell
root@ip-10-10-31-240:~/Downloads# python3 -m http.server 8000
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
10.10.20.180 - - [12/Mar/2025 18:09:33] code 404, message File not found
10.10.20.180 - - [12/Mar/2025 18:09:33] "GET /percorrerdir.sh HTTP/1.1" 404 -
10.10.20.180 - - [12/Mar/2025 18:11:05] code 404, message File not found
10.10.20.180 - - [12/Mar/2025 18:11:05] "GET /percorredir.shwget HTTP/1.1" 404 -
10.10.20.180 - - [12/Mar/2025 18:11:05] code 404, message File not found
10.10.20.180 - - [12/Mar/2025 18:11:05] "GET /percorredir.shwget HTTP/1.1" 404 -
10.10.20.180 - - [12/Mar/2025 18:11:05] code 404, message File not found
10.10.20.180 - - [12/Mar/2025 18:11:05] "GET /percorredir.shwget HTTP/1.1" 404 -
10.10.20.180 - - [12/Mar/2025 18:11:05] "GET /percorredir.sh HTTP/1.1" 200 -
```

Target:

```shell
[new-user@ip-10-10-20-180 ~]$ wget http://10.10.31.240:8000/percorredir.sh
```

```shell
chmod +x percorredir.sh
```

Got the paths:

```shell
[new-user@ip-10-10-20-180 ~]$ ./percorredir.sh
Searching for...8V2L
/etc/8V2L
Searching for...bny0
Searching for...c4ZX
/mnt/c4ZX
Searching for...D8B3
/mnt/D8B3
Searching for...FHl1
/var/FHl1
Searching for...oiMO
/opt/oiMO
Searching for...PFbD
/opt/PFbD
Searching for...rmfX
/media/rmfX
Searching for...SRSq
/etc/ssh/SRSq
Searching for...uqyw
/var/log/uqyw
Searching for...v2Vb
/home/v2Vb
Searching for...X1Uy
/X1Uy
```

`wc -l` word count -l (to count the lines)

... (didn't finish writing the rest of the post but completed the room)
