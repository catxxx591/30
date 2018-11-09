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

再查nltk官方查到algorithm有四種參數，我就隨便改個algorithm= 'iis'就能跑了

```python
>>> chunker = ConsecutiveNPChunker(train_sents)
>>> print (chunker .evaluate(test_sents))

ChunkParse score:
    IOB Accuracy:  92.9%%
    Precision:     79.9%%
    Recall:        86.8%%
    F-Measure:     83.2%%
```
