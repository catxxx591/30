處理原始文本
==
- 從網絡和硬盤訪問文本
NLTK語料庫集合中有古騰堡項目的一小部分樣例文本。然而，你可能對分析古騰堡項目的其它文本感興趣。
你可以在http://www.gutenberg.org/catalog/上瀏覽25,000本免費在線書籍的目錄，獲得ASCII碼文本文件的URL。
雖然90％的古騰堡項目的文本是英語的，它還包括超過50種語言的材料，包括加泰羅尼亞語、中文、荷蘭語、芬蘭語、法語、德語、意大利語、葡萄牙語和西班牙語（每種語言都有超過100個文本）。
- 電子書
編號2554的文本是《罪與罰》的英文翻譯，我們可以如下方式訪問它。
```python
>>> from urllib import request
>>> url = "http://www.gutenberg.org/files/2554/2554-0.txt" 
>>> response = request.urlopen(url)
>>> raw = response.read ().decode( 'utf8' )
>>> print(raw[:50])
The Project Gutenberg EBook of Crime and Punishme
```

文本常有一些作者簡介及出版社等等的其他文字，可以先以人工查看本文開始及結束的句子，將raw的範圍縮至本文內容
```python
>>> raw.find( "PART I" )
 5338
>>> raw.rfind( "END OF THIS PROJECT GUTENBERG EBOOK CRIME AND PUNISHMENT" )" )
 1157887
>>> raw = raw[5338:1157887] >>> raw.find( "PART I" )
 0
```

- HTML
使用requests搭配BeautifulSoup4

```python
>>> url = "http://news.bbc.co.uk/2/hi/health/2284783.stm" 
>>> html = request.urlopen(url).read().decode( 'utf8' )
>>> html[:60]
>>> from bs4 import BeautifulSoup
>>> raw = BeautifulSoup(html).get_text()
>>> tokens = word_tokenize(raw)
>>> tokens
['BBC', 'NEWS', '|', 'Health', '|', 'Blondes', "'to", 'die', 'out', ...]
```

### 讀取本地文件

```python
>>> f = open( '../txt/booksx.txt',encoding='utf8' )
>>> book = f.read()
>>> book[:200]

"""
'\ufeff獨立宣言\n前言\n英國與其美洲殖民地之間的戰爭於一七七五年四月開始。隨著戰爭的延續，和解的希望逐漸消失，完全獨立已成為殖民地的目標。一七七六年六月七日，在大陸會議的一次集會中，維吉尼亞的理查．亨利．李提出一個議案，宣稱: 「這些殖民地是自由和獨立的國家，並且按其權利必須是自由和獨立的國家。」六月十日大陸會議指定一個委員會草擬獨立宣言。實際的起草工作由湯瑪斯．傑佛遜負責。七月四日獨立宣言獲得通過，並'
"""
```
可使用os套件檢查目錄的檔案
```python
>>> import os
>>> os.listdir( '.' )
```

