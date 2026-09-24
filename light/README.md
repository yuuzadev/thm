# Light Write-up | 报告
<details>
  <summary>Click to view in Chinese (点击查看中文版)</summary>

---



</details>

---

This is my write-up for the [Light](https://tryhackme.com/room/lightroom) room. 

# Overview

This room is about connecting to port 1337 through Netcat and find some info - admin's username, password and flag. We also got a username to start with: "..the application is running on port 1337. You can connect to it using `nc machine_ip 1337`.
You can use the username **smokey** in order to get started."

# Reconnaissance

We got three questions here:

1. **"What is the admin username?"**;

2. **"What is the password to the username mentioned in question 1?"**;

3. **"What is the flag?"**;

After getting machine IP, we can connect to it with Netcat:

> Netcat (often abbreviated as nc) is a versatile command-line utility designed to read and write data across network connections using TCP or UDP protocols. A netcat reverse shell allows a target system to connect back to an attacker's machine, bypassing inbound firewall restrictions by initiating an outbound connection.

```
nc machine_ip 1337
```

<img width="485" height="181" alt="image" src="https://github.com/user-attachments/assets/c44e0feb-ce50-4b03-bcc6-1d2d6376cefb" />

We see a line (or form) that reads our input and gives us an output. If you ever see something like this (for example in login pages in websites), you have to test if it's SQL.

> SQL stands for Structured Query Language, a standardized programming language used to communicate with and manipulate relational databases. It allows users to create, retrieve, update, and delete data stored in tables consisting of rows and columns.

To test it out you can just type: `'` (apostrophe) and press Enter.

<img width="584" height="85" alt="image" src="https://github.com/user-attachments/assets/a99a1956-e16a-4644-b6c2-b597ea4defef" />

And the error output "Error: unrecognized token: "''' LIMIT 30" " tells us that we can use an SQL injection here. But first, what is it and how does it work?

# Exploitation

SQL Injection is when an attacker types special characters into an input field (like a login form) to change the SQL query behind the scenes. If the app doesn't sanitize the input, something like `'` breaks the query and shows that the input is being read as SQL code, not just plain text. Once you confirm that, you can add things like `' OR 1=1 --` to bypass login, get data out of the database, or even modify it.

So if in login form I type "admin" in username field and "password123" in password field, the SQL query is going to look like this:

<img width="1307" height="59" alt="image" src="https://github.com/user-attachments/assets/dd6acb88-9ca6-4ed7-bbd0-bf763f38fcd0" />

Look at the query — `'admin'` and `'password123'` are wrapped in single quotes `'`. Those quotes tell the database that inside them is a value. But if you type `'` in the input, you close that quote early. Now everything after it is no longer inside the string, so the database reads it as actual SQL code. That's the whole trick. You "escape" the string and start writing your own query. 

Here's the most popular payload to bypass to login: `' OR '1'='1`

So the query is going to be like this:

```
SELECT * FROM users WHERE username='' OR '1'='1' AND password='password123'
```

`'1'='1'` is always true, so the whole condition becomes true no matter what the real password is. The database just returns the first user (usually admin) and you're in.

Or you can also type something like: `' --` because `--` means a comment, so everything in query after `username=''` is going to be a comment.

<img width="1271" height="107" alt="image" src="https://github.com/user-attachments/assets/85edfc96-23f1-4f3a-9694-15996d9dccea" />

It got a filter for `--`, but not for `' OR '1'='1` payload. We got a password, but it's useless. 

Since we know that we can use SQLi (SQL injection), now we need to get the admin username and it's password from the database. I found very useful repository that contains a lot of payloads for different types of SQL: https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection

Let's find out which exactly SQL we have. We need to get some errors, and we're going to figure it out from them. 

<img width="424" height="51" alt="image" src="https://github.com/user-attachments/assets/c19da94c-0ccb-4e48-8a33-ae1262794765" />

Google the error:

<img width="891" height="426" alt="image" src="https://github.com/user-attachments/assets/6ece7292-c98a-4637-91ca-a500ee563640" />

It's SQLite. Now we can use some payloads from that repository:

<img width="1128" height="568" alt="image" src="https://github.com/user-attachments/assets/1f05d951-875e-43b6-ab68-b9074b9eeac3" />

We need to "Extract Table Name". I'll use this payload:

```
SELECT tbl_name FROM sqlite_master WHERE type='table
```

<img width="1169" height="53" alt="image" src="https://github.com/user-attachments/assets/a478a967-9d52-4350-85f3-d4806bb7d251" />

It has a filter for some words like SELECT or FROM, we can bypass it by changing the case of words. 

<img width="1186" height="53" alt="image" src="https://github.com/user-attachments/assets/4c453590-e70a-47de-8e46-0e7a89a3497c" />

I got an error. Why? So, let's remember the structure of the query:

```
SELECT * FROM users WHERE username='admin' AND password='password123'
```

So when we use that payload, the query looks like this:

```
SELECT * FROM users WHERE username='' SeLect tbl_name FrOm sqlite_master WheRe type='table' AND password='password123'
```

We want to run our own query `SELECT tbl_name FROM sqlite_master WHERE type='table'`. But you can't just paste it after the original one, the database would see two SELECT keywords back to back and give an error.

`UNION` is the word that lets us run a second SELECT and show both results together. So the final query becomes:

```
SELECT * FROM users WHERE username='' AND password=''
UNION
SELECT tbl_name FROM sqlite_master WHERE type='table'
```
> Example of structure

So i ran this:
```
' UnIon SeLect tbl_name FrOm sqlite_master WheRe type='table
```

<img width="1258" height="54" alt="image" src="https://github.com/user-attachments/assets/100dde05-69de-4c65-98b9-7d293e69dcaf" />

We got table's name. Now we can see what's inside it by using this payload:
```
SELECT sql FROM sqlite_master WHERE tbl_name='<TABLE_NAME>
```

<img width="1288" height="128" alt="image" src="https://github.com/user-attachments/assets/adfa0c37-c635-4b04-8bfa-9e11765a29ea" />

We got three columns: `id`, `username`, `password`. Let's check them by using this payload:

```
' UNION SELECT username FROM 'admintable
' UNION SELECT password FROM 'admintable
```

<img width="962" height="107" alt="image" src="https://github.com/user-attachments/assets/71b3e51e-e698-4e15-bbec-40eca294c1d4" />

In username column we have admin's username, and in password column we have the flag. Let's search for admin's password, using his username:

```
' UnIon SeLect password FroM 'admintable' WheRe username='(admins_username)
```

<img width="1409" height="55" alt="image" src="https://github.com/user-attachments/assets/1f35e210-1909-42c8-9d07-e3235bed7e69" />

And we have admin's password.

## Lessons Learned

I had to learn SQL injection basics to solve this challenge, so this is why I tried to make a simple explanation here. Check the forms using `'` to see if it's using SQL. Actually, SQLi is really interesting and I'm goig to learn it deeper.
