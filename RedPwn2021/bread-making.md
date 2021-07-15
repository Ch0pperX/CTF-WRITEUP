Redpwn2021 - bread-making
===
## Info
**題目敘述**
My parents aren't home! Quick, help me make some bread please...
nc mc.ax 31796


**能力**
- ELF靜態分析
- ELF動態分析

**難易度**
★★★☆☆

**FLAG**
flag{m4yb3_try_f0ccac1a_n3xt_t1m3???0r_dont_b4k3_br3ad_at_m1dnight}

## WriteUp
題目是ELF執行檔，執行起來像要輸入什麼東西，但沒有任何提示，所以摸不著方向...

![](https://i.imgur.com/K0eO2QM.png)

### 靜態分析
`main`函式中可以看到一個while組成的巢狀迴圈

![](https://i.imgur.com/uuqTYHa.png)

而在巢狀迴圈之後接著一些判斷式，在判斷式最內圈呼叫`sub_25c0`

![](https://i.imgur.com/oVaAcKp.png)

`sub_25c0`主要實現讀檔及輸出檔案內容，而檔案名稱為`flag.txt`，很明顯是我們的目標

![](https://i.imgur.com/ygeHeqV.png)

整體邏輯總結下來，透過巢狀迴圈讀取Input的東西，然後比較後在記憶體的指定位置標記(`dword_641c`、`dword_6418`、`dword_6414`、`dword_6410`、`dword_6400c`)，最後檢查這些標記，若都符合則讀檔並印出flag。

利用字串明文(`shift`+`F12`)檢查，可以看到除了pseudo code以外的字串，猜測這些字串應該是用來跟input比較的。

![](https://i.imgur.com/ggW5LIv.png)




### 搭配動態分析
動態分析時得知A迴圈的`sub_20E0`為`puts`功能，而`&off_6020`是一個table

![](https://i.imgur.com/XNEd4oW.png)

v7應該是會根據56行而變化，推測v7範圍為0~11

```
v7=0;
++qword_6440;
v7=qword_6440;
```

<FONT SIZE='2'>

| Round(v7)| off_6020 | PUT_content | PUT_content(hint) | choices |
| -------- | -------- | -------- |-----|-----|
|   0      | unk_6340 | add ingredients to the bowl  |we don't have that ingredient at home! | 4|
|   1      | unk_61A0 | the ingredients are added and stirred into a lumpy dough     |mom comes home before you find a place to put the bowl |3|
|   2     | unk_6120 |   the bread needs to rise   | the dough has been forgotten, making an awful smell the next morn | 2|
|   3      | unk_6300 |   it is time to finish the dough   | you've shuffled around too long, mom wakes up and sees you | 2|
|   4      | unk_6160 | the dough is done, and needs to be baked     |the dough wants to be baked | 2|
|   5      | unk_63A0 |  the bread is in the oven, and bakes for 45 minutes    |you've forgotten how long the bread bakes|2|
|   6      | unk_6200 | 45 minutes is an awfully long time  | you've moved around too much and mom wakes up, seeing you bread |2|
|   7      | unk_6080 | there's no time to waste   |the flaming loaf sets the kitchen on fire, setting off the fire alarm and waking up the entire house| 2|
|   8      | unk_62A0 | there's smoke in the air |one of the fire alarms in the house triggers, waking up the entire house|3|
|   9     | unk_6240 | the kitchen is a mess | you've taken too long and fall asleep |4|
|   10     | unk_60C0 | time to go to sleep | you've taken too long and fall asleep |3

</FONT>


透過動態分析看每一次input後的字串，做了哪些處理...
動態分析追蹤到第一個比較方式為`strcspn`，這是為了把每一次`fgets`字串尾端的`\n`改成`\0`

#### `v7=0`：add ingredients to the bowl

![](https://i.imgur.com/uPlrVYY.png)

透過`strcmp`比較輸入的字串，共比較四次，分別為`add flour`、`add yeast`、`add salt`、`add water` 
- 若為`add flour`，呼叫`sub_2A50`，將`dword_6438`設成`1`，檢查`dword_6438`、`dword_6434`、`dword_6430`、`dword_642C`是否有值
    - 若皆有值，回傳0，不繼續迴圈B，`v7`+1，回到迴圈A，進入下一回合
    - 若其中有一個無值，回傳1，繼續迴圈B，`fgets`需要的材料
    
    ![](https://i.imgur.com/5zDABWy.png)
    
- 若為`add yeast`，呼叫`sub_2AB0`，將`dword_6434`設成`1`，後續流程和`add flour`相同
- 若為`add salt`，呼叫`sub_2B10`，將`dword_6430`設成`1`，後續流程和`add flour`相同
- 若為`add water`，呼叫`sub_2B70`，將`dword_642C`設成`1`，後續流程和`add flour`相同
- 若皆非上述4項，就到`LABEL_17`
    

    
#### `v7=1`：the ingredients are added and stirred into a lumpy dough

透過`strcmp`比較輸入的字串，共比較三次，分別為`leave the bowl on the counter`、`put the bowl on the bookshelf`、`hide the bowl inside a box`
- 若為`leave the bowl on the counter`，則會呼叫`sub_27A0`，輸出`mom comes home and finds the bowl`，回傳`-1`
    - 回傳`-1`會導致47行條件不成立，輸出`mom comes home before you find a place to put the bowl`後，往`LABEL_18`執行，最後程式結束
- 若為`put the bowl on the bookshelf`，會和`leave the bowl on the counter`依樣狀況
- 若為`hide the bowl inside a box`，輸出`the box is nice and warm`，回傳0，
    - 程式流程回到迴圈A，進入下一回合
#### 小結
- 透過`off_6020`表可以追查程式預期輸入的選項，和經過子函式回傳的值判斷正確的選項為何
- 若選項為多選，如`v7=0`，只要將所有選項輸入，並且不用考入順序
- 若選項為單選，如`V7=1`，子函式回傳為0者，即為正解 
- 其實看懂流程後可以發現因為比較順序的關係，如果是單選，答案會是最後一個選項


| Round(v7)| off_6020 | single/multi | correct(return 0 /1) |
| -------- | -------- | -------- |-----|
|   0      | unk_6340 | M |add flour<br>add yeast<br>add salt<br>add water|
|   1      | unk_61A0 | S|hide the bowl inside a box|
|   2      | unk_6120 | S|wait 3 hours|
|   3      | unk_6300 | S|work in the basement|
|   4      | unk_6160 | S|preheat the toaster oven|
|   5      | unk_63A0 | S|set a timer on your phone|
|   6      | unk_6200 | S|watch the bread bake|
|   7      | unk_6080 | S|pull the tray out with a towel|
|   8      | unk_62A0 | M|unplug the oven<br>unplug the fire alarm<br>open the window|
|   9     | unk_6240 | M|wash the sink<br>clean the counters<br>flush the bread down the toilet<br>get ready to sleep|
|   10     | unk_60C0 | M|close the window<br>replace the fire alarm<br>brush teeth and go to bed|

### sol.py
```python=
from pwn import *

#p = process('bread')
p = remote('mc.ax',31796)

p.recvuntil('\n')
#round0
p.sendline('add flour')
p.recvuntil('\n')
p.sendline('add yeast')
p.recvuntil('\n')
p.sendline('add salt')
p.recvuntil('\n')
p.sendline('add water')
p.recvuntil('\n')
#round1
p.sendline('hide the bowl inside a box')
p.recvuntil('\n')
#round2
p.sendline('wait 3 hours')
p.recvuntil('\n')
#round3
p.sendline('work in the basement')
p.recvuntil('\n')
#round4
p.sendline('preheat the toaster oven')
p.recvuntil('\n')
#round5
p.sendline('set a timer on your phone')
p.recvuntil('\n')
#round6
p.sendline('watch the bread bake')
p.recvuntil('\n')
#round7
p.sendline('pull the tray out with a towel')
p.recvuntil('\n')
#round18
p.sendline('unplug the oven')
p.recvuntil('\n')
p.sendline('unplug the fire alarm')
p.recvuntil('\n')
p.sendline('open the window')
p.recvuntil('\n')
#round9
p.sendline('wash the sink')
p.recvuntil('\n')
p.sendline('clean the counters')
p.recvuntil('\n')
p.sendline('flush the bread down the toilet')
p.recvuntil('\n')
p.sendline('get ready to sleep')
p.recvuntil('\n')
#round10
p.sendline('close the window')
p.recvuntil('\n')
p.sendline('replace the fire alarm')
p.recvuntil('\n')
p.sendline('brush teeth and go to bed')

p.interactive()
```

![](https://i.imgur.com/coMk1cS.png)
