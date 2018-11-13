中文情感分析練習
==
這幾天找到中文的情感字典，與之前不一樣的是這是字典而不是語料庫，所以想說能練習看看

先把字典分別做成list

```python
with open('../dict/台湾大学简体中文情感极性词典ntusd/ntusd-negative.txt', mode='r', encoding='utf-8') as f:
    negs = f.readlines()
with open('../dict/台湾大学简体中文情感极性词典ntusd/ntusd-positive.txt', mode='r', encoding='utf-8') as f:
    poss = f.readlines()
pos = []
for i in poss:
    a=re.findall(r'\w+',i) 
    pos.extend(a)
neg = []
for i in negs:
    a=re.findall(r'\w+',i) 
    neg.extend(a)
```
```python
print(neg[:5])
['一下子爆发', '一下子爆发的一连串', '一巴掌', '一再', '一再叮嘱']
print(pos[:5])
['一帆风顺', '一帆风顺的', '一流', '一致', '一致的']
```

接著去除停用詞及分詞
```python
import jieba
stop_words = [w.strips() for w in open('stop_words.txt').readlines()]
def sent2word(sentence,stop_words=stop_words):
   words = jieba.cut(sentence)
   words = [w for w in words if w not in stop_words]
   return words
```

爬文看到這時候還需要兩種字典

1. 程度字典
2. 否定詞字典

程度字典包含整理出來的副詞吧，像:最、非常、很、似乎，字典種整理了這些字，並附上數字等級，與情感數值相乘。這情感字典我好像要換一本

否定詞字典當中的字可能會反轉整句話的正反，像:不討厭、不值錢

參考資料:
<https://blog.csdn.net/bcj296050240/article/details/46686797>
<https://blog.csdn.net/chenpe32cp/article/details/77801600>
