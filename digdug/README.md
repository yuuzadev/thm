# Dig Dug Write-up | 报告

<details>
  <summary>Click to view in Chinese (点击查看中文版)</summary>

---

这是我对 [Dig Dug](https://tryhackme.com/room/digdug) 房间的write-up。

# 概述

目标是在DNS记录中找到flag，但题目说明我们必须处理 `givemetheflag.com` 这个域名，因为"它只响应针对 `givemetheflag.com` 域名的特殊类型请求"。另外，"原来这台 MACHINE_IP 机器也是一台DNS服务器！"

# 侦察

获取到 MACHINE_IP 后，我通过 `dig` 命令搜索了DNS记录。

dig 命令是一个灵活的网络管理工具，用于查询域名系统（DNS）服务器，以获取域名记录的详细信息。

```
dig MACHINE_IP
```
<img width="1138" height="370" alt="image" src="https://github.com/user-attachments/assets/85662e34-27c4-433d-b943-fb3b9721bf7a" />

DNS 中有 4 个数据部分：

| 部分 | 用途 |
|----------|----------|
| QUESTION（问题）   | 你所查询的内容（只是回显你的查询）   |
| ANSWER（回答）   | A、MX、TXT、CNAME 等记录  |
| AUTHORITY（权威）   | 该区域的 SOA / NS 记录   |
| ADDITIONAL（附加）   | 粘合记录（名称服务器的 IP 地址）   |

如你所见，我们得到了一些列出的名称服务器：a.root-servers.net 和 nstld.verisign-grs.com，这是 AUTHORITY 记录，忽略它们。我们需要关注的是 ANSWER 记录。

如你所见，我们得到了一些域名：`a.root-servers.net` 和 `nstld.verisign-grs.com`，但我们不需要关注这些，因为提示是关于DNS记录的。

之后我决定扫描 MACHINE_IP 的开放端口，但这是一个错误，纯粹是浪费时间。我会在解题部分之后写一些建议。

# 解决方案

过了一会儿，我明白了我们需要将 MACHINE_IP 与域名 `givemetheflag.com` 结合使用 `dig`：

```
dig @MACHINE_IP givemetheflag.com
```
这个命令让我们可以看到特定DNS服务器关于该域名所看到的记录。

<img width="806" height="351" alt="image" src="https://github.com/user-attachments/assets/a281f651-ef5e-4737-9298-ae60ce24a7d2" />

而TXT记录揭示了flag。

## 经验教训

在我解这个房间的时候，我过度挖掘了。我用Nmap扫描了开放端口，然后尝试访问SSH……我忘记了房间的核心思想——DNS枚举。请始终在脑海中牢记房间的主题，始终只记住你真正需要的信息，不要去尝试寻找你根本不需要的东西。不要犯我的错误。这样会节省你的时间和精力。

  </details>

---

This is my write-up for the [Dig Dug](https://tryhackme.com/room/digdug) room. 

# Overview

The goal is to find the flag in DNS (Domain Name System) records, but it says that we have to work with `givemetheflag.com` domain, because "this only responds to a special type of request for a `givemetheflag.com` domain". Also, "turns out, this MACHINE_IP machine is also a DNS server!".

# Reconnaissance

After getting MACHINE_IP, I searched for DNS records via `dig` command.

The dig command is a flexible network administration tool used for querying Domain Name System (DNS) servers to retrieve detailed information about domain records.

```
dig MACHINE_IP
```
<img width="1138" height="370" alt="image" src="https://github.com/user-attachments/assets/7beb3598-d8fa-465d-a807-ce60febf484a" />

In DNS there's 4 sections of data:

| Section | Purpose |
|----------|----------|
| QUESTION   | What you asked (just echoes your query)   |
| ANSWER   | A, MX, TXT, CNAME, etc.  |
| AUTHORITY   | SOA / NS records for the zone   |
| ADDITIONAL   | Glue records (IPs for name servers)   |

As you can see we got some name servers listed: `a.root-servers.net` & `nstld.verisign-grs.com`, it is an AUTHORITY records, ignore them. We need to focus on ANSWER records.

After that I decided to look for open ports on MACHINE_IP, but it was a mistake and just a waste of time. I'll write some advices down here after the solving part.

# Solution

A little later I understood that we need to combine MACHINE_IP with the domain `givemetheflag.com` using `dig`:

```
dig @MACHINE_IP givemetheflag.com
```
This command lets us see what records specific DNS server sees about the domain.

<img width="806" height="351" alt="image" src="https://github.com/user-attachments/assets/b1f57a3d-52d4-4216-b087-616f6056cd45" />

And the TXT-record reveals the flag.

## Lessons Learned

When I was solving the room, I was digging too much. I searched for open ports using Nmap, then I tried to access to SSH... I forgot about the main idea of the room, DNS enumeration. Please, always keep in your head the concept of the room, always remember only the info you really need, do not try to find something that you don't even need. Don't make my mistakes. It will save your time and energy.
