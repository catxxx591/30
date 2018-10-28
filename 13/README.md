有監督分類的更多例子
==
## 句子分割
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

