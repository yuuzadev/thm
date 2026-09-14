# Fools Mate Write-up | 报告

<details>
  <summary>Click to view in Chinese (点击查看中文版)</summary>

---



  </details>

---

This is my write-up for the [Fools Mate](https://tryhackme.com/room/foolsmate) room. 

# Overview
The goal is to bypass the engine and get the flag. It is on a web app on `http://machine_ip/`.

# Reconnaissance
After getting machine IP, I opened `http://machine_ip/` and saw the EndgameTrainer web page.

<img width="1909" height="821" alt="image" src="https://github.com/user-attachments/assets/80803ce4-7112-4daf-8261-ef8020400e6a" />

This is just a chess application, we can move figures, do anything, but when it comes to rook (the figure at the bottom-left at the beginning) - I moved it to the top-left corner, and I got fake error window:

<img width="828" height="414" alt="image" src="https://github.com/user-attachments/assets/0b448dfa-96e3-4795-81ab-6780de55f991" />

It blocks this move and we're not able to do it. I focused on this, and since the challenge is about engine bypass, I opened devtools by presing Ctrl + Shift + C to see what do we have about this "error" window.

In "Debugger" -> "Sources", I opened machine IP's folder, then the "js" folder, and found the "app.js", the application that is responsible for "error".

<img width="1900" height="752" alt="image" src="https://github.com/user-attachments/assets/d3a1bcef-9119-48c2-b24a-ce8230053ced" />

Let's read the code of the error window (you can find it by pressing Ctrl + F (Find) and typing keywords, "shut" for example):

```
  if (result && probe.isCheckmate()) {
    showSystemNotice('I\'ll shut down your PC if you play that.');
    return false;
  }
  return true;
}
```

Here's the thing, since we have `return false;` line, we'll keep getting error window. We need to change "false" to "true", and see what happens. 

# Solution

You can change the code of apps in web applications, but it will work only locally. To do that, right click on app.js on the left, and choose "Add script override".

<img width="1225" height="786" alt="image" src="https://github.com/user-attachments/assets/bdb5a1d3-b070-48c2-85d7-e0449e426632" />

You just need to save it wherever you want. After that, you need to open that file with any text/code editor, I'll use nano:

<img width="398" height="132" alt="image" src="https://github.com/user-attachments/assets/bfa7d5b0-2551-4a8b-83b9-14122decb0e8" />

<img width="1233" height="775" alt="image" src="https://github.com/user-attachments/assets/cf71e982-e629-4ebc-903b-66603cebae72" />

Search for this block of code (you can find it by pressing Ctrl + F and typing keywords, "shut" for example):

```
  if (result && probe.isCheckmate()) {
    showSystemNotice('I\'ll shut down your PC if you play that.');
    return false;
  }
  return true;
}
```

And change the `return false;` line to `return true;`:

<img width="827" height="523" alt="image" src="https://github.com/user-attachments/assets/a01341b4-896b-48c4-8160-c5bf91c0b58c" />

Then save the file (if you're using nano too: Ctrl + O then hit Enter, Ctrl + X ), go back to the web page and press Ctrl + F5 ro reload it. Now let's try to move rook to the top-left corner:

And we see a flag on top right:

<img width="1210" height="739" alt="image" src="https://github.com/user-attachments/assets/23e0544c-5dd9-453c-95b8-11ae3385dc26" />

## Lessons Learned

Read and experimentate with source codes. They can handle hidden useful info. Especially if it's something that stops you.
