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
re庫使用格式re.search(r,str)，第一個參數輸入正則表達式，第二個放入被搜尋的字串或文本。

正則表達式的基本與法規則可從wiki查詢<https://zh.wikipedia.org/wiki/%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F>。

尋找wordlist中所有"ing"結尾的單字。
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

![](https://github.com/catxxx591/30/blob/master/img/cellphone_char.jpg?raw=true)
T9系統用於在手機上輸入文本（見3.5 )）。兩個或兩個以上以相同擊鍵順序輸入的詞彙，叫做textonyms。例如，hole和golf都是通過序列4653輸入。

還有哪些其它詞彙由相同的序列產生？這裡我們使用正則表達式« ^[ghi][mno][jlk][def]$ »：
```python
>>> [w for w in wordlist if re.search( '^[ghi][mno][jlk][def]$' , w)]
 ['gold', 'golf', 'hold', 'hole' ]
```
- 練習
在W3C日期時間格式中，日期像這樣表示：2009-12-31。
Replace the ? in the following Python code with a regular expression, in order to convert the string '2009-12-31' to a list of integers [2009, 12, 31] :
```python
>>> [int(n) for n in re.findall('[0-9]+', '2009-12-31' )]
[2009, 12, 31]
```
- 查找詞幹

```python
>>> re.findall(r '^.*(ing|ly|ed|ious|ies|ive|es|s|ment)$' , 'singsing' )
 ['ing']
```
使用()會有印出()內容的效果，如果要印出整個字串要在()中開頭加上?:
```python
>>> re.findall(r '^.*(?:ing|ly|ed|ious|ies|ive|es|s|ment)$' , 'singsing' )
 ['singing']
 ```
 
 如果想將詞分成詞幹和後綴兩部分，可分別用兩個()隔開。
```python
>>> re.findall( r'^(.*?)(ing|ly|ed|ious|ies|ive|es|s|ment)$', 'singing' )
[('sing', 'ing')]
```

這看起來很有用途，但仍然有一個問題。讓我們來看看另外的詞，processes：
```python
>>> re.findall(r '^(.*)(ing|ly|ed|ious|ies|ive|es|s|ment)$' , 'processes' )
 [('processe', 's') ]
```

資料來源: Python 自然语言处理 第二版<https://usyiyi.github.io/nlp-py-2e-zh/>
正則表達式錯誤地找到了後綴-s，而不是後綴-es。這表明另一個微妙之處：星號操作符是“貪婪的”，所以表達式的.*部分試圖盡可能多的匹配輸入的字符串。如果我們使用“非貪婪”版本的“*”操作符，寫成*?，我們就得到我們想要的：
```python
>>> re.findall(r '^(.*?)(ing|ly|ed|ious|ies|ive|es|s|ment)$' , 'processes' )
 [('process', 'es' )]
```
如果單字沒有後綴詞，可在第二組()加上?，來得到空後綴：
```python
>>> re.findall(r'^(.*?)(ing|ly|ed|ious|ies|ive|es|s|ment)?$' , 'apple' )
[('apple', '')]
```
這種方法仍然有許多問題，但我們仍將繼續定義一個函數來獲取詞幹，並將它應用到整個文本：
```python
>>> def  stem (word):
...     regexp = r '^(.*?)(ing|ly|ed|ious|ies|ive|es|s|ment)?$' 
...     stem, suffix = re.findall(regexp, word)[0]
...     return stem
... 
>>> raw = """DENNIS: Listen, strange women lying in ponds distributing swords 
... is no basis for a system of government . Supreme executive power derives from 
... a mandate from the masses, not from some farcical aquatic ceremony.""" 
>>> tokens = word_tokenize(raw)
>>> [stem(t) for t in tokens]
['DENNIS', ':', 'Listen', ',', 'strange', 'women', 'ly', 'in', 'pond', 'distribut', 
'sword', 'i', ' no', 'basi', 'for', 'a', 'system', 'of', 'govern', '.', 'Supreme', 
'execut', 'power', 'deriv', 'from' , 'a', 'mandate', 'from', 'the', 'mass', ',', 
'not', 'from', 'some', 'farcical', 'aquatic', 'ceremony', ' .']
```
正則表達式不但將ponds的s刪除，也將is和basis的刪除。它產生一些非詞如distribut和deriv，但這些在一些應用中是可接受的詞幹。



