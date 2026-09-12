# CyberHeroes Write-up | 报告
<details>
  <summary>Click to view in Chinese (点击查看中文版)</summary>

---

这是我对 [CyberHeroes](https://tryhackme.com/room/cyberheroes) 房间的write-up。

# 概述
目标是访问 URL：http://machine_ip/ 并找到登录的方法。

# 侦察
获取到 MACHINE_IP 后，我打开了 http://machine_ip/，看到了 Cyber Heros 网站。

我阅读了下面的信息，但没有任何有用的内容。

然后我打开了左侧的"Login"选项卡。

我们有两个输入框："username"和"password"。点击下方的"Login"按钮后，我们看到一个窗口，上面写着"Incorrect Password, try again.. you got this hacker !"，仅此而已。

# 解决方案
我在登录选项卡中使用了 Ctrl + U 快捷键来打开源代码，在页面几乎最末尾的地方我们看到了这段脚本：

它直接告诉了我们如何通过登录页面，我们可以看到两个值：a（'uname'，即用户名）和 b（'pass'，显然是密码）。

```
      a = document.getElementById('uname')
      b = document.getElementById('pass')
      const RevereString = str => [...str].reverse().join('');
      if (a.value=="h3ck3rBoi" & b.value==RevereString("54321@terceSrepuS")) {
```
现在我们知道用户名是"h3ck3rBoi"，密码是"54321@terceSrepuS"，真的是这样吗？

我用它尝试登录，但它再次阻止了我，说密码不正确。然后我仔细看了页面源代码中的那段脚本。我们可以看到在密码前面有一个名为"RevereString"的函数，它的作用是将字符串反转。所以我尝试使用反转后的密码"SuperSecret@12345"。

它成功了，显示出了带有flag的窗口。

## 经验教训
永远要检查页面的源代码！你可以在注释中找到很多有用的信息、正在使用的服务版本等等。

  </details>

---

This is my write-up for the [CyberHeroes](https://tryhackme.com/room/cyberheroes) room. 

# Overview
The goal is to navigate to URL: http://machine_ip/ and find a way to log in.

# Reconnaissance
After getting MACHINE_IP, I opened http://machine_ip/ and saw the Cyber Heros website.

I read the information below, but there was nothing useful.

Then I opened "Login" tab on the left side.

We have two input fields: "username" and "password". After pressing "Login" button below, we're seeing a window which says "Incorrect Password, try again.. you got this hacker !", and that's it.

# Solution

I used Ctrl + U hotkey in login tab to open the source code, and almost at the end of the page we have this script:

It literally tells us how to pass the login page, we can see two values: a ('uname', a username) and b ('pass', obviously a password).

```
      a = document.getElementById('uname')
      b = document.getElementById('pass')
      const RevereString = str => [...str].reverse().join('');
      if (a.value=="h3ck3rBoi" & b.value==RevereString("54321@terceSrepuS")) { 
```

Now we know that the username is "h3ck3rBoi", and the password is "54321@terceSrepuS", or is it?

I used it to try to log in but it stopped me again, saying that the password is incorrect. Then I looked closely to that script in the source code of the page. We can see that before password there's a function "RevereString" which job is to reverse strings. So I tried to use reversed version of password, "SuperSecret@12345".

And it worked, revealing window with the flag. 

## Lessons Learned

Always check the source code of the page! You can find a lot of useful info in the comments, versions of the services that is being used and more. 
