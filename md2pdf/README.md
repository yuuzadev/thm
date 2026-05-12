# MD2PDF Write-up | 报告

<details>
  <summary>Click to view in Chinese (点击查看中文版)</summary>
  
## 中文:
> 我在这个房间使用了 TryHackMe 的 AttackBox。

---

这是我的 [MD2PDF](https://tryhackme.com/room/md2pdf) 房间的 Write-up。目标是利用一个将文本转换为 PDF 的 Web 应用程序来访问包含 flag 的页面。

我将其分为三个步骤：

1. 信息收集
2. 漏洞利用
3. 总结

## 1. 信息收集
### 端口扫描

获取目标 IP 地址后，我们可以使用 Nmap 扫描开放端口：

```
nmap 10.82.178.215
```

<img width="962" height="337" alt="изображение" src="https://github.com/user-attachments/assets/7c7da6c5-7fb9-4cbc-8b6c-1e64979c4f91" />


我们发现了 3 个开放端口：80、22、5000。现在我们可以检查这些端口上运行了什么服务。
在浏览器中访问：```http://10.82.178.215:80```。

我们看到一个接收文本的表单，点击"转换为 PDF"按钮后，会重定向到包含 PDF 文本页面的网站。

<img width="1218" height="530" alt="изображение" src="https://github.com/user-attachments/assets/1cf41027-478c-45be-8f8e-39a7bc9025d7" />
<img width="1113" height="602" alt="изображение" src="https://github.com/user-attachments/assets/92c504df-75d5-4a0c-a186-7743aa78095d" />

让我们查看 5000 端口：```http://10.82.178.215:5000```

<img width="500" height="242" alt="изображение" src="https://github.com/user-attachments/assets/c6da6576-8d43-429e-a40f-ac6d6edc60b5" />

这个页面与上一个类似，但"转换为 PDF"按钮没有反应。

### 目录枚举

我使用 Gobuster 来查找 Web 服务器上的隐藏目录：

```bash
gobuster dir -u http://10.82.178.215 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
```

运行命令后，我们得到如下输出：

<img width="960" height="539" alt="изображение" src="https://github.com/user-attachments/assets/70a09021-a657-4e52-b455-d9bfe19f22c2" />


它显示有 2 个隐藏目录，`/admin` 和 `/convert`。现在我们可以尝试访问这些页面：

```http://10.82.178.215/admin```

<img width="480" height="188" alt="изображение" src="https://github.com/user-attachments/assets/74e785b4-6a94-49c5-bc10-8988d908f980" />

进入 `/admin` 页面后，提示访问被禁止，只能由 localhost:5000（5000 端口）查看。这是一条重要信息。那么 `/convert` 呢？

```http://10.82.178.215/convert```

<img width="504" height="182" alt="изображение" src="https://github.com/user-attachments/assets/eaeae537-20cd-4065-8676-5e20935d147a" />

我们没得到任何东西。所以让我们专注于那个 `/admin` 页面。

## 2. 漏洞利用

信息收集之后，我们掌握了：
* 通过 nmap 知道哪些端口是开放的（80、22、5000）
* 通过 gobuster 发现了隐藏目录（`/admin`、`/convert`）
* 知道 `/admin` 页面只能由 localhost:5000 查看

我在表单中注入了一个指向受限页面的 iframe：
```
<iframe src= "http://127.0.0.1:5000/admin"></iframe>
```

<img width="613" height="141" alt="изображение" src="https://github.com/user-attachments/assets/0438ea21-e191-4c42-81d6-fece099e52f9" />

`127.0.0.1` 始终是本地主机的 IP，这就是我们使用它的原因。

生成的 PDF 显示了 `127.0.0.1:5000/admin` 页面的内容，从而泄露了 flag。

## 3. 总结

* 漏洞：主要漏洞是 PDF 生成过程中，通过用户输入导致的 SSRF
* 影响：这允许攻击者访问内部服务（如 5000 端口上的服务）并获取信息（flag）
* 缓解措施：对用户输入进行清理，并为允许的 URL 方案实施白名单。

> 注意：
什么是 SSRF？（服务器端请求伪造）—— 这是一种计算机安全漏洞，使攻击者能够从存在漏洞的服务器向内部或外部系统或服务器本身发送请求。
</details>

> I used TryHackMe AttackBox for this room.

---

This is my write-up for the [MD2PDF](https://tryhackme.com/room/md2pdf) room. The goal was to exploit a web application that converts text to PDF to access a page with flag.  
   
I divided it into three steps:  

1. Reconnaissance
2. Exploitation
3. Conclusion 

## 1. Reconnaissance 
### Scanning

After getting target IP-address, we can scan it to search for open ports using Nmap:

```
nmap 10.82.178.215
```

<img width="962" height="337" alt="изображение" src="https://github.com/user-attachments/assets/7c7da6c5-7fb9-4cbc-8b6c-1e64979c4f91" />


We got 3 open ports: 80, 22, 5000. Now we can check what services are running on these ports.  
Type: ```http://10.82.178.215:80``` in your browser to go to the site.

We see a form that gets a text, then after you press the "Convert to PDF" button, it redirects you to the site with PDF text page.  

<img width="1218" height="530" alt="изображение" src="https://github.com/user-attachments/assets/1cf41027-478c-45be-8f8e-39a7bc9025d7" />
<img width="1113" height="602" alt="изображение" src="https://github.com/user-attachments/assets/92c504df-75d5-4a0c-a186-7743aa78095d" />

Let's check 5000 port: ```http://10.82.178.215:5000```  

<img width="500" height="242" alt="изображение" src="https://github.com/user-attachments/assets/c6da6576-8d43-429e-a40f-ac6d6edc60b5" />

This page is similiar to the previous one, but the "Convert to PDF" button was unresponsive.  

### Enumeration 

I used Gobuster to find hidden directories on the web server:

```bash
gobuster dir -u http://10.82.178.215 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
```

After the command we have an output:  

<img width="960" height="539" alt="изображение" src="https://github.com/user-attachments/assets/70a09021-a657-4e52-b455-d9bfe19f22c2" />


It says that there's 2 hidden drectories, `/admin` and `/convert`. Now we can try to go to these pages: 

```http://10.82.178.215/admin```  

<img width="480" height="188" alt="изображение" src="https://github.com/user-attachments/assets/74e785b4-6a94-49c5-bc10-8988d908f980" />

After going to the `/admin` page, it says that it is forbidden and can be seen only by localhost:5000 (port 5000). This is an important information. What about ```/convert```?

```http://10.82.178.215/convert```  

<img width="504" height="182" alt="изображение" src="https://github.com/user-attachments/assets/eaeae537-20cd-4065-8676-5e20935d147a" />

We didn't get anything. So let's focus on that `/admin` page.  

## 2. Exploitation

After the reconnaissance we have:
* Which ports are open (80, 22, 5000) by using nmap
* Hidden directories (`/admin`, `/convert`) by using gobuster
* The info that the `/admin` page can be only seen by localhost:5000  

I injected an iframe pointing to the restricted page into the form:  
```
<iframe src= "http://127.0.0.1:5000/admin"></iframe>
```   

<img width="613" height="141" alt="изображение" src="https://github.com/user-attachments/assets/0438ea21-e191-4c42-81d6-fece099e52f9" />

`127.0.0.1` is always a local host's IP, this is why we using it.

The PDF displayed the contents of the ```127.0.0.1:5000/admin``` page, revealing the flag.  

## 3. Conclusion

* Vulnerability: The main vulnerability was an **SSRF** in the PDF generation, by user input
* Impact: This allowed an attacker to access internal services (like on port 5000) and retrieve information (the flag)
* Mitigation: Sanitize user input and implement a whitelist for allowed URL schemes.  

> Note:
What's SSRF? (Server-side request forgery) - it's a computer security vulnerability that enables an attacker to send requests from a vulnerable server to internal or external systems or the server itself.
