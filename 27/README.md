依存關係和依存文法
==
短語結構文法是關於詞和詞序列如何結合起來形成句子成分的。一個獨特的和互補的方式，依存語法，集中關注的是詞與其他詞之間的關係。依存關係是一個中心詞與它的依賴之間的二元對稱關係。一個句子的中心詞通常是動詞，所有其他詞要么依賴於中心詞，要么依賴路徑與它聯通。

一個句子的中心詞通常是動詞，所有其他詞要么依賴於中心詞，要么依賴路徑與它聯通。

依存結構：箭頭從中心詞指向它們的依賴；標籤表示依賴的語法功能如：主語、賓語或修飾語。
![](https://usyiyi.github.io/nlp-py-2e-zh/Images/ff868af58b8c1843c38287717b137f7c.jpg)

下面是NLTK為依存語法編碼的一種方式——注意它只能捕捉依存關係信息，不能指定依存關係類型：
```python
>>> groucho_dep_grammar = nltk.DependencyGrammar.fromstring( """ 
... 'shot' -> 'I' | 'elephant' | 'in' 
... 'elephant' -> 'an' | 'in' 
... 'in' -> 'pajamas' 
... 'pajamas' -> 'my' 
... """ )
>>> print (groucho_dep_grammar)

 Dependency grammar with 7 productions 
  'shot' -> 'I' 
  'shot' -> 'elephant' 
  'shot' -> 'in' 
  'elephant' -> 'an' 
  'elephant' -> 'in'
  'in' -> 'pajamas' 
  'pajamas' -> 'my'
```

- 樹庫和語法

corpus模塊定義了 treebank語料的閱讀器，其中包含了賓州樹庫語料的10％的樣本。

```python
>>> from nltk.corpus import treebank
>>> t = treebank.parsed_sents( 'wsj_0001.mrg' )[0]
>>> print (t)

(S 
  (NP-SBJ 
    (NP (NNP Pierre) (NNP Vinken) ) 
    (, ,) 
    (ADJP (NP (CD 61) (NNS years)) (JJ old)) 
    (, ,)) 
  (VP 
    (MD will) 
    (VP 
      (VB join) 
      (NP (DT the) (NN board) ) 
      (PP-CLR 
        (IN as) 
        (NP (DT a) (JJ nonexecutive) (NN director))) 
      (NP-TMP (NNP Nov.) (CD 29)))) 
  (. .))
```

到這裡為止之後的東西我實在搞不懂在幹嘛了，最後就貼個小結上來，剩下幾天試著用中文樹庫把這章複習一遍

小結
==
- 句子都有內部組織結構，可以用一棵樹表示。組成結構的顯著特點是：遞歸、中心詞、補語和修飾語。
- 語法是一個潛在的無限的句子集合的一個緊湊的特性；我們說，一棵樹是符合語法規則的或語法樹授權一棵樹。
- 語法是用於描述一個給定的短語是否可以被分配一個特定的成分或依賴結構的一種形式化模型。
- 給定一組句法類別，上下文無關文法使用一組生產式表示某類型A的短語如何能夠被分析成較小的序列α 1 ... α n。
- 依存語法使用產生式指定給定的中心詞的依賴是什麼。
- 一個句子有一個以上的句法分析就產生句法歧義（如介詞短語附著歧義）。
- 分析器是一個過程，為符合語法規則的句子尋找一個或多個相應的樹。
- 一個簡單的自上而下分析器是遞歸下降分析器，在語法產生式的幫助下遞歸擴展開始符號（通常是S），嘗試匹配輸入的句子。這個分析器並不能處理左遞歸產生式（如NP -> NP PP）。它盲目擴充類別而不檢查它們是否與輸入字符串兼容的方式效率低下，而且會重複擴充同樣的非終結符然後丟棄結果。
- 一個簡單的自下而上的分析器是移位-規約分析器，它把輸入移到一個堆棧中，並嘗試匹配堆棧頂部的項目和語法產生式右邊的部分。這個分析器不能保證為輸入找到一個有效的解析，即使它確實存在，它建立子結構而不檢查它是否與全部語法一致。


參考資料:
Python 自然语言处理 第二版<https://usyiyi.github.io/nlp-py-2e-zh/>
