CHTB2021-Authenticator
===
## Info
**題目敘述**
We managed to steal one of the extraterrestrials' authenticator device. If we manage to understand how it works and get their credentials,we may be able to bypass all of their security locked doors and gain access everywhere!

**能力**
- 靜態分析(ELF檔)
- 程式撰寫能力

**難易度**
★☆☆☆☆

###### tags: `CTF-WRITEUP`

## Write Up
### 靜態分析
使用IDA PRO分析檔案，並鎖定`MAIN`函式

![](https://i.imgur.com/bGpMKFE.png)

從這函式可以看到使用者要輸入兩項資訊：
- Alien ID
    - 從第14行可以看到id必須為`11337`
- Pin
    - 將讀取的pin送進`checkpin`進行驗證

觀察`checkpin`函式內容，可以發現會和`aAVhAG8j89gvPDv[i] ^ 9`做比較

![](https://i.imgur.com/7Yd0vxa.png)

- aAVhAG8j89gvPDv
    - ![](https://i.imgur.com/exxHLSU.png)

### sol.py

寫個程式完成`aAVhAG8j89gvPDv[i] ^ 9`即可知道Pin應該是多少
```python
s='}a:Vh|}a:g}8j=}89gV<p<}:dV8<Vg9}V<9V<:j|{:'
flag=''
for i in range (len(s)):
    flag+=chr(ord(s[i])^9)

print(flag)
```