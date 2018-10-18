WordNet
==
WordNet是面向語義的英語詞典，類似與傳統辭典，但具有更豐富的結構。
NLTK中包括英語WordNet，共有155,287個詞和117,659個同義詞集合。
我們將以尋找同義詞和它們在WordNet中如何訪問開始。
```python
from nltk.corpus import wordnet as wn
```
- 意義與同義字

我們可以使用wordnet.synsets()來從wordnet中查詢單字
```python
>>>print(wn.synsets( 'motorcar' ))
>>>print(wn.synsets( 'automobile' ))
[Synset('car.n.01')]
[Synset('car.n.01'), Synset('automobile.v.01')]
```

car.n.01被稱為synset或“同義詞集”，意義相同的詞（或“詞條”）的集合，motorcar只有一個可能的含義，它被定義為car.n.01，car的第一個名詞意義。
```python
>>>wn.synset('car.n.01).lemmas()
[Lemma('car.n.01.car'),
 Lemma('car.n.01.auto'),
 Lemma('car.n.01.automobile'),
 Lemma('car.n.01.machine'),
 Lemma('car.n.01.motorcar')]
 
 >>>wn.synset('car.n.01').lemmas()[0].name()
 'car'
 
 >>>wn.synset('car.n.01').lemma_names()
['car', 'auto', 'automobile', 'machine', 'motorcar']

```
wordnet.synset()用來查詢同義詞集，能查看單字的定義及例句
synset()、synsets()好像查單字的要+s
```python
>>>print(wn.synset( 'car.n.01' ).definition())
>>>print(wn.synset('car.n.01').examples())

a motor vehicle with four wheels; usually propelled by an internal combustion engine
['he needs a car to get to work']
```
## wordnet層次結構
wordnet之中將單字以類似標籤的方式做分類，並整理出類似樹狀圖的層次結構，節點會有上位詞及下位詞，越下位會越具體
- 語義相似度

如果兩個同義詞集共用一個非常具體的上位詞——在上位詞層次結構中處於較低層的上位詞——它們一定有密切的聯繫。

用一些簡單的名詞來測試
```python
milk = wn.synset('milk.n.01')
dog = wn.synset('dog.n.01')
cat = wn.synset('cat.n.01')
bird = wn.synset('bird.n.01')
spider = wn.synset('spider.n.01')
fish = wn.synset('fish.n.01')
pig = wn.synset('pig.n.01')
ship = wn.synset('ship.n.01')
bee = wn.synset('bee.n.01')
ant = wn.synset('ant.n.01')
egg = wn.synset('egg.n.01')
duck = wn.synset('duck.n.01')
```
使用.lowest_common_hypernyms()尋找兩單字最接近的上位詞，通常越接近就表示兩者越相關
```python
>>>print(cat.lowest_common_hypernyms(dog))
>>>print(cat.lowest_common_hypernyms(bird))
>>>print(bee.lowest_common_hypernyms(ant))
>>>print(bee.lowest_common_hypernyms(spider))
>>>print(bird.lowest_common_hypernyms(duck))
>>>print(bird.lowest_common_hypernyms(egg))
[Synset('carnivore.n.01')]#肉食動物
[Synset('vertebrate.n.01')]#脊椎動物
[Synset('hymenopterous_insect.n.01')]#昆蟲
[Synset('arthropod.n.01')]#節肢動物
[Synset('bird.n.01')]#鳥類
[Synset('living_thing.n.01')]#生物
```
可以看出個單字兩兩間的關係，以結果來看bird和egg似乎是關係最遠的，其他組別就較不明顯。

.path_similarity()是基於上位詞層次結構中相互連接的概念之間的最短路徑在0 - 1範圍的打分（兩者之間沒有路徑就返回-1）。
我們可以用.path_similarity()作簡單的量化來比較兩單字的相關性
```python
print(cat.path_similarity(dog))
print(cat.path_similarity(bird))
print(bee.path_similarity(ant))
print(bee.path_similarity(spider))
print(bird.path_similarity(duck))
print(bird.path_similarity(egg))

0.2
0.14285714285714285
0.3333333333333333
0.16666666666666666
0.2
0.09090909090909091
```

參考資料: Python 自然語言處理第二版<https://usyiyi.github.io/nlp-py-2e-zh/>
