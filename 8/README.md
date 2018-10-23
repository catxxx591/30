提取關鍵字
==
在字串當中抓取單字的方法，依算法分為TF-IDF與TextRank兩種
## 1. TF-IDF

jieba.analyse.extract_tags(sentence, topK=20, withWeight=False, allowPOS=())
- sentence為待提取的文本
- topK為返回幾個TF / IDF權重最大的關鍵詞，默認值為20
- withWeight為是否一併返回關鍵詞權重值，默認值為False
- allowPOS僅包括指定詞性的詞，默認值為空，即不篩選
- jieba.analyse.TFIDF（idf_path = None）新建TFIDF實例，idf_path為IDF頻率文件


找出的關鍵詞會依照詞頻權重排列
```python
>>> s = "我出門去買早餐"
>>> print(jieba.analyse.extract_tags(s, topK=20, withWeight=False, allowPOS=()))
>>> for x, w in jieba.analyse.extract_tags(s, withWeight=True):
...     print('%s %s' % (x, w))

['出門', '早餐']
出門 5.97738375145
早餐 4.29868637196
```



## 2. TextRank

jieba.analyse.textrank（sentence, topK=20, withWeight=False, allowPOS=('ns','n','vn','v'））直接使用，接口相同，注意默認過濾詞性。
jieba.analyse.TextRank（）新建自定義TextRank實例

```python
>>> print(jieba.analyse.textrank(s,  withWeight=False))
>>> for x, w in jieba.analyse.textrank(s, withWeight=True):
...     print('%s %s' % (x, w))
    
['早餐', '出門']
早餐 1.0
出門 0.9961264494011037
```

## 詞性標註

```python
>>> import jieba.posseg as pseg
>>> words = pseg.cut(s)
>>> for word, flag in words:
...    print('%s %s' % (word, flag))

我 r
出門 v
去 v
買 v
早餐 n
```
