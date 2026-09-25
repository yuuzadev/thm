# Neighbour Write-up | 报告
<details>
  <summary>Click to view in Chinese (点击查看中文版)</summary>

---

这是我对 [Neighbour](https://tryhackme.com/room/neighbour) 房间的write-up。

# 概述

这个房间是关于 IDOR（不安全的直接对象引用）的——我们需要在邻居的已登录页面上找到flag。

# 侦察

获取到机器IP后，我按照挑战描述打开了"云服务"。

```
http://machine_ip
```

我们被重定向到 ```http://machine_ip/login.php``` 网页，在那里我们可以看到登录表单。既然这是关于 IDOR 的，让我们专注于它。

> 不安全的直接对象引用（IDOR）是一种访问控制漏洞，当应用程序使用用户输入直接访问内部对象而不检查用户是否有权限时就会发生。
> 该应用在 URL 或请求参数中暴露数据库 ID、文件名或账号。攻击者将链接中的 ID 号改为另一个值（比如把 id=101 改成 id=102）来查看或修改属于其他用户的数据。

<img width="961" height="434" alt="image" src="https://github.com/user-attachments/assets/dcc9e418-784f-4996-8210-ec75d341910a" />

这里，在"Login"按钮下方他们说我们可以使用访客账户，并附有说明"(Ctrl + U)"。按下这个快捷键后，我们看到了页面的源代码。

<img width="919" height="587" alt="image" src="https://github.com/user-attachments/assets/21e2baf5-1119-486a-bd72-9dd94c64800e" />

在代码几乎最末尾的地方有一条注释：

```
</div>
            <p>Don't have an account? Use the guest account! (<code>Ctrl+U</code>)</p>
            <!-- use guest:guest credentials until registration is fixed. "admin" user account is off limits!!!!! -->
        </form>
```

注释是 ```<!-- use guest:guest credentials until registration is fixed. "admin" user account is off limits!!!!! -->```，所以我在登录表单中使用了"guest"作为用户名和密码。"admin"没有成功。

然后我们进去了。

<img width="1483" height="252" alt="image" src="https://github.com/user-attachments/assets/9ad1c9fc-d0bf-4304-aeed-fdc6c0bc790b" />

# 漏洞利用

这是一个 IDOR 挑战，所以看看顶部的搜索栏：

```
http://machine_ip/profile.php?user=guest
```

在这里你可以看到一个字段，显示我们当前处于访客账户：```profile.php?user=guest```

我直接把它改成了 `admin`，所以它看起来像这样：

```
http://machine_ip/profile.php?user=admin
```

然后按 Enter。 我被重定向到了 admin 的账户页面，揭示了flag。

<img width="1845" height="241" alt="image" src="https://github.com/user-attachments/assets/518be3f9-dda9-4c46-948a-eeea9b8ec6b5" />

## 经验教训

这个房间很适合在实践中轻松理解 IDOR 漏洞是如何工作的。这个挑战提醒你要检查网页的源代码和页面的网址。


</details>

---

This is my write-up for the [Neighbour](https://tryhackme.com/room/neighbour) room. 

# Overview

This room is about IDOR (Insecure Direct Object References) - we need to find the flag on our neighbor's logged in page.

# Reconnaissance

After getting machine IP, I opened the "cloud service", as the challenge description says.

```
http://machine_ip
```

We're being redirected to ```http://machine_ip/login.php``` web page, where we can see login form. Since it's about IDOR, let's focus on that.

> Insecure Direct Object References (IDOR) are an access control vulnerability that happens when an application uses user input to access internal objects directly without checking if the user has permission.
The app exposes a database ID, file name, or account number in the URL or request parameters. An attacker changes the ID number in the link to another value (like switching id=101 to id=102) to view or change data belonging to other users.

<img width="961" height="434" alt="image" src="https://github.com/user-attachments/assets/dcc9e418-784f-4996-8210-ec75d341910a" />

Here, below the "Login" button they say that we can use guest account, with instruction "(Ctrl + U)". After pressing this hotkeys, we see page's source code.

<img width="919" height="587" alt="image" src="https://github.com/user-attachments/assets/21e2baf5-1119-486a-bd72-9dd94c64800e" />

Almost at the end of a code there's a comment:
```
</div>
            <p>Don't have an account? Use the guest account! (<code>Ctrl+U</code>)</p>
            <!-- use guest:guest credentials until registration is fixed. "admin" user account is off limits!!!!! -->
        </form>
```

The comment is ```<!-- use guest:guest credentials until registration is fixed. "admin" user account is off limits!!!!! -->```, so I used "guest" as a username and password in login form. "admin" didn't work.

And we're in.

<img width="1483" height="252" alt="image" src="https://github.com/user-attachments/assets/9ad1c9fc-d0bf-4304-aeed-fdc6c0bc790b" />

# Exploitation

This is an IDOR challenge, so look at your search bar at the top:

```
http://machine_ip/profile.php?user=guest
```

Here you can see a field which says that we're on guest account right now: ```profile.php?user=guest```

I just changed it to `admin`, so it looked like this: 

```
http://machine_ip/profile.php?user=admin
```

And pressed Enter. I got redirected to admin's account page, revealing the flag.

<img width="1845" height="241" alt="image" src="https://github.com/user-attachments/assets/518be3f9-dda9-4c46-948a-eeea9b8ec6b5" />

## Lessons Learned

This room is good to easily understand how does IDOR vulnerability work on practice. This challenge reminds you to check source code of the web page and page's web address properly.
