詞性標註
==
我們使用Brown語料庫找出當中文本單字字尾的出現頻率
(Brown語料庫是nltk中的內建語料庫，特色是所有文本依照主題分類)

定義一個fdist之後將brown語料庫中單字的末三個字母組合統計起來

suffix_fdist = nltk.FreqDist()步驟可能是為了後面使用.most_common()
```python
>>> from nltk.corpus import brown
>>> suffix_fdist = nltk.FreqDist()
>>> for word in brown.words():
...     word = word.lower()  
...     suffix_fdist[word[-1 :]] += 1
...     suffix_fdist[word[-2:]] += 1
...     suffix_fdist[word[-3:]] += 1
>>> print(suffix_fdist)

FreqDist({'e': 202946, ',': 175002, '.': 152999, 's': 128722, 'd': 105687, 't': 94459, 'he': 92084, 'n': 87889, 'a': 74912, 'of': 72978, ...})
```

接著使用.most_common()只挑出頻率最高的100個字尾組合
```python
>>> common_suffixes = [suffix for (suffix, count) in suffix_fdist.most_common(100)]
>>> print (common_suffixes)

['e', ',', '.', 's', 'd', 't', 'he', 'n', 'a', 'of', 'the', 'y', 'r', 'to', 'in', 'f', 'o', 'ed', 'nd', 'is', 'on', 'l', 'g', 'and', 'ng', 'er', 'as', 'ing', 'h', 'at', 'es', 'or', 're', 'it', '``', 'an', "''", 'm', ';', 'i', 'ly', 'ion', 'en', 'al', '?', 'nt', 'be', 'hat', 'st', 'his', 'th', 'll', 'le', 'ce', 'by', 'ts', 'me', 've', "'", 'se', 'ut', 'was', 'for', 'ent', 'ch', 'k', 'w', 'ld', '`', 'rs', 'ted', 'ere', 'her', 'ne', 'ns', 'ith', 'ad', 'ry', ')', '(', 'te', '--', 'ay', 'ty', 'ot', 'p', 'nce', "'s", 'ter', 'om', 'ss', ':', 'we', 'are', 'c', 'ers', 'uld', 'had', 'so', 'ey']
```

接著用剛做好的common_suffixes定義一個特徵提取器函數，檢查給定的單詞的這些後綴：

```python
>>> def  pos_features (word):
...     features = {}
...     for suffix in common_suffixes:
...         features[ 'endswith({})' .format(suffix)] = word.lower().endswith (suffix)
...     return features
```

我們使用brown.tagged_words()整理出brown語料庫中指定類別的單詞並附上詞性標籤做成特徵集
```python
>>> tagged_words = brown.tagged_words(categories= 'news' )
>>> featuresets = [(pos_features(n), g) for (n,g) in tagged_words]
```
將特徵集以9:1比例分為訓練及測試集
```python
>>> size = int(len(featuresets) * 0.1)
>>> train_set, test_set = featuresets[size:], featuresets[:size]
```

使用決策樹模型分析特徵集，並查看精度
```python
>>> classifier = nltk.DecisionTreeClassifier.train(train_set)
>>> nltk.classify.accuracy(classifier, test_set)
0.6270512182993535
```
最後可將新資料帶入剛完成的決策樹模型。此時得到的結果是經由模型計算得出，而非經由特徵集查詢。
```python
>>> classifier.classify(pos_features( 'yee' ))
'NN'
```

我們可以已偽代碼的形式查看決策樹的分支，觀察模型的決策過程
```python
>>> print (classifier.pseudocode(depth=5))

if endswith(the) == False: 
  if endswith(,) == False: 
    if endswith(s) == False: 
      if endswith(.) == False: 
        if endswith(of) == False: return '.'
        if endswith(of) == True: return 'IN'
      if endswith(.) == True: return '.'
    if endswith(s) == True: 
      if endswith(is) == False: 
        if endswith(was) == False: return 'PP$'
        if endswith(was) == True: return 'BEDZ'
      if endswith(is) == True: 
        if endswith(his) == False: return 'BEZ'
        if endswith(his) == True: return 'PP$'
  if endswith(,) == True: return ','
if endswith(the) == True: return 'AT'
```
