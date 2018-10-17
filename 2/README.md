獲得文本和詞彙資源
==
## 獲取語料庫
nltk之中有許多內建的語料庫，可從nltk.cotpus選擇經分類的語料庫，對語料庫使用.fields()查看該語料庫的文本名稱
```python
>>>nltk.corpus.gutenberg.fileids()

['austen-emma.txt',
 'austen-persuasion.txt',
 'austen-sense.txt',
 'bible-kjv.txt',
 'blake-poems.txt',
 'bryant-stories.txt',
 'burgess-busterbrown.txt',
 'carroll-alice.txt',
 'chesterton-ball.txt',
 'chesterton-brown.txt',
 'chesterton-thursday.txt',
 'edgeworth-parents.txt',
 'melville-moby_dick.txt',
 'milton-paradise.txt',
 'shakespeare-caesar.txt',
 'shakespeare-hamlet.txt',
 'shakespeare-macbeth.txt',
 'whitman-leaves.txt']
```

- 閱讀文本
nltk可以對文本執行不同的查看方式，以下先由最基本的方法測試
1. 讀取全部文本，包含隱藏符號
```python
>>>nltk.corpus.gutenberg.raw('austen-emma.txt')

"""
'[Emma by Jane Austen 1816]\n\nVOLUME I\n\nCHAPTER I\n\n\nEmma Woodhouse, 
...
"""
```
2. 以句為單位分割
```python
>>>nltk.corpus.gutenberg.sents('austen-emma.txt')

[['[', 'Emma', 'by', 'Jane', 'Austen', '1816', ']'], ['VOLUME', 'I'], ...]
```
3. 以字為單位分割
```python
nltk.corpus.gutenberg.words('austen-emma.txt')

['[', 'Emma', 'by', 'Jane', 'Austen', '1816', ']', ...]
```
