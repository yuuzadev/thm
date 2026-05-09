# TryHackMe MD2PDF Write-up | 报告

> I used the TryHackMе AttackBox for this room   
> Also available in Chinese translation |  还有中文翻译    

## Overview

This is my write-up for the [MD2PDF room](https://tryhackme.com/room/md2pdf). The goal was to exploit a web application that converts text to PDF to access a restricted page with flag.  
   
I divided it into three steps:  

1. Reconnaissance (Scanning & Enumeration)
2. Exploitation (SSRF via PDF Generation)
3. Conclusion 

### 1. Reconnaissance 

### Scanning

After getting target IP-address, we can scan it to search for open ports using Nmap.  

```bash
nmap (target-ip)
```

<img width="1446" height="519" alt="изображение" src="https://github.com/user-attachments/assets/0c83633f-3fd2-4926-a841-8caedcd42ebf" />

As you can see, we got 3 open ports: 80 (web), 22 (ssh), 5000. Now we can check what services are running on these ports.  
Type: ```http://(target-ip):80``` in your browser to go to the site with 80 port.  

<img width="485" height="73" alt="изображение" src="https://github.com/user-attachments/assets/942ae915-6fe3-4222-9871-67c70735bfd0" />

After going to this site, we see a form that gets a text, then after you press the "Convert to PDF" button, it redirects you to the site with PDF text page.  

<img width="1218" height="530" alt="изображение" src="https://github.com/user-attachments/assets/1cf41027-478c-45be-8f8e-39a7bc9025d7" />
<img width="1113" height="602" alt="изображение" src="https://github.com/user-attachments/assets/92c504df-75d5-4a0c-a186-7743aa78095d" />

Now let's check 5000 port: ```http://(target-ip):5000```  

<img width="500" height="242" alt="изображение" src="https://github.com/user-attachments/assets/c6da6576-8d43-429e-a40f-ac6d6edc60b5" />

This page is similiar to the previous one, but the "Convert to PDF" button was unresponsive.  
What about 22? After ```http://(target-ip):22``` we get an error, so we don't need it. I forgot to take a screenshoot.  

### Enumeration 

I used Gobuster to find hidden directories on the web server.  

I ran the following command:  

```bash
gobuster dir -u http://(target-ip) -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
```

After the command we have an output:  

<img width="724" height="411" alt="изображение" src="https://github.com/user-attachments/assets/4bee595f-26b6-43a8-a60e-a0356ed99aea" />

It says that there's 2 hidden drectories, "/admin" and "/convert". Now we can try to go to these pages. ```http://(target-ip)/admin```  

<img width="480" height="188" alt="изображение" src="https://github.com/user-attachments/assets/74e785b4-6a94-49c5-bc10-8988d908f980" />

After going to the /admin page, it says that it is forbidden and can be seen only by localhost:5000 (port 5000). This is an important information. What about /convert?
```http://(target-ip)/convert```  

<img width="504" height="182" alt="изображение" src="https://github.com/user-attachments/assets/eaeae537-20cd-4065-8676-5e20935d147a" />

We didn't get anything. So let's focus on that /admin page.  

### 2. Exploitation

After the reconnaissance we have:
* Which ports are open (80, 22, 5000) by using nmap
* Hidden directories (/admin, /convert) by using gobuster
* The info that the /admin page can be only seen by localhost:5000  

I injected an iframe pointing to the restricted page into the form:  
```<iframe src= "http://127.0.0.1:5000/admin"></iframe>```   

<img width="613" height="141" alt="изображение" src="https://github.com/user-attachments/assets/0438ea21-e191-4c42-81d6-fece099e52f9" />

Why "127.0.0.1"? Because we need to check the localhost, and "127.0.0.1" is always a local host's IP.  
The PDF displayed the contents of the localhost:5000/admin page, revealing the flag.  

### 3. Conclusion

* Vulnerability: The main vulnerability was an SSRF in the PDF generation, by user input
* Impact: This allowed an attacker to access internal services (like on port 5000) and retrieve information (the flag)
* Mitigation: Sanitize user input and implement a whitelist for allowed URL schemes.  

*Note:*  
What's SSRF? (Server-side request forgery) - it's a computer security vulnerability that enables an attacker to send requests from a vulnerable server to internal or external systems or the server itself.  

## 中文：  

> 我使用了 TryHackMe 的 AttackBox 来攻破这个房间。

## 概述
  
这是我为[MD2PDF房间](https://tryhackme.com/room/md2pdf)撰写的报告。目标是利用一个将文本转换为PDF的网络应用程序，从而访问包含旗帜的受限页面。  

我将其分为三个步骤：

1. 侦察（扫描与枚举）
2. 利用（通过PDF生成实现的SSRF）
3. 结论

### 1. 侦察

### 扫描

获取目标IP地址后，我们可以利用Nmap对其进行扫描以查找开放端口。

```bash
nmap (target-ip)
```

<img width="1446" height="519" alt="изображение" src="https://github.com/user-attachments/assets/0c83633f-3fd2-4926-a841-8caedcd42ebf" />

如您所见，我们有3个开放端口：80（Web）、22（SSH）、5000。现在我们可以检查这些端口上运行着哪些服务。  
在浏览器中输入：```http://(目标IP):80``` 即可访问80端口的网站。

<img width="485" height="73" alt="изображение" src="https://github.com/user-attachments/assets/942ae915-6fe3-4222-9871-67c70735bfd0" />

访问该网站后，我们会看到一个文本输入表单。点击“转换为PDF”按钮后，系统会将您重定向至包含PDF文本内容的页面。  

<img width="1218" height="530" alt="изображение" src="https://github.com/user-attachments/assets/1cf41027-478c-45be-8f8e-39a7bc9025d7" />
<img width="1113" height="602" alt="изображение" src="https://github.com/user-attachments/assets/92c504df-75d5-4a0c-a186-7743aa78095d" />

现在让我们检查5000端口：```http://(目标IP):5000```  

<img width="500" height="242" alt="изображение" src="https://github.com/user-attachments/assets/c6da6576-8d43-429e-a40f-ac6d6edc60b5" />

此页面与前一页类似，但“转换为PDF”按钮无法响应。  
22端口呢？在```http://(目标IP):22```之后出现错误，所以不需要它。我忘了截屏。

### 枚举

我使用Gobuster在Web服务器上查找隐藏目录。  

我执行了以下命令:  

```bash
gobuster dir -u http://(target-ip) -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
```

执行命令后，我们得到以下输出： 

<img width="724" height="411" alt="изображение" src="https://github.com/user-attachments/assets/4bee595f-26b6-43a8-a60e-a0356ed99aea" />

它显示存在两个隐藏目录：“/admin” 和 “/convert”。现在我们可以尝试访问这些页面。```http://(目标IP)/admin```  

<img width="480" height="188" alt="изображение" src="https://github.com/user-attachments/assets/74e785b4-6a94-49c5-bc10-8988d908f980" />
 
访问/admin页面后，提示该页面被禁止访问，仅限本地主机通过5000端口（localhost:5000）访问。这是重要信息。那么/convert页面呢？
```http://(目标IP)/convert```

<img width="504" height="182" alt="изображение" src="https://github.com/user-attachments/assets/eaeae537-20cd-4065-8676-5e20935d147a" />

我们什么都没得到。所以让我们把注意力集中在那个/admin页面上。 

### 2. 利用

侦察后我们获得：
* 通过nmap扫描发现开放端口（80、22、5000）
* 通过gobuster发现隐藏目录（/admin、/convert）
* 确认/admin页面仅可通过localhost:5000访问

我在表单中注入了一个指向受限页面的iframe:  
```<iframe src= “http://127.0.0.1:5000/admin”></iframe>```  

<img width="613" height="141" alt="изображение" src="https://github.com/user-attachments/assets/0438ea21-e191-4c42-81d6-fece099e52f9" />

为何使用“127.0.0.1”？因为我们需要检测本地主机，而“127.0.0.1”始终是本地主机的IP地址。  
该PDF文件显示了localhost:5000/admin页面的内容，从而揭示了密钥。

### 3. 结论

* 漏洞：主要漏洞是PDF生成过程中存在的基于用户输入的SSRF漏洞
* 影响：这使得攻击者能够访问内部服务（如5000端口）并获取信息（即标志）
* 修复建议：对用户输入进行严格过滤，并为允许的URL协议实施白名单机制。

*注:*  
什么是SSRF？（服务器端请求伪造）——这是一种计算机安全漏洞，攻击者可借此从存在漏洞的服务器向内部或外部系统乃至服务器自身发送请求。


<details>
  <summary>Click to view in Chinese (点击查看中文版)</summary>
  
  ## 概述
  这是我为 MD2PDF 撰写的报告...
</details>
