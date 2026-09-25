# Neighbour Write-up | 报告
<details>
  <summary>Click to view in Chinese (点击查看中文版)</summary>

---



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

Here, below the "Login" button they say that we can use guest account, with instruction "(Ctrl + U)". After pressing this hotkeys, we see page's source code.

Almost at the end of a code there's a comment:
```
</div>
            <p>Don't have an account? Use the guest account! (<code>Ctrl+U</code>)</p>
            <!-- use guest:guest credentials until registration is fixed. "admin" user account is off limits!!!!! -->
        </form>
```

The comment is ```<!-- use guest:guest credentials until registration is fixed. "admin" user account is off limits!!!!! -->```, so I used "guest" as a username and password in login form. "admin" didn't work.

And we're in.

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

And pressed Enter.



I got redirected to admin's account page, revealing the flag.

## Lessons Learned

This room is good to easily understand how does IDOR vulnerability work on practice. This challenge reminds you to check source code of the web page and page's web address properly.
