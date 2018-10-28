句子分割
==
句子分割可以看作是一個標點符號的分類任務：每當我們遇到一個可能會結束一個句子的符號，如句號或問號，我們必須決定它是否終止了當前句子。

第一步是獲得一些已被分割成句子的數據，將它轉換成一種適合提取特徵的形式：
```python
>>> sents = nltk.corpus.treebank_raw.sents()
>>> tokens = []
>>> boundaries = set()
>>> offset = 0
>>> for sent in sents:
...     tokens.extend( sent) 
...     offset += len(sent)
...     boundaries.add(offset-1)
```
在這裡，tokens是單獨句子標識符的合併列表，boundaries是一個包含所有句子邊界詞符索引的集合。

由於先在文本由原本的string變成由tokens構成的list，結構有點像是中文的作文稿紙一格一個字，boundaries就是記錄所有的句尾在第幾個格子

再來就製作特徵提取器，我們以標點符號本身及前後出現的單詞作為特徵並加上陣列索引加以區別
```python
>>> def  punct_features (tokens, i):
...     return { 'next-word-capitalized' : tokens[i+1][0].isupper(),
...             'prev-word' : tokens[i -1].lower(),
...             'punct' : tokens[i],
...             'prev-word-is-one-char' : len(tokens[i-1]) == 1}   #加以分辨文章中出現類似1.2.等符號
```

提取特徵後就可以製作特徵集來使用特徵判別標籤，特徵集結構是(特徵,標籤)，這裡我們標籤為標點符號的陣列索引是否在boundaries所記錄的句尾索引上
```python
>>> featuresets = [(punct_features(tokens, i), (i in boundaries))
...                for i in range(1, len(tokens)-1)
...                if tokens[i] in  '.?! ' ]
print(featuresets[:3])
"""
[({'next-word-capitalized': False,
   'prev-word': 'nov',
   'punct': '.',
   'prev-word-is-one-char': False},
  False),
 ({'next-word-capitalized': True,
   'prev-word': '29',
   'punct': '.',
   'prev-word-is-one-char': False},
  True),
 ({'next-word-capitalized': True,
   'prev-word': 'mr',
   'punct': '.',
   'prev-word-is-one-char': False},
  False)]
  """
```
最後就開始跑模型並回傳精度，如果精度不錯我們的標點符號分類器就完成了
```python
>>> size = int(len(featuresets) * 0.1)
>>> train_set, test_set = featuresets[size:], featuresets[:size]
>>> classifier = nltk.NaiveBayesClassifier.train(train_set)
>>> nltk. classify.accuracy(classifier, test_set)
0.936026936026936
```

知道那些符號是句尾後，就能用使用這些符號來斷句了
```python
def  segment_sentences (words):
    start = 0
    sents = []
    for i, word in enumerate(words):
         if word in  '.?!'  and classifier.classify(punct_features(words, i)) == True:
            sents.append(words[start:i+1])
            start = i+1
    if start < len(words):
        sents.append(words[start:])
    return sents
```

書上代碼def  segment_sentences (words):中 words似乎得使用tokens的格式，並且在tokens後面+["end
"]，因為提取特徵會參考符號的前後字，最後一個句點後方沒文字了會出現Error

參考資料: Python 自然语言处理 第二版<https://usyiyi.github.io/nlp-py-2e-zh/>
