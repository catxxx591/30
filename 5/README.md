正則表達式
==
先導入re和nltk庫，並利用nltk做出單字列表做正則表達式的操作
```python
>>> import re,nltk
>>> wordlist = [w for w in nltk.corpus.words.words('en') if w.islower()]
>>> wordlist
['a',
 'aa',
 'aal',
 'aalii',
 'aam',
 'aardvark',
 'aardwolf',
 'aba',
 'abac',
 ...
```
re庫使用格式re.search(r,str)，第一個參數輸入正則表達式，第二個放入被搜尋的字串或文本
正則表達式的基本與法規則可從wiki查詢<https://zh.wikipedia.org/wiki/%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F>
尋找wordlist中所有"ing"結尾的單字
```python
>>> [w for w in wordlist if re.search( 'ing$' , w)]
['abhorring',
 'abiding',
 'abounding',
 'absorbing',
 'abutting',
 'accommodating',
 'according',
 ...
```

.通配符匹配任何單個字符。假設我們有一個8個字母組成的詞的字謎室， j是其第三個字母， t是其第六個字母。
空白單元格中的每個地方，我們用一個句點，字符^匹配字符串的開始，$符號匹配字符串的結尾，使字串維持在8個字母。
```python
>>>[w for w in wordlist if re.search( '^..j..t..$' , w)]
['abjectly',
 'adjuster',
 'dejected',
 'dejectly',
 'injector',
 'majestic',
 'objectee',
 ...
```
- 範圍與閉包
