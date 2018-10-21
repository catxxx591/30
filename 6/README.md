分詞斷句
==
從資料夾中開啟美國獨立宣言文本
```python
>>> usa = open('../txt/usa_en.txt',encoding='utf8').read()
>>> usa = ''.join(usa)
```

使用re.split()將單字分開做成list
```python
>>> print(re.split(r'\W+', usa))
['Declaration', 'of', 'Independence', 'The', 'Unanimous', 'Declaration', 'of', 'the', 'Thirteen', 'United', 'States', 'of', 'America', 'When', 'in', 'the', 'course', 'of', 'human', 'events', 'it', 'becomes', 'necessary', 'for', 'one', 'people',...
```
使用r'\W+'結果讓nature's God被分為['nature','s','God']
改用r'\s+'只挑選非空格字串
```python
>>> re.split(r'\s+', usa)
['Declaration',
 'of',
 'Independence',
 ...
 'of',
 "nature's",
 'God',
 'entitle',
 ...
```
## 練習
- 定義一個字符串s = 'colorless'。寫一個Python語句將其變為“colourless”，只使用切片和連接操作。
```python
>>> s = 'colorless'
>>> print(s[:4]+'u'+s[4:])
colourless
```

- 我們可以為分片指定一個“步長”。下面的表達式間隔一個字符返回一個片內字符：monty[6:11:2]。也可以反向進行：monty[10:5:-2]。自己嘗試一下，然後實驗不同的步長。
```python
>>> monty = '12345678901234567890'
>>> monty[10:2:-3]
'185'
```

- 寫正則表達式匹配下面字符串類：
1. 一個單獨的限定符（假設只有a , an和the為限定符）。
2. 整數加法和乘法的算術表達式，如2*3+8。[]
```python
[a,an,the]
看不懂

- 寫一個工具函數以URL為參數，返回刪除所有的HTML標記的URL的內容。使用from urllib import request和request.urlopen( 'http://nltk.org/' ).read().decode( 'utf8' )來訪問URL的內容。
```python
>>> def cleanhtml(url=""):
...     from urllib import request
...     from bs4 import BeautifulSoup
...     html = request.urlopen( url ).read().decode( 'utf8' )
...     raw = BeautifulSoup(html).get_text()
...     return raw
>>> cleanhtml('http://nltk.org/')
```

- 將一些文字保存到文件corpus.txt。定義一個函數load(f)以要讀取的文件名為其唯一參數，返回包含文件中文本的字符串。
1. 使用nltk.regexp_tokenize()創建一個分詞器分割這個文本中的各種標點符號。使用一個多行的正則表達式，使用verbose標誌(?x)帶有行內註釋。
2. 使用nltk.regexp_tokenize()創建一個分詞器分割以下幾種表達式：貨幣金額；日期；個人和組織的名稱。
```python
>>> def load(f=""):
...     usa = open('../txt/'+f)
...     return ''.join(usa.read())
>>> load('usa_en.txt')
```

- 你能寫一個正則表達式以這樣的方式來分詞嗎，將詞don't分為do和n't？解釋為什麼這個正則表達式無法正常工作：« n't|\w+ »。
```python
>>> re.findall(r"n't|\w+{2}","don't")
['do', "n't"]
```
- 分詞的一個有趣的挑戰是已經被分割的跨行的詞。例如如果long-term被分割，我們就得到字符串long-\nterm

寫一個正則表達式，識別連字符連結的跨行處的詞彙。這個表達式將需要包含\n字符。
使用re.sub()從這些詞中刪除\n字符。
你如何確定一旦換行符被刪除後不應該保留連字符的詞彙，如'encyclo-\npedia'？
```python
>>> txt = "long-\ntrem"
>>> re.findall(r'.+(?:-\n).+',txt)
>>> re.sub(r'\n',"",txt)
```
