中文文本分類練習
==
今天先做了這幾件事，調整精確度

1. 把即時和專題兩個類別拿掉，發現問題不在這裡，因為精確度更低

2. 把爬蟲過濾的內文re拿掉，re功力不夠，迴圈出現的例外狀況太多，有時候只有斷續的內文。發現精確度還是低

3. 調整停止詞和過濾的單詞，有些高頻字沒列進停止詞中卻散布在各類別，對分類辨識度沒有幫助。發現精確度還是低

4. 調整特徵從判定全文單字的出現與否，改成只判定高頻字。還是不行

5. 最後發現在做特徵提取迴圈有bug，讓不同類別文章的都特徵都寫進一篇文章的特徵中，所以特徵不對自然分類錯誤

改正之後精確度暴衝至96%。經驗告訴我這麼高一定有錯

結果在特徵提取的部分找到邏輯的問題
```python
for tok in toks:
    features[tok] = tok in [i[0] for i in lab_fwords[labname]]
```
在這裡如果我要提取特徵，我就必需先知道這篇文章正確的類別才行，等於是我先告訴他答案他再跟我講答案還說正確率不是100%，但做機器學習的目的就是讓電腦判定未知資料的類別

所以把這段改成以下，意思是這些字有出現
```python
for tok in toks:
    features[tok] = True
```

## 所以模型學習的內容就是:文章出現的高頻字是否吻合某類別的高頻字

特徵提取器
```python
def pos_features(article,top):
    features={}
    t = ''
    seg_list = jieba.cut(article, cut_all=False)
    t = ",".join(seg_list)
    article_tokens = t.split(',')
    filt_tokens = list(filter(lambda a: a not in stopwords and a not in filterwords , article_tokens))#and len(a)>1, article_tokens))
    toks = [w[0] for w in nltk.FreqDist(filt_tokens).most_common(top)]
#   print(toks)
    for tok in toks:
#            features[tok] = tok in [i[0] for i in lab_fwords[labname]]
        features[tok] = True
    return features
```

利用特徵提取器製作特徵集，這裡我做了一個數字參數top，功能是調整要以前幾名的高頻詞做為特徵
```python
def get_featuresets(books,top=200):

    featuresets=[]
    for book in books:
#        features = {}
        article = book[0]
        labname = book[1]
        features = pos_features(article,top)
        feature_item = (features,labname)
        featuresets.append(feature_item)

        print(labname)

    return featuresets
```
得到特徵集
```python
featuresets = get_featuresets(books,200)
```

最後建立模型回傳精確度
```python
size = int(len(featuresets)*0.7)
train_set, test_set = featuresets[:size], featuresets[size:]
classifier = nltk.NaiveBayesClassifier.train(train_set)
print(nltk.classify.accuracy(classifier, test_set))

0.8388888888888889
```

## 精確度有83.9%!!!

覺得在特徵提取的部分還有進步空間

接著可以拿其他資料測試，這裡我就用手動複製貼上

```python
test_news="""

Apple 秋季給我們帶來了一波新產品，召開的新品電腦發布會發布的新款產品更是讓我們不知所云。一開始 Tim Cook 在台上給我們帶來全新的 MacBook Air 產品。對於該產品，早在3月份的教育發布會上被公佈了出來，直到現在才與我們見面。


新款 MacBook Air 發布會上發讓不少人感到了失望，因為在它身上看不到多少的變化，並且定位尷尬。首先就是處理器性能差，官方稱配備全新第八代 intel Core i5處理器。但是這一顆處理器僅有1.6GHz的基礎頻率，並且處理器核心只有兩個。在 Intel 官方網站上得知型號為 i5-8210Y，集成了 Intel UHD Graphics 617 圖像顯示卡，功耗上僅有 7W。這個處理器與目前市場上的四核八線程的第八代 Core 低壓處理器性能有很大的差距。

 


第二點是外觀無亮點，甚至沒有什麼變化，外觀與不帶 Touch Bar 版本的 MacBook Pro 沒有什麼區別。厚度雖然比舊款縮減了10%，但是實際最厚處與現款在售的 13吋 MacBook Pro 差不了多少。重量上與 Macbook Pro 亦只差 100g 左右。

在插口上，擁有兩個 Thunderbolt 3 接口和新加的Touch ID，仔細觀察會發現全新的 MacBook Air 可以說是目前 12吋 MacBook 與 13吋 MacBook Pro 的設計結合。

 


第三就是售價過於貴，原本以為定位在教育的產品，新款 MacBook Air 售價會「低廉」一些。但是新款 MacBook Air 卻高達 HK$9,499。而有趣的地方在於舊款模具的 MacBook Air 仍然在蘋果官方商店內銷售，售價卻僅為HK$7,488，成為了目前最便宜的 MacBook，這樣才是面向教育的定價。

說了新款 MacBook 的不好之處，那該如何選購 MacBook？


新款 MacBook Air 處在 12吋 MacBook 和 MacBook Pro 之間，便攜上沒有 12吋 MacBook 輕薄，性能僅僅比 12吋 MacBook 多出來一個 Thunderbolt 3 接口。性能遠沒有 MacBook Pro 好。所以，想要超便攜的筆記本電腦，可以考慮 12吋 的MacBook，如果想要性能與便攜性兼備可以考慮不帶 Touch Bar 的 MacBook Pro。雖是舊款，但性能方面仍在新款 MacBook Air 之上，價格亦只需 HK$9,988。

 


對於新版筆記本電腦，全面屏 iPad Pro 反而更具有考慮入手的價值，因為它換用了 USB Type-C 接口，並且為全面屏，還使用了 7nm 製程的 A12X 處理器，如今的 iPad Pro 其實已經可以取代一些輕薄筆記本電腦的地位，所以想要超輕薄電腦的朋友還不如考慮新款的 iPad Pro。

對於最新的 7nm 製程的 A12X 處理器，很有可能會被塞入到那款沒有更新的 12吋 MacBook 中，所以還是得進行一番等待的。

為您推薦更多相關文章：

新 MacBook 系列全面加入 T2 保安晶片，自動中斷收音咪防竊聽或監聽！


Apple 沒有放棄，2018 版全新 MacBook Air 發佈！

"""
>>> print(classifier.classify(pos_features(test_news,300)))

科技
```

成功的判斷出是科技類的新聞

不過這個模型還是有基本上的限制

1. 只能辨識訓練過的類別，我訓練的類別並沒有"軍事"，如果給他軍事新聞可能會回傳"兩岸"或"國際"吧??

2. 學習資料只有中央社新聞，在判別其他家新聞的類別時精確度大概不剩80%。例如，中央社的娛樂新聞電競內容不多，判斷電競新聞可能出現其他答案

不過我想多學幾家網站文章一定會變強的

等我整理好一點再貼github的code，雖然沒人想要owO

參考資料: 

Python 自然语言处理 第二版<https://usyiyi.github.io/nlp-py-2e-zh/>

<http://enginebai.logdown.com/posts/241676/knn>

<http://mropengate.blogspot.com/2015/06/ai-ch14-3-naive-bayes-classifier.html>
