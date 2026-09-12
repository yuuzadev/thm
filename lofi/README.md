# Lo-Fi Write-up | 报告

<details>
  <summary>Click to view in Chinese (点击查看中文版)</summary>

---
  
这是我对 [Lo-Fi](https://tryhackme.com/room/lofi) 房间的write-up。

# 概述

这是一个专注于文件系统遍历的CTF挑战题。

# 侦察

获取到机器的IP地址后，我们可以访问网页 `http://MACHINE_IP`：

<img width="1919" height="828" alt="image" src="https://github.com/user-attachments/assets/05896a24-ec88-4eae-87cc-740a275aa870" />

这个房间的主题是文件系统，所以我使用FFuF工具进行了目录枚举：

FFUF（Fuzz Faster U Fool）是一款开源的高性能 Web 模糊测试工具，用于发现隐藏的 Web 内容，例如目录、文件、参数和虚拟主机。

```
ffuf -u http://MACHINE_IP/FUZZ -w /usr/share/wordlists/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
```
（如果你没有这个词表，可以通过运行 `sudo apt install seclists` 来下载）

<img width="980" height="606" alt="Untitled" src="https://github.com/user-attachments/assets/76116272-0ecc-4931-a810-98530d517488" />

枚举之后，我们发现了 `server-status` 目录，但没有权限访问它。

<img width="860" height="275" alt="image" src="https://github.com/user-attachments/assets/ba222dda-69b8-4a34-b390-6749d31958c1" />

接着我查看了网页源码，找到了一些带有文件的目录，但并没有找到flag。

<img width="1919" height="808" alt="image" src="https://github.com/user-attachments/assets/c3da96de-ec2a-4c29-87b0-da690d9d0290" />

网页上有一个搜索框。如果你输入内容并点击“Go!”按钮，它会改变页面的URL并执行搜索。

<img width="1484" height="368" alt="Untitled" src="https://github.com/user-attachments/assets/3828a67a-ed0c-4cfa-892d-af3fcc4d66da" />

# 解决方案

我们可以利用这一点。就像在终端里操作一样：通过使用多个 `../` 命令，我们尝试进入文件系统的根目录，并访问 `/etc/passwd` 目录。另外，别忘了把URL中的 `?search=` 改成 `?page=`，这样才能看到结果。

<img width="1484" height="616" alt="Untitled" src="https://github.com/user-attachments/assets/a3eb6a14-d70a-4b1c-82ff-42f3a437a314" />

结果证明我们可以通过URL注入来访问数据。接下来让我们检查 `flag.txt`。

页面最终显示了flag。

## 经验教训
在这个房间里，我学到了如何：

* 使用URL注入来访问数据。

  </details>

---

This is my write-up for the [Lo-Fi](https://tryhackme.com/room/lofi) room. 

# Overview

This is CTF challenge focused on filesystem traversal.

# Reconnaissance

After getting an IP address of the machine, we can go to the web page `http://MACHINE_IP`:

<img width="1919" height="828" alt="image" src="https://github.com/user-attachments/assets/05896a24-ec88-4eae-87cc-740a275aa870" />

This room is about filesystem, so I enumerated directories using FFuF tool:

FFUF (Fuzz Faster U Fool) is an open-source, high-performance web fuzzing tool designed to discover hidden web content such as directories, files, parameters, and virtual hosts.

```
ffuf -u http://MACHINE_IP/FUZZ -w /usr/share/wordlists/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
```
(if you don't have this wordlist, download them by running `sudo apt install seclists`)

<img width="980" height="606" alt="Untitled" src="https://github.com/user-attachments/assets/76116272-0ecc-4931-a810-98530d517488" />

After enumeration we found the `server-status` directory, but we are not permitted to access it.

<img width="860" height="275" alt="image" src="https://github.com/user-attachments/assets/ba222dda-69b8-4a34-b390-6749d31958c1" />

Then I checked web source and found directories with files, but there was no flag.

<img width="1919" height="808" alt="image" src="https://github.com/user-attachments/assets/c3da96de-ec2a-4c29-87b0-da690d9d0290" />

The web page contains a search bar. If you type something and press "Go!" button, it will change the URL of the page and execute the search.

<img width="1484" height="368" alt="Untitled" src="https://github.com/user-attachments/assets/3828a67a-ed0c-4cfa-892d-af3fcc4d66da" />

# Solution

We can use it. Think just like in your terminal: we're going to root of the filesystem by using several `../` commands and going to `/etc/passwd` directory. Also don't forget to change `?search=` in URL to `?page=`, so we can see it.

<img width="1484" height="616" alt="Untitled" src="https://github.com/user-attachments/assets/a3eb6a14-d70a-4b1c-82ff-42f3a437a314" />

The result proves that we can access data by URL injection. Let's check for `flag.txt`.

The page reveals the flag.

## Lessons Learned

In this room, I learned how to:
* Use URL injection to access data.
