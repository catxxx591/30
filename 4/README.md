處理原始文本
==
- 從網絡和硬盤訪問文本
NLTK語料庫集合中有古騰堡項目的一小部分樣例文本。然而，你可能對分析古騰堡項目的其它文本感興趣。
你可以在http://www.gutenberg.org/catalog/上瀏覽25,000本免費在線書籍的目錄，獲得ASCII碼文本文件的URL。
雖然90％的古騰堡項目的文本是英語的，它還包括超過50種語言的材料，包括加泰羅尼亞語、中文、荷蘭語、芬蘭語、法語、德語、意大利語、葡萄牙語和西班牙語（每種語言都有超過100個文本）。

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
>>> raw.rfind( "End of Project Gutenberg's Crime" )
 1157743
>>> raw = raw[5338:1157743] >>> raw.find( "PART I" )
 0
```




