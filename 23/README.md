從文本提取信息
==
新聞分類到85%精確度就上不去了，再繼續探索nltk

- 信息提取
信息有很多種形狀和大小。如果我們的數據是表格形式，那麼回答這些問題就很簡單了。
```python
>>> locs = [( 'Omnicom' , 'IN' , 'New York' ),
...         ( 'DDB Needham' , 'IN' , 'New York' ),
...         ( 'Kaplan Thaler Group' , 'IN' , 'New York' ),
...         ( 'BBDO South' , 'IN' , 'Atlanta' ),
...         ( 'Georgia-Pacific' , 'IN' , 'Atlanta' )]
>>> query = [e1 for (e1, rel, e2)in locs if e2== 'Atlanta' ]
>>> print (query)

['BBDO South', 'Georgia-Pacific']
```

- 詞塊劃分
我們將用於實體識別的基本技術是詞塊劃分，它分割和標註多詞符的序列，如圖所示。小框顯示詞級分詞和詞性標註，大框顯示高級別的詞塊劃分。

每個這種較大的框叫做一個詞塊。就像分詞忽略空白符，詞塊劃分通常選擇詞符的一個子集。同樣像分詞一樣，詞塊劃分器生成的片段在源文本中不能重疊。
![](https://usyiyi.github.io/nlp-py-2e-zh/Images/0e768e8c4378c2b0b3290aab46dc770e.jpg)

1. 名詞短語詞塊劃分
我們將首先思考名詞短語詞塊劃分或NP詞塊劃分任務，在那裡我們尋找單獨名詞短語對應的詞塊。
![](https://usyiyi.github.io/nlp-py-2e-zh/Images/da516572f97daebe1be746abd7bd2268.jpg)
2. 標記模式
組成一個詞塊語法的規則使用標記模式來描述已標註的詞的序列。一個標記模式是一個詞性標記序列，用尖括號分隔。標記模式類似於正則表達式模式
```
<DT>?<JJ>*<NN>
```


3. 用正則表達式進行詞塊劃分
```python
grammar = r """ 
  NP: {<DT|PP\$>?<JJ>*<NN>} # chunk determiner/possessive, adjectives and noun 
      {<NNP>+} # chunk sequences of proper nouns 
"""
cp = nltk.RegexpParser(grammar)
sentence = [( "Rapunzel" , "NNP" ), ( "let" , "VBD" ), ( "down" , "RP" ), [1]
                 ( "her" , "PP$" ), ( "long" , " JJ" ), ( "golden" , "JJ" ), ( "hair" , "NN" )]
```
4. 探索文本語料庫
我們看到了我們如何在已標註的語料庫中提取匹配的特定的詞性標記序列的短語。我們可以使用詞塊劃分器更容易的做同樣的工作
```
>>> cp = nltk.RegexpParser( 'CHUNK: {<V.*> <TO> <V.*>}' )
>>> brown = nltk.corpus.brown
>>> for sent in brown.tagged_sents() :
...     tree = cp.parse(sent)
...     for subtree in tree.subtrees():
...         if subtree.label() == 'CHUNK' : print (subtree)
```
5. 詞縫加塞
有時定義我們想從一個詞塊中排除什麼比較容易。我們可以定義詞縫為一個不包含在詞塊中的一個詞符序列。在下面的例子中，barked/VBD at/IN是一個詞縫：
```
[ the/DT little/JJ yellow/JJ dog/NN ] barked/VBD at/IN [ the/DT cat/NN ]
```
6. 詞塊的表示：標記與樹
作為標註和分析之間的中間狀態，詞塊結構可以使用標記或樹來表示。最廣泛的文件表示使用IOB標記。

在這個方案中，每個詞符被三個特殊的詞塊標記之一標註，I（內部），O（外部）或B（開始）。
![](https://usyiyi.github.io/nlp-py-2e-zh/Images/542fee25c56235c899312bed3d5ee9ba.jpg)

- 開發和評估詞塊劃分器
1. 讀取IOB格式與CoNLL2000語料庫

使用corpus模塊，我們可以加載已經標註並使用IOB符號劃分詞塊的《華爾街日報》文本。這個語料庫提供的詞塊類型有NP，VP和PP。
```python
from nltk.corpus import conll2000
```
2. 簡單的評估和基準
```python
>>> from nltk.corpus import conll2000
>>> cp = nltk.RegexpParser( "" )
>>> test_sents = conll2000.chunked_sents( 'test.txt' , chunk_types=[ 'NP' ])
>>> print (cp .evaluate(test_sents))

 ChunkParse score: 
    IOB Accuracy: 43.4% 
    Precision: 0.0% 
    Recall: 0.0% 
    F-Measure: 0.0%
```
IOB標記準確性表明超過三分之一的詞被標註為O，即沒有在NP詞塊中。然而，由於我們的標註器沒有找到任何詞塊，其精度、召回率和F-度量均為零。

參考資料:
Python 自然语言处理 第二版<https://usyiyi.github.io/nlp-py-2e-zh/>
