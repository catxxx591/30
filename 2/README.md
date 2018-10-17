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

## 布朗語料庫
布朗語料庫是第一個百萬詞級的英語電子語料庫的，由布朗大學於1961 年創建。這個語料庫包含500 個不同來源的文本，按照文體分類，如：新聞、社論等

```python
>>> from nltk.corpus import brown
>>> brown.categories()

['adventure', 'belles_lettres', 'editorial', 'fiction', 'government', 'hobbies',
'humor', 'learned', 'lore', 'mystery', 'news', 'religion', 'reviews', 'romance',
'science_fiction']
```
布朗語料庫是一個研究文體之間的系統性差異——一種叫做文體學的語言學研究——很方便的資源。讓我們來比較不同文體中的情態動詞的用法。第一步是產生特定文體的計數。
```python
from nltk.corpus import brown
>>> news_text = brown.words(categories='news')
>>> fdist = nltk.FreqDist(w.lower() for w in news_text)
>>> modals = ['can', 'could', 'may', 'might', 'must', 'will']
>>> for m in modals:
...     print(m + ':', fdist[m], end=' ')

can: 94 could: 87 may: 93 might: 38 must: 53 will: 389 
```
## 條件頻率分佈
當語料文本被分為幾類，如文體、主題、作者等時，我們可以計算每個類別獨立的頻率分佈。這將允許我們研究類別之間的系統性差異。
將布朗語料庫中的每份文本，依照類別作出配對
```python
>>>genre_word = [(genre, word) 
...               for genre in [ 'news' , 'romance' ] 
...               for word in brown.words(categories=genre)]
>>>genre_word[:3]
[('news', 'The'), ('news', 'Fulton'), ('news', 'County')]
>>>genre_word[-3:]
[('romance', 'not'), ('romance', "''"), ('romance', '.')]
```
再把genre_word丟進ConditionFreqDist裡，就可以在cfd中依照類別找出單字頻率
```python
>>> cfd = nltk.ConditionalFreqDist(genre_word)
>>>cfd['news'].most_common(10)
[('the', 5580),
 (',', 5188),
 ('.', 4030),
 ('of', 2849),
 ('and', 2146),
 ('to', 2116),
 ('a', 1993),
 ('in', 1893),
 ('for', 943),
 ('The', 806)]
 ```
## 詞彙列表語料庫
詞彙語料庫是Unix中的/usr/share/dict/words文件，被一些拼寫檢查程序使用。我們可以用它來尋找文本語料中不尋常的或拼寫錯誤的詞彙
```python
def  unusual_words (text):
    text_vocab = set(w.lower() for w in text if w.isalpha())
    english_vocab = set(w.lower() for w in nltk.corpus.words.words())
    unusual = text_vocab - english_vocab
    return sorted(unusual)

>>> unusual_words(nltk.corpus.gutenberg.words( 'austen-sense.txt' ))

還有一個停用詞語料庫，就是那些高頻詞彙，如the，to和also，我們有時在進一步的處理之前想要將它們從文檔中過濾。停用詞通常幾乎沒有什麼詞彙內容，而它們的出現會使區分文本變困難。
```python
from nltk.corpus import stopwords
stopwords.words( 'english' )
```
定義一個函數來計算文本中沒有在停用詞列表中的詞的比例：
```python
>>> def  content_fraction (text):
 ...     stopwords = nltk.corpus.stopwords.words( 'english' )
 ...     content = [w for w in text if w.lower() not  in stopwords]
 ...     return len(content) / len(text)
 ... 
>>> content_fraction(nltk.corpus.reuters.words())
 0.7364374824583169
```
