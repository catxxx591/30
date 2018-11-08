標註IOB詞塊標記
==

昨天定義了UnigramChunker類，使用一元標註器給句子加詞塊標記。這個類的大部分代碼只是用來在NLTK的ChunkParserI接口使用的詞塊樹表示和嵌入式標註器使用的IOB表示之間鏡像轉換。

類定義了兩個方法：一個構造函數，當我們建立一個新的UnigramChunker時調用；以及parse方法，用來給新句子劃分詞塊。

先準備資料
```python
>>> from nltk.corpus import conll2000
>>> test_sents = conll2000.chunked_sents('test.txt', chunk_types=['NP'])
>>> train_sents = conll2000.chunked_sents('train.txt', chunk_types=['NP'])
```
建了書上的class
```python
class UnigramChunker(nltk.ChunkParserI):
    def __init__(self, train_sents): 
        train_data = [[(t,c) for w,t,c in nltk.chunk.tree2conlltags(sent)]
                      for sent in train_sents]
        self.tagger = nltk.UnigramTagger(train_data) 

    def parse(self, sentence): 
        pos_tags = [pos for (word,pos) in sentence]
        tagged_pos_tags = self.tagger.tag(pos_tags)
        chunktags = [chunktag for (pos, chunktag) in tagged_pos_tags]
        conlltags = [(word, pos, chunktag) for ((word,pos),chunktag)
                     in zip(sentence, chunktags)]
        return nltk.chunk.conlltags2tree(conlltags)
```

主要流程是使用 nltk.chunk.tree2conlltags()將train_sents 每個字分為word,tag,chunk三元組並將train_data作為nltk.UnigramTagger()可使用的格式。

train_data格式是 [  [(tag,chunk),(tag,chunk)...] , [(tag,chunk),(tag,chunk)...]...  ]，不知道怎麼表達...就長這樣

把做好的train_data丟進nltk.UnigramTagger()會訓練出一個標註器，之後使用unigram_chunker.tagger.tag()就可以為詞性標記標註IOB詞塊標記

再來看書上範例
```python
>>> postags = sorted(set(pos for sent in train_sents
...                      for (word,pos) in sent.leaves()))
>>> print (unigram_chunker.tagger.tag(postags))

 [('#' , 'B-NP'), ('$', 'B-NP'), ("''", 'O'), ('(', 'O'), (')', 'O') , 
(',', 'O'), ('.', 'O'), (':', 'O'), ('CC', 'O'), ('CD', 'I-NP '), 
('DT', 'B-NP'), ('EX', 'B-NP'), ('FW', 'I-NP'), ('IN', 'O'), 
( 'JJ', 'I-NP'), ('JJR', 'B-NP'), ('JJS', 'I-NP'), ('MD', 'O'), 
('NN', 'I-NP'), ('NNP', 'I-NP'), ('NNPS', ' I-NP'), ('NNS', 'I-NP'), 
('PDT', 'B-NP'), ('POS', 'B-NP'), ('PRP', 'B- NP'), ('PRP$', 'B-NP'),
('RB', 'O'), ('RBR', 'O'), ('RBS', 'B-NP'), ('RP', 'O'), ('SYM', 'O' ), 
('TO', 'O'), ('UH', 'O'), ('VB', 'O'), ('VBD', 'O'), ('VBG', 'O' ), 
('VBN', 'O'), ('VBP', 'O'), ('VBZ', 'O'), ('WDT', 'B-NP'), 
('WP', ' B-NP'), ('WP$', 'B-NP'), ('WRB', 'O'), ('``', 'O')]
```

覺得定義postags的那行好噁心，我還是不能肯定sent.leaves()那個sent是什麼東西來自哪裡

今天整理新單元的東西，看懂上面在寫什麼就花好多時間了QQ

參考資料:
Python 自然语言处理 第二版<https://usyiyi.github.io/nlp-py-2e-zh/>
