# CyberHeroes Write-up | 报告
<details>
  <summary>Click to view in Chinese (点击查看中文版)</summary>

---



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
