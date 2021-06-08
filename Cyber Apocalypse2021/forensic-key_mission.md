CHTB2021-key_mission
===
## Info
**題目敘述**

The secretary of earth defense has been kidnapped. We have sent out elite team on the enemy’s base to find his location. Our team only managed to intercept this traffic. You mission is to retrieve secretary’s hidden location

**能力**
- wireshark
- tshark
- python
- google it!

**難易度**
★★☆☆☆

###### tags: `CTF-WRITEUP`

## Writeup
### 封包解析
題目是pcap檔案，用wireshark打開會發現裡面有大量的USB封包
![](https://i.imgur.com/UXw1PVE.png)
參考網路上的[USB PCAP ANALYSIS](https://book.hacktricks.xyz/forensics/basic-forensic-methodology/pcap-inspection/usb-keyboard-pcap-analysis)

根據文章說明，USB封包大小是 40 Bytes，且最後 8 Bytes會記錄資料
最後8 Bytes記錄資料是有表依據，可以查詢的[USB scancode](https://gist.github.com/MightyPork/6da26e382a7ad91b5496ee55fdc73db2)
- $1^{st}$ byte  
    `02`開頭的代表有按`SHIFT`，小寫`a`+`SHIFT`->`A`
- $3{rd}$ byte  
    ![](https://i.imgur.com/4EVzy9P.png)


以下圖為例傳送的資料為`00 00 10 00 00 00 00 00`，代表`g`

![](https://i.imgur.com/yfNGjTU.png)

再看一個例子，傳送的資料為`02 00 30 00 00 00 00 00`，代表`]`+`SHIFT`->`}`

![](https://i.imgur.com/JdXAxds.png)

### 擷取資料

- Wireshark
    - `((usb.transfer_type == 0x01) && (frame.len == 72)) && !(usb.capdata == 00:00:00:00:00:00:00:00)`
    - 將資料匯出csv檔案
    - `cat leftdata | cut -d “,” -f 7 | cut -d “\”” -f 2 | grep -vE “Leftover Capture Data” > hexoutput.txt
`
- Tshark
    - `tshark -r usb1.pcapng -T fields -e usb.capdata > usbdata.txt`
    - `tshark -r key_mission.pcap -T fields -e usb.capdata | grep -E "^.{23}$" | grep -v 00:00:00:00:00:00:00:00 > data0.txt` (筆者用這個)
    
![](https://i.imgur.com/nGCm54c.png)


### sol.py
寫python程式做讀檔、字串處理和比對
```
KEY_CODES = {
    0x04:['a', 'A'],
    0x05:['b', 'B'],
    0x06:['c', 'C'],
    0x07:['d', 'D'],
    0x08:['e', 'E'],
    0x09:['f', 'F'],
    0x0A:['g', 'G'],
    0x0B:['h', 'H'],
    0x0C:['i', 'I'],
    0x0D:['j', 'J'],
    0x0E:['k', 'K'],
    0x0F:['l', 'L'],
    0x10:['m', 'M'],
    0x11:['n', 'N'],
    0x12:['o', 'O'],
    0x13:['p', 'P'],
    0x14:['q', 'Q'],
    0x15:['r', 'R'],
    0x16:['s', 'S'],
    0x17:['t', 'T'],
    0x18:['u', 'U'],
    0x19:['v', 'V'],
    0x1A:['w', 'W'],
    0x1B:['x', 'X'],
    0x1C:['y', 'Y'],
    0x1D:['z', 'Z'],
    0x1E:['1', '!'],
    0x1F:['2', '@'],
    0x20:['3', '#'],
    0x21:['4', '$'],
    0x22:['5', '%'],
    0x23:['6', '^'],
    0x24:['7', '&'],
    0x25:['8', '*'],
    0x26:['9', '('],
    0x27:['0', ')'],
    0x28:['\n','\n'],
    0x2C:[' ', ' '],
    0x2D:['-', '_'],
    0x2E:['=', '+'],
    0x2F:['[', '{'],
    0x30:[']', '}'],
    0x32:['#','~'],
    0x33:[';', ':'],
    0x34:['\'', '"'],
    0x36:[',', '<'],
    0x38:['/', '?'],
    0x37:['.', '>'],
    0x2b:['\t','\t']
}

datas = open('data.txt').read().split('\n')[:-1]
flag=''
for i in datas:
    data = i.split(':')
    if data[0]=='02':
        try:
            idx= int(data[2],16)
            #print (KEY_CODES[idx][1])
            flag+=KEY_CODES[idx][1]
        except:
            continue
    else:
        try:
            idx=int(data[2],16)
            flag+=KEY_CODES[idx][0]
        except:
            continue

print (flag)
        #flag=flag+KEY_CODES[int(data[2],16)][1] 

```

![](https://i.imgur.com/Noel6Pn.png)






