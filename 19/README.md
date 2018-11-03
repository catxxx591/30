中文文本分類練習
==

昨天做到要篩選停止詞，今天在網上找了停止詞字庫，還有找到繁體中文用語的字庫

<https://github.com/ldkrsi/jieba-zh_TW>

照著他的方法把停止詞加了進來

再來就繼續接續昨天的步驟，找出分類的高頻詞(今天又修了一些昨天的bug)
```python
lab_fwords={}
for labname in lab_tags.keys():
    print(labname,end=',')
    lab_words = lab_tags[labname]
    seg_list = jieba.cut(lab_words, cut_all=False)
    txt = ",".join(seg_list)
    tokens = txt.split(',')
    tokens = list(filter(lambda a: a not in stopwords and a not in filterwords, tokens))
    lab_fwords[labname] = nltk.FreqDist(tokens).most_common(2000)

即時,生活,娛樂,運動,政治,科技,國際,專題,兩岸,產經,社會,地方,證券,文化,
```

先印出來大致用直覺判斷正確性
```python
>>>　print(lab_fwords['娛樂'])
[('電影', 115),
 ('台灣', 77),
 ('年', 71),
 ('人', 59),
 ('後', 58),
 ('月', 56),
 ('今年', 54),
 ('屆', 39),
 ('作品', 37),
 ('導演', 36),
 ...
```
```python
>>> print(lab_fwords['科技'])
[('技術', 31),
 ('iPhone', 28),
 ('月', 20),
 ('年', 20),
 ('影像', 18),
 ('台灣', 17),
 ('研究', 17),
 ('科技', 15),
 ('管理局', 15),
 ('XS', 14),
 ('園區', 14),
 ...
```
```python
>>> print(lab_fwords['證券'])
[('點', 186),
 ('元', 134),
 ('月', 131),
 ('億元', 110),
 ('漲', 78),
 ('市場', 70),
 ('指數', 60),
 ('11', 53),
 ('每股', 51),
 ('台股', 50),
 ...
```

恩，我覺得可以


再來就是做特徵篩選器，先照著之前nltk教學判斷文章單字是否出現在高頻詞當中

特徵篩選器做出來後再建立特徵集，這裡我把這兩件事情定義再同一個函數當中。
```python
def get_featuresets(books):
    features = {}
    featuresets=[]
    for book in books:
        txt = book[0]
        labname = book[1]
        seg_list = jieba.cut(txt, cut_all=False)
        txt = ",".join(seg_list)
        tokens = txt.split(',')
        for tok in tokens:
            features[tok] = tok in [i[0] for i in lab_fwords[labname]]
        featuresets.append((features,labname))
    random.shuffle(featuresets)
    return featuresets
```
後來發現沒有獨立定義特徵提取器，想測試新資料會無法提取特徵，所以不能這樣做


有特徵集就可以跑模型了
```python
>>> featuresets = get_featuresets(books)
>>> train_set, test_set = featuresets[:840], featuresets[840:]
>>> classifier = nltk.NaiveBayesClassifier.train(train_set)
>>> nltk.classify.accuracy(classifier, test_set)

0.25146198830409355
```

千辛萬苦做出來的精確度只有25%，不過我放的類別有14種，最差的精度是1/14=7%，所以還是有功勞啦o_O"

我覺得可能在"即時"和"專題"這兩個類別上識別度不高。即時也可以包含運動比賽的最新消息或娛樂圈藝人突然的病危消息等等，專題也是

希望明天調整後精確度能超過70%...

參考資料:

Python 自然语言处理 第二版<https://usyiyi.github.io/nlp-py-2e-zh/>

結巴中文斷詞台灣繁體版本<https://github.com/ldkrsi/jieba-zh_TW>
