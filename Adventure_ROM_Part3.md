UTCTF2021-Adventure ROM Part 3
===
## Info
**題目敘述**
- hint1:
The file is a Game Boy ROM and should run fine in most emulators.
- hint2:
You do not need to modify the game to win and make it print the flag -- you just need to play the game "correctly"

**能力**
- 檔案分析
- 尋找合適的工具
- 靜態分析(x16)
- 動態分析

** 難易度**
★★★☆☆

###### tags: `CTF-WRITEUP`

## Write Up

### GB檔
掌上型遊戲機GameBoy的遊戲檔，
GB Studio是款圖型化2D RPG遊戲開發工具，
讓沒有程式開發經驗的一般使用者也能製作出自己的遊戲，
並可透過整合工具將遊戲封裝為Game Boy專用的Rom檔案。

### 使用工具
1. [Ghidra](https://ghidra-sre.org/) 
    -  [GhidraBoy plugin](https://github.com/Gekkio/GhidraBoy)
    -  要安裝GhidraBoy extension才可以分析，安裝成功會發現指令即架構有GameBoy的選項
     ![](https://i.imgur.com/kBrerPv.png)
     
2. [bgb debugger](https://bgb.bircd.org/)


### 靜態分析
透過明文字串追蹤，找到遊戲起始點為在`FUN_097F()`
遊戲中，總共要吃10把鑰匙才會結束，吃鑰匙的動作會在`DO-WHILE`中迴圈實作。

![](https://i.imgur.com/ViGuKEQ.png =500x)

分析回圈內的函式
- FUN_032f()
    - 判斷遊戲結束與否，若`DAT_cb7b='\0'`，則代表輸了這場遊戲
    ![](https://i.imgur.com/vkXdoAx.png =500x)
    - `DAT_0551`和`DAT_0702`顯然式FLAG存放的位置，但追蹤過去看到的是亂碼，應該是有ENCODE過
    ![](https://i.imgur.com/Miq4zGq.png)


- FUN_09ac()
    - 呼叫許多`FUN_0574()`
        - 取記憶體某些資料進行運算比較，若符合條件，則將`DAT_cb7b`歸零
        ![](https://i.imgur.com/3163hlk.png)
        
        - 當符合```(char)((DAT_cb7a =='\n') << 7) < '\0'```,則會呼叫`FUN_032f()` 

### 動態分析
透過動態分析去追蹤條件判斷中`DAT_cb66`、`DAT_cb7a`、`DAT_cd62`和`DAT_cd63`分別代表什麼資料

當小人往 **上** 走會發現**DAT_cd63**從**16變成15**,可以確定**DAT_cd63**代表**y軸座標**，上方數字較小  
同理可得**DAT_cd62**為x軸座標，左方數字較小

![](https://i.imgur.com/RfdBuFb.png =500x)

![](https://i.imgur.com/Mq5ZzHy.png =500x)

```c
sVar1=((ushort)DAT_cd63 * 0x10 + (ushort)DAT_cd63 * 9) * 2;
bVar2=(byte)sVar1;
```
`bVar2`代表`50*y`取一個byte大小的值

![](https://i.imgur.com/hjZqNxO.png)

**DAT_cb7a**為當下鑰匙的數量

![](https://i.imgur.com/8Pkfeqo.png)


而DAT_cb66是字元陣列，index為(鑰匙數量*2)，故用來比較的值為黃底的值
![](https://i.imgur.com/J2YKOpy.png)

找出所有鑰匙的座標並依上述邏輯做比較，就可以知道鑰匙的順序了
![](https://i.imgur.com/7Ew2JSG.png)

依照順序去吃完鑰匙就完成了。





