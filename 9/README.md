學習分類文本
==
本章的目的是要回答下列問題：

1. 我們怎樣才能識別語言數據中能明顯用於對其分類的特徵？
2. 我們怎樣才能構建語言模型，用於自動執行語言處理任務？
3. 從這些模型中我們可以學到哪些關於語言的知識？

## 1. 有監督分類
分類是為給定的輸入選擇正確的類標籤的任務。在基本的分類任務中，每個輸入被認為是與所有其它輸入隔離的，並且標籤集是預先定義的。這裡是分類任務的一些例子：

- 判斷一封電子郵件是否是垃圾郵件。
- 從一個固定的主題領域列表中，如“體育”、“技術”和“政治”，決定新聞報導的主題是什麼。
- 決定詞bank給定的出現是用來指河的坡岸、一個金融機構、向一邊傾斜的動作還是在金融機構裡的存儲行為。

## 性別鑑定
男性和女性的名字有一些鮮明的特點。以a，e和i結尾的很可能是女性，而以k，o，r，s和t結尾的很可能是男性。讓我們建立一個分類器更精確地模擬這些差異。

建立一個函數，回傳輸入名字的最後一個字母

```python
>>> def  gender_features (word):
...     return { 'last_letter' : word[-1]}
>> gender_features( 'Shrek' )
 {'last_letter': 'k'}
```
這個函數返回的字典被稱為特徵集，映射特徵名稱到它們的值。

接下來，我們使用特徵提取器處理names數據，並劃分特徵集的結果鍊錶為一個訓練集和一個測試集。訓練集用於訓練一個新的“樸素貝葉斯”分類器。

```python
>>> featuresets = [(gender_features(n), gender) for (n, gender) in labeled_names]
>>> train_set, test_set = featuresets[500:], featuresets[:500]
>>> classifier = nltk.NaiveBayesClassifier. train(train_set)
```

使用剛建立的訓練集進行測試

```python
>>> print(classifier.classify(gender_features( 'Neo' )))
>>> print(classifier.classify(gender_features( 'Trinity' )))
male
female
```

將做出來的訓練集由測試集測試得出評分
```python
>>> print (nltk.classify.accuracy(classifier, test_set))
 0.77
```

找出最有效的特徵
```python
>>> classifier.show_most_informative_features(5)
Most Informative Features 
             last_letter = 'a' female : male = 33.2 : 1.0 
             last_letter = 'k' male : female = 32.6 : 1.0 
             last_letter = 'p' male : female = 19.7 : 1.0 
             last_letter = 'v' male : female = 18.6 : 1.0 
             last_letter = 'f' male : female = 17.3 : 1.0
```
在處理大型語料庫時，構建一個包含每一個實例的特徵的單獨的列表會使用大量的內存。

在這些情況下，使用函數nltk.classify.apply_features，返回一個行為像一個列表而不會在內存存儲所有特徵集的對象：
```python
>>> from nltk.classify import apply_features
>>> train_set = apply_features(gender_features, labeled_names[500:])
>>> test_set = apply_features(gender_features, labeled_names[:500])
```

## 選擇正確的特徵
特徵提取通過反複試驗和錯誤的過程建立的，由哪些信息是與問題相關的直覺指引的。

它通常以“廚房水槽”的方法開始，包括你能想到的所有特徵，然後檢查哪些特徵是實際有用的。

我們在名字性別特徵採取這種做法。
```python
def  gender_features2 (name): 
    features = {} 
    features[ "first_letter" ] = name[0].lower() 
    features[ "last_letter" ] = name[-1].lower()
    for letter in  'abcdefghijklmnopqrstuvwxyz' : 
        features[ "count({})" .format(letter)] = name.lower().count(letter) 
        features[ "has({})" .format(letter)] = (letter in name.lower()) 
    return features
```

然而，你要用於一個給定的學習算法的特徵的數目是有限的——如果你提供太多的特徵，那麼該算法將高度依賴你的訓練數據的特性，而一般化到新的例子的效果不會很好。
這個問題被稱為過擬合，當運作在小訓練集上時尤其會有問題。

我們給了資料更豐富的訓練集，精度卻比起只提供字尾降低了
```python
>>> featuresets = [(gender_features2(n), gender) for (n, gender) in labeled_names]
>>> train_set, test_set = featuresets[500:], featuresets[:500]
>>> classifier = nltk.NaiveBayesClassifier. train(train_set)
>>> print (nltk.classify.accuracy(classifier, test_set))
0.768
```
一旦初始特徵集被選定，完善特徵集的一個非常有成效的方法是錯誤分析。
首先，我們選擇一個開發集，包含用於創建模型的語料數據。然後將這種開發集分為訓練集和開發測試集。
```python
>>> train_names = labeled_names[1500:]
>>> devtest_names = labeled_names[500:1500]
>>> test_names = labeled_names[:500]
```
```python
>>> train_set = [(gender_features(n), gender) for (n, gender) in train_names]
>>> devtest_set = [(gender_features(n), gender) for (n, gender) in devtest_names]
>>> test_set = [(gender_features(n), gender) for (n, gender) in test_names]
>>> classifier = nltk.NaiveBayesClassifier.train(train_set) 
>>> print (nltk.classify.accuracy(classifier, devtest_set))
0.756
```

