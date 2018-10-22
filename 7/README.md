中文分詞
==
## 使用jieba套件

- jieba.cut()分詞功能，分為全模式與精確模式

```python
>>> seg_list = jieba.cut("垃圾不分藍綠", cut_all=True)
>>> print("Full Mode: " + "/ ".join(seg_list))  # 全模式

>>> seg_list = jieba.cut("垃圾不分藍綠", cut_all=False)
>>> print("Default Mode: " + "/ ".join(seg_list))  # 精确模式

>>> seg_list = jieba.cut("我送你離開千里之外你無聲黑白")  # 默认是精确模式
>>> print(", ".join(seg_list))

>>> seg_list = jieba.cut_for_search("沉默年代或許不該太遙遠的相愛")  # 搜索引擎模式
>>> print(", ".join(seg_list))


Full Mode: 垃圾/ 不分/ 藍/ 綠
Default Mode: 垃圾/ 不分/ 藍綠
我, 送, 你, 離開, 千里, 之外, 你, 無聲, 黑白
沉默, 年代, 或, 許不該, 太遙遠, 的, 相愛
```
"或許"的斷詞怪怪的，也許中國不講"或許"?


## 載入詞典
- 開發者可以指定自己自定義的詞典，以便包含jieba詞庫裡沒有的詞。雖然jieba有新詞識別能力，但是自行添加新詞可以保證更高的正確率
- 用法：jieba.load_userdict（file_name）＃file_name為文件類對像或自定義詞典的路徑
- 格式英漢詞典狀語從句：dict.txt一樣，一個詞佔一行;每一行分三部分：詞語，詞頻（可省略），詞性（可省略），用空格隔開，不可順序顛倒file_name若為路徑或二進制方式打開的文件，則文件必須為UTF-8編碼。
- 詞頻省略時使用自動計算的能保證分出該詞的詞頻。

先建立自定義辭典的文字檔，在後面接上字數與詞性並用空白隔開
```
或許 2
```

```python
#encoding=utf-8
>>> from __future__ import print_function, unicode_literals
>>> import sys
>>> sys.path.append("../")
>>> import jieba
>>> jieba.load_userdict("userdict.txt")
>>> import jieba.posseg as pseg

>>> ieba.add_word('或許')


>>> test_sent = ("沉默年代或許不該太遙遠的相愛")
>>> words = jieba.cut(test_sent)
>>> print('/'.join(words))

沉默/年代/或許/不該/太遙遠/的/相愛
```

## 調整詞典
- 使用add_word(word, freq=None, tag=None)狀語從句：del_word(word)柯林斯在程序中動態修改英漢詞典。
- 使用suggest_freq(segment, tune=True)柯林斯調節單個詞語的詞頻，使其能（或不能）被分出來。
- 注意：自動計算的詞頻在使用HMM新詞發現功能時可能無效。
```python
>>> print('/'.join(jieba.cut('如果放到post中將出錯。',HMM = False)))

>>> jieba.suggest_freq(['中','將'],True)

>>> print('/'. join(jieba.cut('如果放到post中將出錯。',HMM = False)))

##################################################################

>>> print('/'. join(jieba.cut(' “台中”正確應該不會被切開',HMM = False)))

>>> jieba.suggest_freq('台中',True)

>>> print('/'.join(jieba.cut(' “台中”正確應該不會被切開',HMM = False)))

如果/放到/post/中/將/出/錯/。
如果/放到/post/中/將/出/錯/。
 /“/台中/”/正/確/應/該/不/會/被/切/開
 /“/台中/”/正/確/應/該/不/會/被/切/開
```
