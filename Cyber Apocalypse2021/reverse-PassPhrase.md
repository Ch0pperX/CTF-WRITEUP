CHTB2021-PassPhrase
===
## Info
**題目敘述**

You found one of their space suits forgotten in a room. You wear it, but before you go away, a guard stops you and asks some questions.

**能力**
- 靜態分析(ELF檔)


**難易度**
★☆☆☆☆


## Write up
### 靜態分析
觀察`main`函式，輸入的`s`會和`s1`做比較，`s1`第一個字元是`'3'`，後面的字元要待程式執行時才會定義
當程式執行時，`var_4F`會定義為`'x'`，依序`var_4E`定義為`'t'`....  
因此整個`s1`的字串為`'3xtr4.....hum4n5'`
![](https://i.imgur.com/2Mpk6Ma.png)
