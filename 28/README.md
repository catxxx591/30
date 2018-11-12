標註中文詞性
==
nltk.corpus內有sinica_treebank繁體中文的語料庫，用法和corpus內其他文章滿相似的

```python
print(nltk.corpus.sinica_treebank.tagged_sents()[:10])
```
```
[[('一', 'Neu')],
 [('友情', 'Nad')],
 [('嘉珍', 'Nba'),
  ('和', 'Caa'),
  ('我', 'Nhaa'),
  ('住在', 'VC1'),
  ('同一條', 'DM'),
  ('巷子', 'Nab')],
 [('我們', 'Nhaa'), ('是', 'V_11'), ('鄰居', 'Nab')],
 [('也', 'Dbb'), ('是', 'V_11'), ('同班', 'Nv3'), ('同學', 'Nab')],
 [('我們', 'Nhaa'), ('常常', 'Dd'), ('一起', 'Dh'), ('上學', 'VA4')],
 [('一起', 'Dh'), ('回家', 'VA13')],
 [('有一天', 'DM')],
 [('上學', 'VA4'), ('時', 'Ng')],
 [('我', 'Nhaa'), ('到', 'P61'), ('她', 'Nhaa'), ('家', 'Ncb'), ('等候', 'VK2')]]
```
這裡很多詞性我都找不到是怎樣...


依前幾天學到的，有詞性標記的語料庫就能夠訓練出詞性標註器
```python
tagged_sents = sinica_treebank.tagged_sents()
size = int(len(tagged_sents) * 0.7)
train_sents = _tagged_sents[:size]	
test_sents = tagged_sents[size:]	

tagged_sents = sinica_treebank.tagged_sents()
size = int(len(tagged_sents) * 0.7)
train_sents = tagged_sents[:size]
test_sents = tagged_sents[size:]

t0 = nltk.DefaultTagger('Nab')
t1 = nltk.UnigramTagger(train_sents, backoff=t0)
t2 = nltk.BigramTagger(train_sents, backoff=t1)	
 

print (t2.evaluate(test_sents))	

0.6980264134144532
```
參考資料:
Python 自然語言處理第二版<https://usyiyi.github.io/nlp-py-2e-zh/>
NLTK中文詞性自動標註<https://www.itread01.com/p/436065.html>
學習筆記CB002:詞幹提取、詞性標註、中文切詞、文件分類<https://codertw.com/%E4%BA%BA%E5%B7%A5%E6%99%BA%E6%85%A7/4420/>
