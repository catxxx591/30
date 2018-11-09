劃分詞塊
==
- 訓練基於分類器的詞塊劃分器

昨天透過train_sents訓練了標註器為詞性標記標註IOB詞塊標記，已經標記了93.3%的詞塊，再透過parse回傳詞塊樹。

今天再看書上補充兩個class
```python
class  ConsecutiveNPChunkTagger (nltk.TaggerI):

    def  __init__ (self, train_sents):
        train_set = []
        for tagged_sent in train_sents:
            untagged_sent = nltk.tag.untag(tagged_sent)
            history = []
            for i, (word, tag) in enumerate(tagged_sent):
                featureset = npchunk_features(untagged_sent, i, history) 
                train_set.append( (featureset, tag) )
                history.append(tag)
        self.classifier = nltk.MaxentClassifier.train( 
            train_set, algorithm= 'megam' , trace=0)
    def  tag (self, sentence):
        history = []
        for i, word in enumerate(sentence):
            featureset = npchunk_features(sentence, i, history)
            tag = self.classifier.classify(featureset)
            history.append(tag)
        return zip(sentence, history)
        
        
class  ConsecutiveNPChunker (nltk.ChunkParserI): 
    def __init__ (self, train_sents):
     
        tagged_sents = [[((w,t),c) for (w,t,c) in
                         nltk.chunk.tree2conlltags(sent)]
                        for sent in train_sents]
        self.tagger = ConsecutiveNPChunkTagger(tagged_sents)

    def  parse (self, sentence):
        tagged_sents = self.tagger.tag(sentence)
        conlltags = [(w,t,c) for ((w,t),c) in tagged_sents]
        return nltk.chunk.conlltags2tree(conlltags)
```

這兩行還要使用一個特徵提取器才能夠運行
```python
>>> def  npchunk_features (sentence, i, history):
...     word, pos = sentence[i]
...     return { "pos" : pos}
```

我在運行的時候__init__底下algorithm= 'megam'會出問題，查了一下似乎windows無法用megam。

再查nltk官方查到algorithm有四種參數，我就隨便改個algorithm= 'iis'就能跑了，會跑很久

```python
>>> chunker = ConsecutiveNPChunker(train_sents)
>>> print (chunker .evaluate(test_sents))

ChunkParse score:
    IOB Accuracy:  92.9%%
    Precision:     79.9%%
    Recall:        86.8%%
    F-Measure:     83.2%%
```

差別在於原來的特徵集是(word,tag)改為((word,i,history),tag)，在好幾天前有用過類似這樣加強前後文關係的特徵取法

接著書上再加強特徵，添加一個特徵表示前面詞的詞性標記。添加此特徵允許詞塊劃分器模擬相鄰標記之間的相互作用
```python
>>> def  npchunk_features (sentence, i, history):
...     word, pos = sentence[i]
...     if i == 0:
...         prevword, prevpos = "<START>" , "<START> " 
...     else :
...         prevword, prevpos = sentence[i-1]
...     return { "pos" : pos, "prevpos" : prevpos}
>>> chunker = ConsecutiveNPChunker(train_sents)
>>> print ( chunker.evaluate(test_sents))

ChunkParse score: 
    IOB Accuracy: 93.6% 
    Precision:81.9%
    Recall: 87.2% 
    F-Measure: 84.5%
```

下一步，書上為當前詞增加特徵，因為我們假設這個詞的內容應該對詞塊劃有用。發現這個特徵確實提高了詞塊劃分器的表現，大約1.5個百分點（相應的錯誤率減少大約10％）。
```
return { "pos" : pos, "word" : word, "prevpos" : prevpos}
```

最後書上加上"tags-since-dt"這個特徵，抽出自身之前的限定詞與自身之間所有單字的詞性標記，或如果沒有限定詞則在索引i之前自語句開始以來遇到的所有詞性標記。再定義一個tags-since-dt()放進去
```python
>>> def  npchunk_features (sentence, i, history):
...     word, pos = sentence[i]
...     if i == 0:
...         prevword, prevpos = "<START>" , "<START> " 
...     else :
...         prevword, prevpos = sentence[i-1]
...     if i == len(sentence)-1:
...         nextword, nextpos = "<END>" , "<END> " 
...     else :
...         nextword, nextpos = sentence[i+1]
...     return { "pos" :pos, 
...            "word" : word,
...             "prevpos" : prevpos,
...             "nextpos" : nextpos, 
... "prevpos+pos" : "%s+%s" % (prevpos, pos),   
... " pos+nextpos" : "%s+%s" % (pos, nextpos),
... "tags-since-dt" : tags_since_dt(sentence, i)}
```
```python
>>> def  tags_since_dt (sentence, i):
...     tags = set()
...     for word, pos in sentence[:i]:
...         if pos == 'DT' :
...             tags = set ()
...         else :
...             tags.add(pos)
...     return '+' .join(sorted(tags))
```

最後用調整好的特徵執行一遍
```python
>>> chunker = ConsecutiveNPChunker(train_sents)
>>> print (chunker.evaluate(test_sents))

ChunkParse score: 
    IOB Accuracy: 96.0% 
    Precision: 88.6% 
    Recall: 91.0% 
    F-Measure: 89.8%
```

相比昨天單純以詞性判斷詞塊，正確率從93.3%上升到96%，感覺很微小。

不過也許用在中文訓練上面會有更為明顯的效果，因為中文單字不像英文單字單字本身就有很強的詞性特徵，所以單純以詞性判斷詞塊精確度應該低很多

參考資料:
Python 自然语言处理 第二版 <https://github.com/catxxx591/30/edit/master/25/README.md>
