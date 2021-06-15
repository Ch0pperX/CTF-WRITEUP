Circle City Con CTF 2021 - Happy little Osint
===
## Info
**題目敘述**
Part 1 of 3 in the Happy Lil Osint Series

Someone is bullying a fan account of the happy little con! We have managed to get some information about the accounts in question.

--Scope--
- The account being bullied is mutuals with Circle City Con, and they are only following Circle City Con.
- The bullying account only has 3 tweets which are all from the past 5 days.
- The tags of both accounts are related to circle city con, and its theme!

Flag format is CCC{...} and begins with `wh` (just so things dont get out of order)

**能力**
- 社群網站基本常識
- python 
- 基本爬蟲觀念

**難易度**
★★★★☆

###### tags: `CTF-WRITEUP`
## Write Up
從**HappyLittleCon**帳號去關聯

![](https://i.imgur.com/vJpjNzg.png)

目的是找2個Twitter帳號，條件如下：
1. 受害者：和@CircleCityCon互相追蹤，受害者只追蹤@CircleCityCon。
2. 加害者：總共只有三篇推文，發布日期都是五天內。
3. tags和circle city con有關聯

因為跟隨中和跟隨者的數量太過龐大，故利用Twitter爬蟲先過濾符合`條件1`的帳號
```python
for user in tweepy.Cursor(api.friends, screen_name="CircleCityCon").items():
    print(user.screen_name, file=f)
    if (user.friends_count==1):
        print(user.screen_name)
```
可以找到以下結果：
```
happy_lil_con
WhereIsBiggles
...
Rate limit reached. Sleeping for: 855
...
Rate limit reached. Sleeping for: 855
...
Rate limit reached. Sleeping for: 855
...

```
其中**happy_lil_con**符合`條件1`和`條件3`

![](https://i.imgur.com/qGO2RSl.png)

看3個追隨者相關資訊繼續關聯...就會看到flag

![](https://i.imgur.com/nDk8SXK.png)

### sol.py
```
import tweepy

# assign the values accordingly
consumer_key = "pecl6T0grjVjNDJa3JmB41n3F"
consumer_secret = "lMakJFG8Rb7ZSSrMwh06Z5KidIRvfKn203drOLvdO0yjvajB3Q"
access_token = "1403927521712766982-l8Dkpyxk4q7tjKZDIpBILVeIZc47YH"
access_token_secret = "3Kvk5PhVKVAZ9d0eHPg6aC9zQnZYuZfMHs2e1iqNvbtRa"

# authorization of consumer key and consumer secret
auth = tweepy.OAuthHandler(consumer_key, consumer_secret)

# set access to user's access key and access secret 
auth.set_access_token(access_token, access_token_secret)

# calling the api 
api = tweepy.API(auth, wait_on_rate_limit=True, wait_on_rate_limit_notify=True)
#user = api.get_user(1434647058)
#followers_count = user.followers_count
#print(followers_count)

with open('following.txt', 'a') as f:    
    for user in tweepy.Cursor(api.friends, screen_name="CircleCityCon").items():
        print(user.screen_name, file=f)
        if (user.friends_count==1):
            print(user.screen_name)

print("end")
```
