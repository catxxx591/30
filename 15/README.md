文檔分類練習
==
### 1. 使用本章討論過的電影評論文檔分類器，產生對分類器最有信息量的30個特徵的列表。你能解釋為什麼這些特定特徵具有信息量嗎？你能在它們中找到什麼驚人的發現嗎？
```python
import nltk
from nltk.corpus import movie_reviews
import random
documents = [(list(movie_reviews.words(fileid)), category)
             for category in movie_reviews.categories()
             for fileid in movie_reviews.fileids(category )]
random.shuffle(documents)
all_words = nltk.FreqDist(w.lower() for w in movie_reviews.words())
word_features = list(all_words)[:2000] 

def document_features (document): 
    document_words = set(document)
    features = {}
    for word in word_features:
        features[ 'contains({})' .format(word)] = (word in document_words)
    return features
featuresets = [(document_features(d), c) for (d,c) in documents]
train_set, test_set = featuresets[100:], featuresets[:100]
classifier = nltk.NaiveBayesClassifier.train(train_set)
>>> [re.search(r'\(\w+',w[0]).group()[1:] 
... for w in classifier.most_informative_features(30)]

['unimaginative',
 'schumacher',
 'atrocious',
 'suvari',
 'mena',
 'shoddy',
 'turkey',
...]
```
特徵集中於某標籤上

正負評價標籤各占一半
```python
>>> [f[1] for f in featuresets].count('neg')/len(featuresets)
0.5
```
查看前100信息量的特徵，在正負評價標籤數量相同條件下，依特徵辨識出的標籤多是負面評價，負面評價的特徵較為明顯
```python
>>> classifier.show_most_informative_features(100)
"""
Most Informative Features
 contains(unimaginative) = True              neg : pos    =      7.7 : 1.0
    contains(schumacher) = True              neg : pos    =      7.5 : 1.0
        contains(suvari) = True              neg : pos    =      7.1 : 1.0
          contains(mena) = True              neg : pos    =      7.1 : 1.0
        contains(shoddy) = True              neg : pos    =      7.1 : 1.0
     contains(atrocious) = True              neg : pos    =      6.7 : 1.0
        contains(turkey) = True              neg : pos    =      6.6 : 1.0
        contains(wasted) = True              neg : pos    =      5.9 : 1.0
  contains(surveillance) = True              neg : pos    =      5.7 : 1.0
        contains(canyon) = True              neg : pos    =      5.7 : 1.0
       contains(unravel) = True              pos : neg    =      5.6 : 1.0
           contains(ugh) = True              neg : pos    =      5.5 : 1.0
        contains(justin) = True              neg : pos    =      5.5 : 1.0
         contains(bland) = True              neg : pos    =      5.3 : 1.0
         contains(waste) = True              neg : pos    =      5.2 : 1.0
          contains(oops) = True              neg : pos    =      5.1 : 1.0
         contains(awful) = True              neg : pos    =      5.1 : 1.0
       contains(bronson) = True              neg : pos    =      5.1 : 1.0
     contains(underwood) = True              neg : pos    =      5.1 : 1.0
       contains(miscast) = True              neg : pos    =      4.8 : 1.0
    contains(ridiculous) = True              neg : pos    =      4.8 : 1.0
        contains(poorly) = True              neg : pos    =      4.7 : 1.0
    contains(uninspired) = True              neg : pos    =      4.6 : 1.0
     contains(painfully) = True              neg : pos    =      4.6 : 1.0
      contains(explores) = True              pos : neg    =      4.5 : 1.0
"""
```

### 2. 選擇一個本章所描述的分類任務，如名字性別檢測、文檔分類、詞性標註或對話行為分類。使用相同的訓練和測試數據，相同的特徵提取器，建立該任務的三個分類器：：決策樹、樸素貝葉斯分類器和最大熵分類器。比較你所選任務上這三個分類器的準確性。你如何看待如果你使用了不同的特徵提取器，你的結果可能會不同？

```python
>>> classifier = nltk.DecisionTreeClassifier.train(train_set)
>>> nltk.classify.accuracy(classifier, test_set)
0.7695673794132273

>>> classifier = nltk.NaiveBayesClassifier.train(train_set)
>>> nltk.classify.accuracy(classifier, test_set)
0.8166086524117354

>>> classifier = nltk.MaxentClassifier.train(train_set)
>>> nltk.classify.accuracy(classifier, test_set)
#跑太久，放棄
```

參考資料:Python 自然语言处理 第二版<https://usyiyi.github.io/nlp-py-2e-zh/>
