文檔分類
==
使用nltk庫中的電影評論語料庫，當中的文本已經被分類為neg和pos兩類，我們將文本的做成單字列表並附上類別的標籤定義為douments
```python
>>> from nltk.corpus import movie_reviews
>>> documents = [(list(movie_reviews.words(fileid)), category)
...              for category in movie_reviews.categories()
...              for fileid in movie_reviews.fileids(category )]
>>> random.shuffle(documents)
```

接著挑出這些文本頻率最高的2000字做為文本的特徵
```python
>>> all_words = nltk.FreqDist(w.lower() for w in movie_reviews.words())
>>> word_features = list(all_words)[:2000]
```

定義一個特徵提取器，簡單地檢查這些字是否在一個給定的文檔中。
```python
>>> def document_features (document): 
...     document_words = set(document)
...     features = {}
...     for word in word_features:
...         features[ 'contains({})' .format(word)] = (word in document_words)
...     return features
```
將一開始建立的douments單字列表部份加上特徵提取器document_features()做成特徵集，並分為訓練及測試集
```python
featuresets = [(document_features(d), c) for (d,c) in documents]
train_set, test_set = featuresets[100:], featuresets[:100]
classifier = nltk.NaiveBayesClassifier.train(train_set)
```
最後使用.show_most_informative_features()來找出哪些特徵是分類器發現最有信息量的。
```python
>>> classifier.show_most_informative_features()
Most Informative Features
        contains(welles) = True              neg : pos    =      8.4 : 1.0
    contains(schumacher) = True              neg : pos    =      7.5 : 1.0
          contains(mena) = True              neg : pos    =      7.1 : 1.0
 contains(unimaginative) = True              neg : pos    =      7.1 : 1.0
        contains(suvari) = True              neg : pos    =      7.1 : 1.0
        contains(shoddy) = True              neg : pos    =      6.4 : 1.0
       contains(jumbled) = True              neg : pos    =      6.4 : 1.0
        contains(neatly) = True              pos : neg    =      6.3 : 1.0
     contains(atrocious) = True              neg : pos    =      6.3 : 1.0
        contains(turkey) = True              neg : pos    =      6.2 : 1.0
```

