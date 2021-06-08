UCTF2021-Recur
===
## Info
**題目敘述**
I found this binary that is supposed to print flags.
It doesn't seem to work properly though...

**能力**
- 靜態分析
- 程式撰寫
- 演算法

**難易度**
★★☆☆☆

###### tags: `CTF-WRITEUP`

## Write_Up
將檔案放進IDA Pro 64-bit的環境裡
Main和Recurrence函式顯而易見
### Main
![](https://i.imgur.com/EoUx6p9.png)

- flag為長度28的字元陣列
    ![](https://i.imgur.com/WPcSagN.png)
    
- 將$flag[i]$依序與$Recurrence(i^2)$做xor的運算
- 將運算後的結果印出來-->FLAG
- 
### Recurrence
![](https://i.imgur.com/3qfPUGu.png)

- $i=0$ , 回傳 **3**
- $i=1$ , 回傳 **5**
- 若$i\neq0$ 且 $i\neq0$ , 回傳遞迴數列 

$\begin{cases}
f(o)=3\\
f(1)=5\\
f(n)=2f(n-1)+3f(n-2)
\end{cases}$.

如果程式碼照上述聯立方程式刻，會跑不出來~
因為i值最大為**27x27**，這樣就會有無窮的遞迴

若將每一次的結果存成數列，在取前兩個值來運算，會比每次重新做遞迴來得好

因此需要建立一個整數陣列存放數值\這樣就可以順利得到f(0)、f(1)、f(4)、f(9)、f(16)、f(25) .....f(729)的結果，並進行xor解碼

### Sol.c.
```c
# include <stdio.h>


int rec(int a1){
    int v2;
    int a2=a1*a1;
    if(!a2) return 3;
    if(a2==1) return 5;
    v2=2*rec((a2-1));
    return v2+3*rec((a2-2));    
}

int main(void) {
    int v3;
    int v4;
    int flag[28]={0x76, 0x71, 0xC5, 0xA9, 0xE2, 0x22, 0xD8, 0xB5, 0x73, 0xF1, 0x92, 0x28, 0xB2, 0xBF, 0x90, 0x5A, 0x76, 0x77, 0xFC, 0xA6, 0xB3, 0x21, 0x90, 0xDA, 0x6F, 0xB5, 0xCF, 0x38};
    int ff[28*28]={0};

    ff[0]=3;
    ff[1]=5;
    for (int j=2;j<=28*28;++j){
    ff[j]=2*ff[j-1]+3*ff[j-2];    
    }

    for(int i=0;i<=27;++i){
    //v3=flag[i];
    //v4=rec(i);
    //printf("%d\n",v4);
    putchar((flag[i]^ff[i*i]));
    fflush(stdout);
    }

    return 0;
}
```






