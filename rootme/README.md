# RootMe Write-up | 报告
<details>
  <summary>Click to view in Chinese (点击查看中文版)</summary>

---



  </details>

---

This is my write-up for the [RootMe](https://tryhackme.com/room/rrootme) room. 

# Overview

This room is about basic reconnaissance and exploitation. It contains simple tasks first, but at the exploitation phase it gets a little more difficult, we need to get reverse shell and elevate our priviliges. 

# Reconnaissance

We got four questions here:

1. **"Scan the machine, how many ports are open?"**;

2. **"What version of Apache is running?"**;

3. **"What service is running on port 22?"**;

**"Find directories on the web server using the GoBuster tool."**; (not necessarily GoBuster)

4. **"What is the hidden directory?"**.

After getting machine IP, we can scan it to search for open ports using Nmap:

Nmap (Network Mapper) is a free, open-source network scanning tool used for host discovery, port scanning, service detection, operating system fingerprinting, and security auditing.

```
nmap -sC -sV machine_ip
```
Make sure to use -sV command for service version detection, because in one of the tasks we have to write the version of the Apache running.

We got two open ports: 22 (ssh) and 80 (http). Apache version is 2.4.41 . Now let's find the hidden directory using ffuf tool, not GoBuster.

 > ffuf is generally considered better than Gobuster for complex web fuzzing due to its flexibility and advanced filtering capabilities.  While both tools are fast and written in Go, ffuf utilizes a FUZZ keyword that can be placed anywhere in an HTTP request (URL, headers, body, parameters), whereas Gobuster is primarily designed for appending paths to URLs.

```
ffuf -u http://machine_ip/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

And I found the "panel" directory.

## Summary:

**..how many ports are open?"** - 2

**"What version of Apache is running?"** - 2.4.41

**"What service is running on port 22?"** - ssh

**"What is the hidden directory?"** - `/panel/`

# Exploitation

**"Find a form to upload and get a reverse shell, and find the flag."**

We found hidden directory `/panel/`, lets check it: `http://machine_ip/panel`

Here we can upload a file, so now we need to find a form to get a reverse shell. 

I browsed internet for "reverse shell php" and found this form from [pentestmonkey](https://github.com/pentestmonkey): https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php

Download the "php-reverse-shell.php" file and take a look at its code.

We have two fields that we need to change: $ip & $port

```
set_time_limit (0);
$VERSION = "1.0";
$ip = '127.0.0.1';  // CHANGE THIS
$port = 1234;       // CHANGE THIS
$chunk_size = 1400;
$write_a = null;
```

# Solution

## Lessons Learned
