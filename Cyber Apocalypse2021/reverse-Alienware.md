CHTB2021-Alienware
===
## Info
**題目敘述**

We discovered this tool in the E.T. toolkit which they used to encrypt and exfiltrate files from infected systems. Can you help us recover the files?

**能力**
- 靜態分析exe檔
- 動態分析提取dll檔
- 靜態分析dll檔
- 動態分析
- 查詢windows API的使用手冊
- 動態修改calling table和參數

**難易度**
難易度：★★★☆☆
###### tags: `CTF-WRITEUP`


## Write_Up
### 使用 IDA Pro競態分析
`SHIFT + F12`看到`xuTaV.dll`、`%s%s`和`encryptFiles`

![](https://i.imgur.com/eAya118.png )

追蹤字串位置在`TlsCallback_0`

![](https://i.imgur.com/NfFtQwn.png)

`TlsCallback_0`做了幾件事：
1. 載入Resource檔案

    ![](https://i.imgur.com/jIgiSaj.png)

2. 向系統申請一段空間並初始為0
3. 逐Byte做xor解碼
4. 將每一個解碼完Byte存進去剛剛申請的空間
5. 寫檔並命名為xuTaV.dll
6. 動態載入xuTaV.dll並使用其中encryptFiles的API

![](https://i.imgur.com/sxCi1P4.png)

因此目標在取得xuTaV.dll並分析encryptFiles怎麼實作加密功能

### 動態分析找出 xuTaV.dll
方法一：從x64dbg的記憶體中提取

![](https://i.imgur.com/OQGUGCT.png)

方法二：程式執行到一半到存放xuTaV.dll的位置拉出來就好
`GetTempPathA`會取得當下User目錄下`Temp`得位置

![](https://i.imgur.com/JcB8VQB.png)

檔案會存在這個目錄下

![](https://i.imgur.com/MhSzWUy.png)

### 靜態分析xuTaV.dll
直接定位到`encryptFiles`的位置
`encryptFiles`做了幾件事：
1. 將 `C:\\ User \\ [user_name] \\ Docs\\ . `路徑下的所有檔案逐一加密
2. 加密後的檔案會另存新檔，並"再"加上附檔名`.alien`
3. 透過`sub_1800011C0`函式進行檔案加密，完成後，會將原本的檔案刪除
 
![](https://i.imgur.com/LTQlUAT.png)

`sub_1800011C0`做了幾件事：
1. 利用`CryptDeriveKey`產生加密的金鑰
2. 使用`CryptEncrypt`函式並引用上一步產生的金鑰進行加密

觀察微軟對這兩個API的說明
- `CryptDeriveKey`
    - Important  This API is deprecated. New and existing software should start using Cryptography Next Generation APIs. Microsoft may remove this API in future releases.
    - The CryptDeriveKey function generates cryptographic session keys derived from a base data value. **This function guarantees that when the same cryptographic service provider (CSP) and algorithms are used, the keys generated from the same base data are identical. The base data can be a password or any other user data.**
    - 加密和解密使用同一把金鑰，且當CSP相同，金鑰會相同
        - 這樣就不用找解密金鑰了~直接用現成的就好 :+1: 
- `CryptEncrypt`
    - Important  This API is deprecated. New and existing software should start using Cryptography Next Generation APIs. Microsoft may remove this API in future releases.
    - The CryptEncrypt function encrypts data. The algorithm used to encrypt the data is designated by the key held by the CSP module and is referenced by the hKey parameter.
        ```
        BOOL CryptEncrypt(
        HCRYPTKEY  hKey,
        HCRYPTHASH hHash,
        BOOL       Final,
        DWORD      dwFlags,
        BYTE       *pbData,
        DWORD      *pdwDataLen,
        DWORD      dwBufLen
          );
        ```
    - 觀察加密和解密(`CryptDecrypt`)的差別--->差一個`dwBufLen`
        ```
        BOOL CryptDecrypt(
        HCRYPTKEY  hKey,
        HCRYPTHASH hHash,
        BOOL       Final,
        DWORD      dwFlags,
        BYTE       *pbData,
        DWORD      *pdwDataLen
        );
        ```
### 動態分析xuTaV.dll
透過動態分析驗證思緒：
1. 建立 C:\\ User \\ [user_name] \\ Docs\\  路徑
2. 將 Confidential.pdf.alien 放在指定路徑下
3. 動態分析將「 斷點 」下在 CryptEncrypt 附近
4. 將多餘的參數刪掉 
5. 修改API呼叫表
6. 將「斷點」下在「 寫檔完成之後」，再檢查指定資料夾下是否解密成功 :question: 

以下為成式執行的流程，再動態分析中有3個重點

![](https://i.imgur.com/1j7431H.png)

1.`NOP`不必要的參數

![](https://i.imgur.com/IwL3l5G.png)

2.修改 CryptEncrypt -> CryptDecrypt

修改後：
![](https://i.imgur.com/Kd0nDlA.png)

3.設斷點在完成寫檔，避免原檔案被刪除

最後會取得「Confidential.pdf.alien.alien」檔案，將副檔名都刪掉打開看看是否會成功^^

![](https://i.imgur.com/j4ocBHZ.png)




