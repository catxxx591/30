練習分類文本
==

### 詞特徵在處理文本分類中是非常有用的，因為在一個文檔中出現的詞對於其語義內容是什麼具有強烈的指示作用。然而，很多詞很少出現，一些在文檔中的最有信息量的詞可能永遠不會出現在我們的訓練數據中。一種解決方法是使用一個詞典，它描述了詞之間的不同。使用WordNet詞典，加強本章介紹的電影評論文檔分類器，使用概括一個文檔中出現的詞的特徵，使之更容易匹配在訓練數據中發現的詞。

前幾天電影評論的分類，特徵篩選是判斷高頻字在類型的文本分布情形，準確率將近80%
```python
>>> from nltk.corpus import movie_reviews
>>> import random
>>> documents = [(list(movie_reviews.words(fileid)), category)
...              for category in movie_reviews.categories()
...              for fileid in movie_reviews.fileids(category )]
>>> random.shuffle(documents)

>>> all_words = nltk.FreqDist(w.lower() for w in movie_reviews.words())
>>> word_features = list(all_words)[:2000] 

>>> def document_features (document): 
...     document_words = set(document)
...     features = {}
...     for word in word_features:
...         features[ 'contains({})' .format(word)] = (word in document_words)
...     return features

>>> featuresets = [(document_features(d), c) for (d,c) in documents]
>>> train_set, test_set = featuresets[:1400], featuresets[1400:]
>>> classifier = nltk.NaiveBayesClassifier.train(train_set)
>>> nltk.classify.accuracy(classifier, test_set)

0.7966666666666666
```

題目說使用wordnet辭典，載入電影評論語料庫

```python
import nltk
from nltk.corpus import movie_reviews
import random
import re
from nltk.corpus import wordnet as wn

>>> documents = [(list(movie_reviews.words(fileid)), category)
...              for category in movie_reviews.categories()
...              for fileid in movie_reviews.fileids(category )]
>>> random.shuffle(documents)
```

先把最後完成的特徵提取器放上來，兩行###之中是新加入使用wordnet的部分
```python
def document_features (document): 
    document_words = set(document)
    features = {}
    
    for word in word_features:
        features[ 'contains({})' .format(word)] = (word in document_words)
    ###
    path = []
    syns = [wn.synsets(w)[0] for w in [w[0] for w in all_words.most_common(5000)][2000:] if len(wn.synsets(w))>0]
    

    for syn in syns:

        word = syn.lemmas()[0].name()
        pos_value = pos.path_similarity(syn)
        neg_value = neg.path_similarity(syn)
        if pos_value and neg_value != None:
            path.append((word,max(pos_value,neg_value)))
        path = [i for i in sorted(path,key=lambda d:d[1],reverse=True) if  i[1]>=0.25]
        
        for i in path:
            if i[0] not in document_words:
                features[i[0]] = True
    ###
    return features
featuresets = [(document_features(d), c) for (d,c) in documents]
```

- wordnet詞條

wordnet.synsets(word)可以列出word單字的同意字詞條，並且可由詞條查取單字定義極例句

把語料庫高頻字先轉換成詞條
```python
>>> all_words = nltk.FreqDist(w.lower() for w in movie_reviews.words())
>>> word_features = [w[0] for w in all_words.most_common(2000)]
>>> syns = [wn.synsets(w)[0] for w in [w[0] for w in all_words.most_common(5000)][2000:] if len(wn.synsets(w))>0]

[Synset('serial.n.01'),
 Synset('value.n.01'),
 Synset('miss.v.01'),
 Synset('oliver.n.01'),
 Synset('hollow.n.01'),
 Synset('sea.n.01'),
 Synset('animal.n.01'),
 Synset('freeman.n.01'),
 Synset('animal.n.01'),
 Synset('crystal.n.01'),
 ...
```

在來使用到wordnet的關聯詞功能，我要找出剛剛找出的高頻字與分類標籤的關聯性，首先要把標籤名稱也轉成詞條
```python
pos = wn.synsets('positive')[0]
neg = wn.synsets('negative')[0]
```
接著用.path_similarity()，他可以找出兩詞條在wordnet資料庫內的最相近的標籤並量化為0~1間的數字。使用這個找出高頻字與標籤的關聯強度
```python
>>>     for syn in syns:
...         word = syn.lemmas()[0].name()
...         pos_value = pos.path_similarity(syn)
...         neg_value = neg.path_similarity(syn)
```

每個字會得到兩個分數分別是pos和neg，我只挑出關聯強度較高的數字(分數較高)，並訂一個門檻值，當強度分數過門檻就加進特徵集當中，這裡門檻值是i[1]>=0.25
```python
>>>         if pos_value and neg_value != None:
...             path.append((word,max(pos_value,neg_value)))
>>>         path = [i for i in sorted(path,key=lambda d:d[1],reverse=True) if  i[1]>=0.25]
        
>>>         for i in path:
...             if i[0] not in document_words:
...                 features[i[0]] = True
```

使用新的特徵集建立模型並回傳精確度
```python
>>> train_set, test_set = featuresets[:1500], featuresets[1500:]
>>> classifier = nltk.NaiveBayesClassifier.train(train_set)
>>> nltk.classify.accuracy(classifier, test_set)

0.802
```

成功的讓精確度由0.797生到0.802提高了0.5%。

調整高頻字的範圍或關聯強度的門檻值都可以影響結果，我調整最高只有0.808，增加1.1%。好希望能90%。

增加樣本也許有更高的精確度，每次調整都得重建模型，這個大小已經讓我跑了半小時左右

感覺增加的幅度實在無感，一定有更好的做法

參考資料:Python 自然语言处理 第二版<https://usyiyi.github.io/nlp-py-2e-zh/>
