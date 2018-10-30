文檔分類練習
==
## 1. 使用本章討論過的電影評論文檔分類器，產生對分類器最有信息量的30個特徵的列表。你能解釋為什麼這些特定特徵具有信息量嗎？你能在它們中找到什麼驚人的發現嗎？
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
