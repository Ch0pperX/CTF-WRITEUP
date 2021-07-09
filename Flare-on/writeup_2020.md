Flare-on 7 (2020)
===
[TOC]
## challenge1：fidler
### Info
**題目敘述**
Welcome to the Seventh Flare-On Challenge!

This is a simple game. Win it by any means necessary and the victory screen will reveal the flag. Enter the flag here on this site to score and move on to the next level.

This challenge is written in Python and is distributed as a runnable EXE and matching source code for your convenience. You can run the source code directly on any Python platform with PyGame if you would prefer.

**能力**
- python

**flag**
idle_with_kitty@flare-on.com

### Write Up
執行`fidler.exe`會先出現一個視窗，要求輸入密碼

![](https://i.imgur.com/Z9iw8Xe.png)

如果密碼輸入錯誤，會出面錯誤的畫面

![](https://i.imgur.com/yaJX4dv.png)

打開題目提供的原始碼，可以看到密碼驗證的方式：

```python
def password_check(input):
    altered_key = 'hiptu'
    key = ''.join([chr(ord(x) - 1) for x in altered_key])
    return input == key
```
依據此驗證方式推測密碼為`ghost`
```python
altered_key = 'hiptu'
key = ''.join([chr(ord(x) - 1) for x in altered_key])
print('password: %s' % key)

-----------
[Output]
password: ghost
```

當密碼驗證通過後會出現 **點貓貓取得金幣** 的畫面

![](https://i.imgur.com/2ygsHUP.png)

點一次貓頭可以獲得1個金幣，要**100 Billion**才能夠買到flag..
觀看原始碼，看看有沒有寫flag相關資訊，發現
```python
def decode_flag(frob):
    last_value = frob
    encoded_flag = [1135, 1038, 1126, 1028, 1117, 1071, 1094, 1077, 1121, 1087, 1110, 1092, 1072, 1095, 1090, 1027,
                    1127, 1040, 1137, 1030, 1127, 1099, 1062, 1101, 1123, 1027, 1136, 1054]
    decoded_flag = []

    for i in range(len(encoded_flag)):
        c = encoded_flag[i]
        val = (c - ((i%2)*1 + (i%3)*2)) ^ last_value
        decoded_flag.append(val)
        last_value = c

    return ''.join([chr(x) for x in decoded_flag])
```
只要找到`decode_flag`函式的參數`frob`就可以自己算flag
而`frob`參數可以追溯到`game_screen`函式
```python
while not done:
    target_amount = (2**36) + (2**35)
    if current_coins > (target_amount - 2**20):
        while current_coins >= (target_amount + 2**20):
                current_coins -= 2**20
        victory_screen(int(current_coins / 10**8))
        return
```
`prob`參數為`int(current_coins / 10**8)`的結果，而`current_coins`會經過一次`if`判斷和`while`運算
- `target_amount`為 `103079215104`，`current_coins`需大於`103078166528` (`target_amount - 2**20`)
- 如果`current_coins`大於等於`103080263680`(`target_amount + 2**20`)，就必須一次扣掉`1048576`(`2**20`)，一直到小於`103080263680`為止
- 因此我們可以確定`current_coins`的範圍為`103080263679`~`103078166529`之間
- 而這區間的數字經過`int(current_coins / 10**8)`運算皆為`1030`

因此`prob`為`1030`，將已知參數運用要flag解碼上，即可得到flag
```python
def decode_flag(frob):
    last_value = frob
    encoded_flag = [1135, 1038, 1126, 1028, 1117, 1071, 1094, 1077, 1121, 1087, 1110, 1092, 1072, 1095, 1090, 1027,
                    1127, 1040, 1137, 1030, 1127, 1099, 1062, 1101, 1123, 1027, 1136, 1054]
    decoded_flag = []

    for i in range(len(encoded_flag)):
        c = encoded_flag[i]
        val = (c - ((i%2)*1 + (i%3)*2)) ^ last_value
        decoded_flag.append(val)
        last_value = c

    return ''.join([chr(x) for x in decoded_flag])

print('flag: %s' % decode_flag(1030))

-----------
[Output]
flag: idle_with_kitty@flare-on.com
```

## challenge2：garbage
### Info
**題目敘述**
One of our team members developed a Flare-On challenge but accidentally deleted it. We recovered it using extreme digital forensic techniques but it seems to be corrupted. We would fix it but we are too busy solving today's most important information security threats affecting our global economy. You should be able to get it working again, reverse engineer it, and acquire the flag.

**能力**
- PE檔案分析
- 檔案修復
- upx

**flag**
C0rruptGarbag3@flare-on.com

### Write Up
題目提供一個EXE執行檔，但卻無法順利執行該檔案

![](https://i.imgur.com/XYVwd75.png)

使用[die](https://horsicq.github.io/)查看檔案，得知這是WIN32的執行檔，且有被UPX加殼過

![](https://i.imgur.com/vobsJ12.png)

但是我們透過[UPX](https://upx.github.io/)進行解殼，卻失敗了，失敗訊息顯示**檔案大小**有錯誤

使用[CFF Explore](https://ntcore.com/?page_id=388)觀察，可以看到`file size`和`PE size`大小不一樣，推測這個檔案應該有經過人工的截斷，在現實場景中可能是鑑識過程中也很常發生類似問題。

![](https://i.imgur.com/1VvMh66.png)


直覺做法就是用[HxD Editor](https://mh-nexus.de/en/hxd/)幫他新增`null byte`塞到足夠大小，共要塞732個bytes。當塞完732個null byte後就可以正常解殼了

![](https://i.imgur.com/gOqIfll.png)

解殼後還是不能正常執行

![](https://i.imgur.com/kBu687I.png)

這是因為雖然我們補足了缺少的檔案，但最後的`configurtion`並不完整

![](https://i.imgur.com/Zct9243.png)

透過CFF Explore將不完整的configuration刪除

![](https://i.imgur.com/57t0v4U.png)

刪除後卻出現另一項錯誤，看氣來是import table出現錯誤，這個有可能是在解upx殼時發生的錯誤

![](https://i.imgur.com/1qcLhAo.png)

透過CFF Explore查看import table，可以透過dll中的function知道原本dll的名稱  
例如，[WriteFile](https://docs.microsoft.com/en-us/windows/win32/api/fileapi/nf-fileapi-writefile)透過windows官方文件可以查到屬於kernel32.dll
![](https://i.imgur.com/kGcPoHs.png)

![](https://i.imgur.com/mMC23js.png)

修復完後，終於可以正常執行了!!! :white_flower: 

![](https://i.imgur.com/2Oxl68A.png)

## challenge3：wednesday