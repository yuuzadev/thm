# TakeOver Write-up | 报告

<details>
  <summary>Click to view in Chinese (点击查看中文版)</summary>
  
## 中文:
> 我在这个房间使用了 TryHackMe 的 AttackBox。

这是我的 [TakeOver](https://tryhackme.com/room/takeover) 房间的 Write-up。这个挑战主要围绕子域名枚举。

在开始之前，你需要将 MACHINE_IP 添加到 `/etc/hosts` 文件中，指向 futurevera.thm（该房间的主机名）。进入 `/etc/hosts`：
```
sudo nano /etc/hosts
```
并将 MACHINE_IP 与 `futurevera.thm` 一起添加到列表中：

<img width="724" height="253" alt="изображение" src="https://github.com/user-attachments/assets/01f4a950-c271-4be6-9508-dd2b0e87c973" />

然后保存文件。


将 futurevera 添加到 `/etc/hosts` 后，我们可以通过访问 ```https://futurevera.thm``` 来打开网站：

<img width="1226" height="879" alt="изображение" src="https://github.com/user-attachments/assets/7d40e3d8-ec3e-42c1-835d-2ea3c79e1083" />

我们会看到一个警告页面，提示存在潜在的安全风险。暂时忽略它，点击"**高级...**" > "**接受风险并继续**"。

这是 FutureVera 网站。

<img width="1240" height="882" alt="изображение" src="https://github.com/user-attachments/assets/a2bdca6b-7bcf-439e-8def-85784e33df22" />

这个挑战围绕子域名枚举，所以我们需要在这方面下功夫。我使用 **ffuf** 工具来做这件事。

```
ffuf -u https://10.146.166.183 -H "host: FUZZ.futurevera.thm" -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt
```

运行这个命令后，我得到了很多输出，但这些输出并没有太大用处。

<img width="695" height="648" alt="изображение" src="https://github.com/user-attachments/assets/ea410335-2108-41eb-a42e-b6888559c62f" />

你可以看到所有结果的大小都是 4605，所以我们可以将它们过滤掉。在命令末尾添加 `-fs 4605` 标志（意为"过滤大小"）并重试。

<img width="1102" height="576" alt="изображение" src="https://github.com/user-attachments/assets/bce390df-7a82-4f8f-9852-34df6a34963d" />

我们得到了两个子域名：`support` 和 `blog`。将它们添加到 `/etc/hosts`：

<img width="638" height="283" alt="изображение" src="https://github.com/user-attachments/assets/db703ce6-208e-43da-9a5f-162901d90e18" />

在 ```https://blog.futurevera.thm``` 没有什么有趣的东西。

<img width="1231" height="876" alt="изображение" src="https://github.com/user-attachments/assets/bd8d7668-4576-42af-b502-bf8d5ce67c96" />

在 ```https://support.futurevera.thm``` 也没有什么。

<img width="1235" height="873" alt="изображение" src="https://github.com/user-attachments/assets/d8e3b245-61eb-4c76-8990-ac9440a43ac0" />

让我们回到 ```https://support.futurevera.thm``` 的警告页面：

<img width="882" height="795" alt="изображение" src="https://github.com/user-attachments/assets/158d27cf-2dc3-49d6-b4cb-0757193b7c2a" />

点击"查看证书"按钮，然后找到 DNS 名称：

<img width="800" height="718" alt="изображение" src="https://github.com/user-attachments/assets/d7f070f3-2436-4138-b084-b66717cc973c" />

复制它并再次添加到 `/etc/hosts`：

<img width="651" height="152" alt="изображение" src="https://github.com/user-attachments/assets/a16fd5c9-3eb2-4765-a9d0-eac2c1454437" />

之后我们可以查看 ```https://secrethelpdesk934752.support.futurevera.thm```：

<img width="1233" height="871" alt="изображение" src="https://github.com/user-attachments/assets/073dd83f-1614-4f1b-bdfc-463c710b163a" />

这里什么都没有。我们应该尝试使用 http 协议访问，而不是 https，然后我们会看到 ```http://secrethelpdesk934752.support.futurevera.thm``` 页面：

<img width="1219" height="700" alt="изображение" src="https://github.com/user-attachments/assets/9bd20e22-4c92-49cc-a9a6-ad17c4cc4780" />

我们得到了一个错误页面，显示了 flag。

## 总结

在这个房间中，我学会了如何：
* 使用 **ffuf** 进行子域名枚举，以及如何使用 `-fs` 标志按大小过滤结果。
* 检查 **SSL 证书** 来查找可能不会出现在字典中的隐藏 DNS 名称。
* 检查网站的 **https** 和 **http** 两个版本，因为它们可能会显示不同的内容。

</details>

> I used TryHackMe AttackBox for this room.

---

This is my write-up for the [TakeOver](https://tryhackme.com/room/takeover) room. This challenge revolves around subdomain enumeration.

Before you start, you should add the MACHINE_IP in `/etc/hosts` for futurevera (room's site). Go to `/etc/hosts`:
```
sudo nano /etc/hosts
```
And add MACHINE_IP with `futurevera.thm` to the list:

<img width="724" height="253" alt="изображение" src="https://github.com/user-attachments/assets/01f4a950-c271-4be6-9508-dd2b0e87c973" />

Then save the file.


After adding futurevera to `/etc/hosts` we can open the site by going ```https://futurevera.thm```:

<img width="1226" height="879" alt="изображение" src="https://github.com/user-attachments/assets/7d40e3d8-ec3e-42c1-835d-2ea3c79e1083" />

We will get the warning page, that says about potential security risk. Ignore it for now and continue by pressing "**Advanced...**" > "**Accept the Risk and Continue**".

Here's the FutureVera site.

<img width="1240" height="882" alt="изображение" src="https://github.com/user-attachments/assets/a2bdca6b-7bcf-439e-8def-85784e33df22" />

This challenge revolves around subdomain enumeration, so we have to work on that. I'm using **ffuf** tool for this.

```
ffuf -u https://10.146.166.183 -H "host: FUZZ.futurevera.thm" -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt
```

By running this command I got a lot of output, which is not really useful.

<img width="695" height="648" alt="изображение" src="https://github.com/user-attachments/assets/ea410335-2108-41eb-a42e-b6888559c62f" />

You can see that the size of all of them is 4605, so we can filter them. Add `-fs 4605` flag (which stands for "Filter Size") to the end of the command and try again.

<img width="1102" height="576" alt="изображение" src="https://github.com/user-attachments/assets/bce390df-7a82-4f8f-9852-34df6a34963d" />

We got two subdomains: `support` and `blog`. Add them to `/etc/hosts`:

<img width="638" height="283" alt="изображение" src="https://github.com/user-attachments/assets/db703ce6-208e-43da-9a5f-162901d90e18" />

At the ```https://blog.futurevera.thm``` nothing interesting.

<img width="1231" height="876" alt="изображение" src="https://github.com/user-attachments/assets/bd8d7668-4576-42af-b502-bf8d5ce67c96" />

At the ```https://support.futurevera.thm``` nothing too.

<img width="1235" height="873" alt="изображение" src="https://github.com/user-attachments/assets/d8e3b245-61eb-4c76-8990-ac9440a43ac0" />

Let's go back to the warning page at ```https://support.futurevera.thm```:

<img width="882" height="795" alt="изображение" src="https://github.com/user-attachments/assets/158d27cf-2dc3-49d6-b4cb-0757193b7c2a" />

Press the "View Certificate" button, then find the DNS Name:

<img width="800" height="718" alt="изображение" src="https://github.com/user-attachments/assets/d7f070f3-2436-4138-b084-b66717cc973c" />

Copy it and add to `/etc/hosts` again:

<img width="651" height="152" alt="изображение" src="https://github.com/user-attachments/assets/a16fd5c9-3eb2-4765-a9d0-eac2c1454437" />

After that we can check ```https://secrethelpdesk934752.support.futurevera.thm```:

<img width="1233" height="871" alt="изображение" src="https://github.com/user-attachments/assets/073dd83f-1614-4f1b-bdfc-463c710b163a" />

Nothing here. We should remove the 's' letter in `https`, and we'll see ```http://secrethelpdesk934752.support.futurevera.thm``` page:

<img width="1219" height="700" alt="изображение" src="https://github.com/user-attachments/assets/9bd20e22-4c92-49cc-a9a6-ad17c4cc4780" />

We're getting an error page, revealing the flag.

## Conclusion

In this room, I learned how to:
* Use **ffuf** for subdomain enumeration and how to filter results by size, using `-fs` flag.
* Check **SSL Сertificates** to find hidden DNS names that might not show up in wordlists.
* Check both **https** and **http** versions of a site, as they might display different content.
