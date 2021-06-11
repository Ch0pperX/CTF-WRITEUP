RITSEC CTF2021- PleaseClickAlltheThing:IceID_Bokbot
===
## Info
**題目敘述**

Note: this challenge is the start of a series of challenges. The purpose of this CTF challenge is to bring real world phishing attachments to the challengers and attempt to find flags (previously executables or malicious domains) within the macros. This is often a process used in IR teams and becomes an extremely valuable skill. In this challenge we’ve brought to the table a malicious html file, GandCrab/Ursnif sample, and a IceID/Bokbot sample. We’ve rewritten the code to not contain malicious execution however system changes may still occur when executing, also some of the functionalities have been snipped and will likely not expose itself via dynamic analysis.

-  Outlook helps, with proper licensing to access necessary features
    ◦ Otherwise oledump or similar would also help but isn’t necessary
-  CyberChef is the ideal tool to use for decoding
This challenge is brought to you by SRA

*PASSWORD: RITSEC*


**能力**
- VBA

**難易度**
★★☆☆☆
###### tags: `CTF-WRITEUP`

## Write Up
檔案巨集有以下內容，執行時會跳出「找不到AutoOpen」的alert
找到`main`並更改module名稱，只要執行main段即可

![](https://i.imgur.com/2w7g8Hl.png)

`main`位於Module2，會對一連串的被加密的資訊進行解密

![](https://i.imgur.com/cjzbFBY.png)


其中Module的`aqv6tf`看起來像flag
```
Public Const aqv6tf As String = "EVGFRP{E0GG1ATZ@YP0Q3}"
```

故監看此字串就可以得到一個flag

![](https://i.imgur.com/c7nuXTb.png)

