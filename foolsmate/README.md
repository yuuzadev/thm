# Fools Mate Write-up | 报告

<details>
  <summary>Click to view in Chinese (点击查看中文版)</summary>

---

这是我对 [Fools Mate](https://tryhackme.com/room/foolsmate) 房间的write-up。

# 概述
目标是绕过引擎并获取flag。它位于 `http://machine_ip/` 上的一个Web应用中。

# 侦察
获取到机器IP后，我打开了 `http://machine_ip/`，看到了 EndgameTrainer 网页。

<img width="1909" height="821" alt="image" src="https://github.com/user-attachments/assets/80803ce4-7112-4daf-8261-ef8020400e6a" />

这只是一个国际象棋应用，我们可以移动棋子，做任何操作，但当涉及到车（开局时左下角的那个棋子）时——我把它移到了左上角，然后得到了一个假的错误窗口：

<img width="828" height="414" alt="image" src="https://github.com/user-attachments/assets/0b448dfa-96e3-4795-81ab-6780de55f991" />

它阻止了这个移动，我们无法完成它。我把注意力集中在这上面，由于这个挑战是关于引擎绕过的，我按 Ctrl + Shift + C 打开了开发者工具，看看关于这个"错误"窗口我们能找到什么。

在"Debugger" -> "Sources"中，我打开了机器IP的文件夹，然后是"js"文件夹，找到了"app.js"，即负责"错误"的应用程序。

<img width="1900" height="752" alt="image" src="https://github.com/user-attachments/assets/d3a1bcef-9119-48c2-b24a-ce8230053ced" />

让我们阅读错误窗口的代码（你可以按 Ctrl + F（查找）并输入关键词，比如"shut"来找到它）：

```
  if (result && probe.isCheckmate()) {
    showSystemNotice('I\'ll shut down your PC if you play that.');
    return false;
  }
  return true;
}
```
问题在这里，由于我们有 return false; 这一行，我们会一直得到错误窗口。我们需要把"false"改成"true"，看看会发生什么。

# 解决方案
你可以修改Web应用中应用程序的代码，但它只会在本地生效。要做到这一点，右键点击左侧的 app.js，然后选择"Add script override"。

<img width="1225" height="786" alt="image" src="https://github.com/user-attachments/assets/bdb5a1d3-b070-48c2-85d7-e0449e426632" />

你只需要把它保存到任何你想要的地方。之后，你需要用任何文本/代码编辑器打开那个文件，我将使用 nano：

<img width="398" height="132" alt="image" src="https://github.com/user-attachments/assets/bfa7d5b0-2551-4a8b-83b9-14122decb0e8" />

<img width="1233" height="775" alt="image" src="https://github.com/user-attachments/assets/cf71e982-e629-4ebc-903b-66603cebae72" />

搜索这段代码（你可以按 Ctrl + F 并输入关键词，比如"shut"来找到它）：

```
  if (result && probe.isCheckmate()) {
    showSystemNotice('I\'ll shut down your PC if you play that.');
    return false;
  }
  return true;
}
```
然后把 return false; 这一行改成 return true;：

<img width="827" height="523" alt="image" src="https://github.com/user-attachments/assets/a01341b4-896b-48c4-8160-c5bf91c0b58c" />

然后保存文件（如果你也使用 nano：Ctrl + O 然后按 Enter，Ctrl + X），回到网页并按 Ctrl + F5 重新加载它。现在让我们尝试把车移到左上角：

然后我们在右上角看到了flag：

<img width="1210" height="739" alt="image" src="https://github.com/user-attachments/assets/23e0544c-5dd9-453c-95b8-11ae3385dc26" />

## 经验教训
阅读并尝试源代码。它们可能包含隐藏的有用信息。尤其是当某些东西阻止你的时候。

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
