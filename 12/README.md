探索上下文語境
==
如果特徵提取僅僅看著目標詞，我們就沒法添加依賴詞出現的上下文語境特徵。然而上下文語境特徵往往提供關於正確標記的強大線索——例如，標註詞"fly"，如果知道它前面的詞是“a”將使我們能夠確定它是一個名詞，而不是一個動詞。

書上定義了一個以句子為參數的特徵提取器
```python
def pos_features (sentence, i): 
    features = { "suffix(1)" : sentence[i][-1:],
                 "suffix(2)" : sentence[i][-2:],
                 "suffix(3) " : sentence[i][-3:]}
    if i == 0:
        features[ "prev-word" ] = "<START>" 
    else :
        features[ "prev-word" ] = sentence[i-1]
    return features
```

接著將昨天的Brown文本放進去，而昨天的代碼是無法使用的
```python
>>> tagged_words = brown.tagged_words(categories= 'news' )
>>> featuresets = [(pos_features(n), g) for (n,g) in tagged_words]
```

昨天文本還是以單字進行分割，而新的特徵其取器參數必須為句子
```python
>>> tagged_sents = brown.tagged_sents(categories= 'news' )
>>> print(tagged_sents[0])

[('The', 'AT'),
 ('Fulton', 'NP-TL'),
 ('County', 'NN-TL'),
 ('Grand', 'JJ-TL'),
 ('Jury', 'NN-TL'),
 ('said', 'VBD'),
 ('Friday', 'NR'),
 ('an', 'AT'),
 ('investigation', 'NN'),
 ('of', 'IN'),
```

觀察tagged_sents中元素是以(單字,詞性)組成，我們依照此結構調整放入pos_features(sentence, i)中做出特徵集。

tagged_sent中單字都附有詞性標籤，在這裡多做一個sent將單字重新組成句子

特徵集的結構是(特徵,標籤)，也就是(pos_features,詞性)
```python
>>> featuresets = []
>>> for s in tagged_sents:
...     sent = []
...     for w in s:
...         sent.append(w[0])
...         featuresets.append((pos_features(sent,s.index(w)),w[1]))
```
看看我們的特徵集，由字尾末三碼組合和前面一個單字加上詞性標籤構成
```python
>>> featuresets[:3]
[({'suffix(1)': 'e',
   'suffix(2)': 'he',
   'suffix(3) ': 'The',
   'prev-word': '<START>'},
  'AT'),
 ({'suffix(1)': 'n',
   'suffix(2)': 'on',
   'suffix(3) ': 'ton',
   'prev-word': 'The'},
  'NP-TL'),
 ({'suffix(1)': 'y',
   'suffix(2)': 'ty',
   'suffix(3) ': 'nty',
   'prev-word': 'Fulton'},
  'NN-TL')]
```

之後就依照昨天的方法將特徵集分為訓練及測試集，並建立決策樹模型回傳精度
```python
>>> size = int(len(featuresets) * 0.1)
>>> train_set, test_set = featuresets[size:], featuresets[:size]
>>> classifier = nltk.DecisionTreeClassifier.train(train_set)
>>> nltk.classify.accuracy(classifier, test_set)

0.765589259075087
```

接著就試著輸入新資料進去執行看看，這裡pos_features(sentence, i)sentence是list型態，所以定義一個函數先靠re做字串轉換
```python
>>> def test_features(txt=str,i=int):
...     txt = re.findall(r'\S+',txt)
...     return classifier.classify(pos_features( txt, i))
```

最後輸入看看
```python
>>> test_features('it is a lazy dog',3)
'JJ'
```
這裡電腦正確回傳'lazy'的詞性是'JJ'形容詞

可以輸入這串代碼對照詞性代號說明
```python
>>> nltk.help.upenn_tagset()
"""
$: dollar
    $ -$ --$ A$ C$ HK$ M$ NZ$ S$ U.S.$ US$
'': closing quotation mark
    ' ''
(: opening parenthesis
    ( [ {
): closing parenthesis
    ) ] }
,: comma
    ,
--: dash
    --
.: sentence terminator
    . ! ?
 .........
"""
```
Part2
---

接著呢，書上提到在特徵提取器中加入前一個單字的詞性，若是句首就標註start。
    
在句子中如果前一個單字是形容詞，那麼在他後面可能會是名詞，加入這個邏輯特徵對照結果
 
### 這部分我太菜了，官方示範碼無法全部理解，不能理解的那部分就是完全看不懂
所以以下是依我對書本文章的理解，嘗試得到相同結果的代碼

將特徵提取器加入history參數，為前一個單字的標籤
```python
def pos_features (sentence, i, history): 
    features = { "suffix(1)" : sentence[i][-1:],
                  "suffix(2)" : sentence[i][-2:],
                  "suffix( 3)" : sentence[i][-3:]}
    if i == 0:
        features[ "prev-word" ] = "<START>" 
#        features[ "prev-tag" ] = "<START>" 
    else :
        features[ "prev-word" ] = sentence[i-1]
        features[ "prev-tag" ] = history#[i-1]
    return features
```

接著製作特徵集
```python
>>> featuresets = []
>>> for s in tagged_sents:
...     sent = []
...     history = '<start>'
...     for w in s:
...         sent.append(w[0])
...         featuresets.append((pos_features(sent,s.index(w),history),w[1]))
...         history=w[1]
>>> print(featuresets[:3])
"""
[({'suffix(1)': 'e',
   'suffix(2)': 'he',
   'suffix( 3)': 'The',
   'prev-word': '<START>'},
  'AT'),
 ({'suffix(1)': 'n',
   'suffix(2)': 'on',
   'suffix( 3)': 'ton',
   'prev-word': 'The',
   'prev-tag': 'AT'},
  'NP-TL'),
 ({'suffix(1)': 'y',
   'suffix(2)': 'ty',
   'suffix( 3)': 'nty',
   'prev-word': 'Fulton',
   'prev-tag': 'NP-TL'},
  'NN-TL')]
"""
```
接著就可以跑模型了
```python
>>> size = int(len(featuresets) * 0.1)
>>> train_set, test_set = featuresets[size:], featuresets[:size]
>>> classifier = nltk.DecisionTreeClassifier.train(train_set)
>>> nltk.classify.accuracy(classifier, test_set)
0.774639482844356
```

# 精度大約從76.5%增加到77.5%增加了1%!!!! = =

不知道官方示範碼跑幾%，我連怎麼用都不懂

參考資料: Python 自然語言處理第二版<https://usyiyi.github.io/nlp-py-2e-zh/>
