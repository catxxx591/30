中文文本分類練習
==
首先先把昨式爬蟲有問題的部分改好

自己定義名稱常常忘記是什麼type或重複命名，搞了很多bug，改好後還是滿亂的，就直接貼結果上來
```python
news={}
for lab in labs_soup:
    lab_name = lab[1]
    lab_urls = lab[0]
    n = 0
    for url in lab_urls:
        res = requests.get(url)
        soup = BeautifulSoup(res.text,'html5lib')
        txt = ""
        try:
            for s in soup.find_all('div',{'class':'paragraph'})[0].findAll('p'):
                txt=txt+s.getText()
        except:IndexError
        news[txt]=lab_name
        n += 1
        if n % 20 == 0:
            print((lab_name,n),end=",")

('地方', 20),('地方', 40),('地方', 60),('地方', 80),('地方', 100),('專題', 20),('專題', 40),('專題', 60),('產經', 20),('產經', 40),('產經', 60),('產經', 80),('產經', 100),('運動', 20),('運動', 40),('運動', 60),('運動', 80),('運動', 100),('社會', 20),('社會', 40),('社會', 60),('社會', 80),('社會', 100),('生活', 20),('生活', 40),('生活', 60),('生活', 80),('生活', 100),('兩岸', 20),('兩岸', 40),('兩岸', 60),('兩岸', 80),('兩岸', 100),('國際', 20),('國際', 40),('國際', 60),('國際', 80),('國際', 100),('證券', 20),('證券', 40),('證券', 60),('證券', 80),('證券', 100),('政治', 20),('政治', 40),('政治', 60),('政治', 80),('政治', 100),('科技', 20),('科技', 40),('科技', 60),('科技', 80),('科技', 100),('文化', 20),('文化', 40),('文化', 60),('文化', 80),('文化', 100),('娛樂', 20),('娛樂', 40),('娛樂', 60),('娛樂', 80),('娛樂', 100),('即時', 20),('即時', 40),('即時', 60),('即時', 80),('即時', 100),('即時', 120),('即時', 140),('即時', 160),('即時', 180),('即時', 200),('即時', 220),('即時', 240),('即時', 260),('即時', 280),('即時', 300),('即時', 320),('即時', 340),('即時', 360),('即時', 380),
```

這裡能大概看看各類文章的比例，網站上除了即時新聞外，最多只能找到100筆

結構大概長這樣
```python
>>> books = list(news.items())
>>> print(books[0])

"""
('（中央社澎湖縣2日電）為帶動婚禮經濟相關產業朝向國際觀光勝地及海島婚禮聖地的理想前進，澎湖縣府舉行「愛在澎湖」成果發表，盼澎湖成為新人拍攝婚紗與舉行婚禮的最佳選擇，留下幸福美好的回憶。澎湖縣政府今天下午在馬公第一漁港遊艇碼頭推出「愛在澎湖」成果發表，提供有戀愛小巴與愛之船兩車兩船，讓新人能在世界最美麗海灣的澎湖，到處趴趴走，拍攝一張張美麗的婚紗照，留下永恆。來自台北、即將結婚的陳姓男子與胡姓女子，今天搶得頭彩，為其中一對搭上戀愛小巴與愛之船的新人，也對澎湖海灣的美深深感動。澎湖縣府旅遊處表示，進入秋冬的結婚旺季，澎湖除有美麗的海灘、陽光、沙灘、海浪等美景外，今年澎湖還舉行世界最美麗海灣組織年會，加上交通方便，不用擔心語言或風俗不同，澎湖將成為最佳新人的海島婚禮聖地，新人可選擇到澎湖拍攝婚紗和跳島旅拍小蜜月，令人感到幸福美好。（編輯：陳政偉）1071102',
 '即時')
"""
```

接著就開始嘗試分類，先使用nltk找出高頻單字的方法。

我先把文章整理成類似brown語料庫Allwords的結構
```python
lab_tags ={}
txt =''
for book in books:
    n = 0
    while n < 14:
        if book[1]==labs[n][1]:
            txt = txt+book[0]
            lab_tags[book[1]] = txt
        n += 1
>>> print(lab_tags['運動'])
"""
"（中央社澎湖縣2日電）為帶動婚禮經濟相關產業朝向國際觀光勝地及海島婚禮聖地的理想前進，澎湖縣府舉行「愛在澎湖」成果發表，盼澎湖成為新人拍攝婚紗與舉行婚禮的最佳選擇，留下幸福美好的回憶。澎湖縣政府今天下午在馬公第一漁港遊艇碼頭推出「愛在澎湖」成果發表，提供有戀愛小巴與愛之船兩車兩船，讓新人能在世界最美麗海灣的澎湖，到處趴趴走，拍攝一張張美麗的婚紗照，留下永恆。來自台北、即將結婚的陳姓男子與胡姓女子，今天搶得頭彩，為其中一對搭上戀愛小巴與愛之船的新人，也對澎湖海灣的美深深感動。澎湖縣府旅遊處表示，進入秋冬的結婚旺季，澎湖除有美麗的海灘、陽光、沙灘、海浪等美景外，今年澎湖還舉行世界最美麗海灣組織年會，加上交通方便，不用擔心語言或風俗不同，澎湖將成為最佳新人的海島婚禮聖地，新人可選擇到澎湖拍攝婚紗和跳島旅拍小蜜月，令人感到幸福美好。（編輯：陳政偉）1071102（中央社記者蘇木春台中2日電）華陶窯明天起至12月2日在台中市港區藝術中心舉辦「花陶生活文化美-流域文化美璞展」，精選9名藝術家的42組柴燒創作，展現台灣流域聚落文化豐沛的人文特色。位於苗栗縣苑裡鎮的華陶窯成立於民國73年，窯主陳文輝堅持傳統柴燒登窯、民俗製陶技法與相思木合而為一的技術，完整呈現柴燒火痕在坯體與落灰結合，成為自然灰釉的質樸美。美展精選華陶窯成立34年來9名藝術家、共42組作品，分為入口意象、人文花陶暨創作材質文件展、花陶流域文化裝置展等系列，走進展場就能見到竹架搭建的竹林，重現華陶窯園區的入口大門景致，名為「浣花」的大提樑壺作品，將古樸與當代陶藝色彩完整呈現。窯主陳文輝受訪表示，展覽的「流域」理念，是指水系幹流、支流所流經的區域，台灣最豐沛的人文，是祖先由西邊渡海來台，溯溪往東拓墾，使閩南、客家與原住民文化碰撞相融，形成的流域聚落文化，展覽完整呈現漂流木、大甲東陶與藺草三大瑰寶。策展人陳文榮說，華陶窯成立30多年來，曾駐園的藝術家逾50人，園區景觀不只是園藝，讓藝術家在陶藝創作中，將自然生態的四季變化都融入其中，使作品反映出在地的文化質感。場內也展示多幅巨型花卉照片牆，都是華陶窯內原生種植物，陳文榮說，窯主陳文輝的夫人陳玉秀是中華花藝老師，因製作花器需求，窯主開始在後山開闢植物園，復育超過700種原生植物，也成為園區重要資產。藝術家陳美華創作地景系列複合媒材作品，將不鏽鋼融入陶器展現獨特風貌，她表示，因喜愛單車運動，平時上山下海發現台灣有很多美景，她將粗曠的三個陶器表現為山景，融入不鏽鋼光澤，藉此衍生為湖面感。她說，地景系列拆開來也是獨立作品，可以拿來當作茶席，翻過來還能當作插花器皿，將陶藝融入生活，不只是表達她內心對台灣美景的感動，也可以是生活中實用的器具。（編輯：翁翠萍）1071102（中央社記者趙麗妍台中2日電）台中世界花卉博覽會明天開幕，建置於后里森林園區的永續發展家園「四口之家」獲西元2018年倫敦設計獎銀獎，為台中花博增添光彩。策展人劉德輔今天開心表示，「四口之家」由7個團隊、10多名志工完成，規劃、施作、策展遭遇許多困難。為完成計畫，一行人還前往德國...
"""
```
lab_tags的keys將會對應該分類所有文章的字串在values

在來使用jieba斷詞做出類似nltk的tokens，在使用nltk算出頻率
```python
>>> lab_words = lab_tags['運動']
>>> seg_list = jieba.cut(lab_words, cut_all=False)
>>>　txt = ",".join(seg_list)
>>> tokens = txt.split(',')
>>> print(nltk.FreqDist(tokens).most_common(100))

[('，', 9226),
 ('。', 2587),
 ('的', 2545),
 ('、', 1729),
 ('（', 1111),
 ('）', 1110),
 ('在', 1069),
 ('「', 812),
 ('」', 812),
 ('也', 587),
 (' ', 575),
 ('與', 551),
 ('是', 515),
 ('他', 496),
 ('：', 494),
 ('1', 478),
 ('2', 472),
 ('有', 469),
 ('表示', 438),
 ('今天', 437),
 ('年', 406),
 ('後', 405),
 ('3', 393),
 ('月', 368),
 ('為', 328),
 ('說', 324),
 ('中央社', 318),
 ('第', 312),
 ('對', 295),
 ('都', 293),
 ('記者', 290),
 ('4', 288),
 ('於', 278),
 ('編輯', 267),
 ('和', 266),
 ('日電', 254),
 ('會', 254),
 ('比', 250),
 ('陳', 249),
 ('但', 243),
 ('讓', 237),
 ('等', 237),
 ('以', 235),
 ('日', 234),
 ('中', 227),
 ('台灣', 224),
 ('；', 220),
 ('長', 219),
 ('到', 215),
 ('上', 214),
 ('就', 200),
 ('分', 199),
 ('總冠軍賽', 197),
 ('人', 196),
 ('來', 195),
 ('及', 195),
 ('10', 193),
 ('台北', 188),
 ('今年', 188),
 ...
```

可以發現結果有點不知所云，很多結果就是nltk教學提到的停止詞，要先套入nltk內的停止詞字典把他們刪除，但nltk沒有中文字典。

在網上有看到中文的停止詞字典，明天再試試看。可能還需要常見地名和人名的字典才能更好，例如，提到某部長名字就很有可能是政治新聞，有某選手名字就很可能是運動新聞