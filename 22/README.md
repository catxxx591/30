今天從資料整理和特徵提取優化嘗試提高精確度

首先是資料整理，新聞網站的原文大部分都長這樣
```
（中央社北京6日綜合外電報導）微軟創辦人比爾．蓋茲今天在北京介紹一種不必用到水，也不需要下水道的未來風格廁所，只用化學品就可將人類排泄物變成肥料。

比爾．蓋茲（Bill Gates）是全球最活躍慈善家，也是其中一位最富裕人士。他手上的事經常多到忙不完，但是手裡拿著人類排泄物，還真是難得一見。

因此，當他手裡握著一罐人類排泄物，現身北京舉行的未來廁所論壇時，現場人士無不為之一驚。

來這一招是想要引起外界關注全球開發中國家廁所不足的問題。

比爾．蓋茲說：「在沒有衛生設施的地方，你解放的辦法不只那樣。」他說的是擺在桌上那個裝有排泄物的透明罐。

他告訴與會者：「但那正是孩童玩樂時，老是得去接觸到的東西。這不只跟生活品質有關，也跟疾病、死亡和營養不良有關。」

這位億萬富豪已將可觀財富其中一部分，投注在全球近半欠缺乾淨、舒適衛生設施人口的身上。

比爾．蓋茲說：「當你們思考跟衛生、夠吃一樣基本的東西時，你會想到，清單上當然也要包括有一間還過得去的廁所。」

路透社報導，比爾．蓋茲表示，這種廁所是全球最大私人慈善組織比爾暨梅琳達蓋茲基金會（Bill and Melinda Gates Foundation）資助的研究計畫，經多年研發後的創意結晶，擁有多款設計，且全都是以分離液態和固態排泄物的方式進行處理。（譯者：葉俐緯/核稿：何宏儒）1071106
```

開頭與結尾都有與內文無關的文字，但文章累積起來這些字常常成為高頻字。

記者姓名可能會與報導分類有關聯，但只要使用其他新聞網站的新聞，該記者名字將不會出現，所以將使模型的精確度被高估。

所以今天想利用re給他去頭去尾，前幾天就有做過嘗試不過都不算成功

問題就在於所有的文章格式並非統一，頭尾長度也不同。使用標點符號判斷也經常碰到例外狀況，圖片的註解文字也會影響，甚至有一個網址有兩篇新聞等等

今天花了不少時間終於試出滿意的結果

```python
txt = re.search(r'[\）\)\：\:\》](.+[\。\!\?\s])+',txt).group()[1:]
```

這裡分為兩部分

1. [\）\)\：\:\》]
2. (.+[\。\!\?\s])+',txt)

1.尋找內文開頭的位置，以這網站的格式這樣能正確判斷所有內文

2.的部分.+表示所有連續的字元，接在後面的[\。\!\?\s]就像之前nltk分析斷句的句尾符號。將這兩個湊為一組反覆執行就能一直讀到內文的最後一個字元

使用整理過的資料再去跑模型精確度並沒有上升

不過在測試的文章

```
年初大作《魔物獵人：世界》獲得各國玩家一致好評，在本月份PC版上市後不到一個月，銷量正式的突破了一千萬套，成違現世代在全球賣的最好的日本遊戲，超越了過去在歐美火紅的《潛龍諜影》和《Final Fantasy》等。

在PC版上市之前，《魔物獵人：世界》就已經是卡普空創社以來銷量最好的作品，連帶帶動開發商卡普空的日經股價從今年2月的2000點一路上揚到3000點。依據八月的數字來看，PS4和XBOX ONE版本的《魔物獵人：世界》已經突破830萬套，而Steam版本再開賣不到一週就衝出了200萬套的佳績，讓《魔物獵人：世界》的銷量正式來到千萬大關。在歐美遊戲媒體上GameSpot評分為8/10，IGN評分為9.5 / 10。儼然成了日本近年來走向世界的代表作品。

日前騰訊平台Wegame在日前慘遭營運五天就強迫下架的命運，不過原先在上市前騰訊還曾發出公告表示本作光是預購就突破了百萬，可見若非遭到下架，《魔物獵人：世界》的銷量還可以衝更高。不過由於前日大陸傳出凍結審批的消息，也讓卡普空的股價從3000元的高點開始重挫到2400左右。

```

分類卻發生了改變，從科技變成產經。原本模型就是有20%會錯誤，不過看這內容也覺得說是產經好像也行

當一個文章的特徵與分類的特徵出現這種重疊模糊的關係時，答案就較難判斷

所以我就試著調整特徵提取器試著解決這種情況

```python
def pos_features2(article,top):
    features={}
    for labname in list(lab_fwords.keys()):

        t = ''
        seg_list = jieba.cut(article, cut_all=False)
        t = ",".join(seg_list)
        article_tokens = t.split(',')
        article_tokens = [w for w in article_tokens if w.isalpha()]

        filt_tokens = list(filter(lambda a: a not in stopwords and a not in filterwords , article_tokens))#and len(a)>1, article_tokens))
        ###
        twowords = list(nltk.bigrams(article_tokens))
        two = nltk.FreqDist(twowords).most_common(int(top*2/3))
        Ftwowords = [i[0]+i[1] for i,j in two]
        for two in Ftwowords:
            features[two+'-'+labname] = two in lab_Ftwowords[labname]#All_Ftwowords
        ###
        toks = [w[0] for w in nltk.FreqDist(filt_tokens).most_common(top)]
    #   print(toks)
        for tok in toks:
                features[tok+'-'+labname] = tok in [i[0] for i in lab_fwords[labname]]
    #        features[tok] = True
#        print(labname)
    return features
```
調整的提取器我把單字一次跑過所有分類，並分別記下特徵。以下我先拿前三高頻字做為提取的特徵
```python
print(pos_features2(test_news,3))

{'魔物獵人-生活': False,
 '獵人世界-生活': False,
 '魔物-生活': False,
 '獵人-生活': False,
 '世界-生活': False,
 '魔物獵人-地方': False,
 '獵人世界-地方': False,
 '魔物-地方': False,
 '獵人-地方': False,
 '世界-地方': False,
 '魔物獵人-證券': False,
 '獵人世界-證券': False,
 '魔物-證券': False,
 '獵人-證券': False,
 '世界-證券': False,
 '魔物獵人-科技': False,
 '獵人世界-科技': False,
 '魔物-科技': False,
 '獵人-科技': False,
 '世界-科技': False,
 '魔物獵人-兩岸': False,
 '獵人世界-兩岸': False,
 '魔物-兩岸': False,
 '獵人-兩岸': False,
 '世界-兩岸': True,
 '魔物獵人-娛樂': False,
 '獵人世界-娛樂': False,
 '魔物-娛樂': False,
 '獵人-娛樂': False,
 '世界-娛樂': True,
 '魔物獵人-運動': False,
 '獵人世界-運動': False,
 '魔物-運動': False,
 '獵人-運動': False,
 '世界-運動': True,
 '魔物獵人-社會': False,
 '獵人世界-社會': False,
...
```

每一個高頻字都會在每個分類提取一次特徵，並且在製作特徵集時附上正確的標籤，所想這樣或許能解決抹些分類較模糊的問題

我嘗試了用前100高頻字建立模型，時間非常久阿...

![](https://github.com/catxxx591/30/blob/master/img/memory_crash.PNG?raw=true)

