練習蒐集資料
==

想練習分析中文文章，但現在還沒找到中文語料庫，先自己抓
今天時間有點趕，code特別爛

- 爬取中央社新聞

使用requests，BeautifulSoup
```python
import re
from bs4 import BeautifulSoup
import requests
import html5lib
```
先找出新聞標題的網址
```python
url = 'https://www.cna.com.tw/list/aspt.aspx'
res = requests.get(url)
soup = BeautifulSoup(res.text,'html5lib')
n=0
labels=[]
while n < len(soup.select_one('#pnProductNavContents > ul').findAll('li')):
    try :
        lab = soup.select_one('#pnProductNavContents > ul').findAll('li')[n].find('a',{'class':'first-level'}).text
        url =soup.select_one('#pnProductNavContents > ul').findAll('li')[n].find('a',{'class':'first-level'})['href']
    except:(TypeError,IndentationError)
    labels.append((url,lab))
    n += 1
labels = list(set(labels))
labels.remove(('http://cnavideo.cna.com.tw', '影音'))
labels.remove(('/list/aall.aspx', '即時'))
labels.append(('https://www.cna.com.tw/list/aall.aspx', '即時'))

>>> print(labels)

[('https://www.cna.com.tw/list/asc.aspx', '證券'),
 ('https://www.cna.com.tw/list/asoc.aspx', '社會'),
 ('https://www.cna.com.tw/list/aopl.aspx', '國際'),
 ('https://www.cna.com.tw/list/aloc.aspx', '地方'),
 ('https://www.cna.com.tw/keyword/九合一.aspx', '九合一'),
 ('https://www.cna.com.tw/list/ait.aspx', '科技'),
 ('https://www.cna.com.tw/list/aspt.aspx', '運動'),
 ('https://www.cna.com.tw/list/sp.aspx', '專題'),
 ('https://www.cna.com.tw/list/aipl.aspx', '政治'),
 ('https://www.cna.com.tw/list/ahel.aspx', '生活'),
 ('https://www.cna.com.tw/list/acul.aspx', '文化'),
 ('https://www.cna.com.tw/list/amov.aspx', '娛樂'),
 ('https://www.cna.com.tw/list/aie.aspx', '產經'),
 ('https://www.cna.com.tw/list/acn.aspx', '兩岸'),
 ('https://www.cna.com.tw/list/aall.aspx', '即時')]
```

接著使用這些網址找出所有的文章

這裡方法有問題只能抓到較近期的新聞，有空再調整
```python
labs_soup={}
for item in labels:
    res = requests.get(item[0])
    soup = BeautifulSoup(res.text,'html5lib')
    labs_soup[item[1]]=soup
lab_urls = {}
m = 0
while m < len(list(labs_soup.keys())):
    lab_name = list(labs_soup.keys())[m]
    soup = labs_soup[lab_name]

    n = 0
    while n < len(soup.findAll('div',{'class':'statement'})[0].findAll('ul',{'class':'mainList imgModule'})[0].findAll('li')):
        lab_url = soup.findAll('div',{'class':'statement'})[0].findAll('ul',{'class':'mainList imgModule'})[0].findAll('li')[n].find_all('a')[0]['href']

        lab_urls[lab_url] = lab_name
        n += 1
    m += 1

>>> print(lab_urls)

{'https://www.cna.com.tw/news/acul/201811010396.aspx': '即時',
 'https://www.cna.com.tw/news/acul/201811010353.aspx': '即時',
 'https://www.cna.com.tw/news/acul/201811010337.aspx': '即時',
 'https://www.cna.com.tw/news/firstnews/201811010298.aspx': '即時',
 'https://www.cna.com.tw/news/acul/201811010203.aspx': '文化',
 'https://www.cna.com.tw/news/acul/201811010174.aspx': '文化',
 'https://www.cna.com.tw/news/acul/201811010162.aspx': '文化',
 'https://www.cna.com.tw/news/acul/201811010142.aspx': '文化',
```
這就是所有新聞的網址了，再來就是再連進去抓內文出來

```python
news={}
for item in list(lab_urls.items()):
    res = requests.get(item[0])
    soup = BeautifulSoup(res.text,'html5lib')
    txt = ""
    try:
        for s in soup.find_all('div',{'class':'paragraph'})[0].findAll('p'):
            txt=txt+s.getText()
    except:IndexError
    news[txt]=item[1]
>>> print(news)

"""
{'（中央社記者田裕斌台北1日電）仁寶集團旗下網通廠鋐寶科技今天舉行上市前業績發表會，預計11月下旬掛牌上市，鋐寶看準5G通訊及智慧家庭商機，今年獲利將大幅成長。鋐寶主要營業項目包括研究、開發及銷售智慧閘道器、數位機上盒及無線寬頻分享器等通訊產品，目前全球市占率約10%，主要客戶為歐洲市場，歐洲前幾大MSO運營商等皆為長期合作客戶，在歐洲市場的市占率更高達40%，憑藉著位居歐洲領先地位的基礎上，鋐寶近年來積極開拓美洲市場，期盼進一步擴大全球市占率。根據IHS市場分析報告，線纜用戶端設備(Cable CPE)年產值約32.3億美元、年出貨量4400萬台，且每年持續有2%至3%成長，成長動能來自於Cable CPE市場MSO運營商不斷推出各式加值服務，強化使用者介面、高速上網、整合家庭網絡和高可靠度的監控服務，持續帶動整合語音、資料、影音、家庭聯網及高速上網需求。法人分析，鋐寶藉由近年歐美寬頻網路建設逐漸成熟，多元化網路服務和智慧家庭市場對於Cable CPE 需求隨之提升，在網路頻寬通訊設備與資訊家電產品結合，應用範圍日漸廣泛下，未來營運成長動能可期。（編輯：李信寬）1071101': '即時',
 '（中央社香港1日綜合外電報導）美國股市昨天收盤連續第2天勁揚，加上北京政府承諾支持全球第二大經濟體中國，亞洲股市今天大多應聲收紅。北京提供今天亞股上漲動力。中國領導當局表示，由於國內接連出現疲軟的經濟數據，包括第3季經濟成長率降至9年來最低水準，當局將推出新措施來提振步履蹣跚的經濟。中共中央政治局昨天召開會議，由總書記習近平主持。會後發布聲明表示，中國今年第1至第3季「經濟運行總體平穩，穩中有進」，但有必要採取更多措施來協助民間企業。聲明還說：「對此要高度重視，增強預見性，及時採取對策。」中國國務院昨天也發布指導意見，要求「進一步增強基礎設施對促進城鄉和區域協調發展、改善民生等方面的支撐作用」，以促進脫貧和改善鐵公路、空運和能源等領域的弱點。上海股市今天收盤上漲0.1%，香港股市收高1.8%，新加坡股市收升1.4%，雪梨股市收漲0.2%，威靈頓股市收高1.1%，台北股市收升0.4%。不過，東京股市今天收盤下挫1.1%，原因是日圓走強，且昨天日股彈升超過2%，吸引投資人獲利回吐。首爾股市今天則開高走低，收跌0.3%。（譯者：張正芊/核稿：陳政一）1071101': '即時',
 '（中央社記者張建中新竹1日電）被動元件廠信昌電與保護元件廠佳邦第3季獲利同步繳出不錯成績單，信昌電第3季每股純益達新台幣3.04元，佳邦每股純益1.27元。受惠市場需求強勁，產品價格走揚，被動元件廠第3季營運普遍繳出亮麗成績單，國內被動元件龍頭廠國巨第3季即賺進超過3個股本。信昌電第3季營收達19.03億元，季增32%，毛利率也攀高至48.13%，較第2季拉升4.84個百分點，稅後淨利5.23億元，季增14.3%，每股純益3.04元；前3季每股純益達6.57元。佳邦第3季稅後淨利1.83億元，更較第2季大增2.48倍，每股純益1.27元；前3季每股純益2.09元，每股淨值達32.41元。（編輯：李信寬）1071101': '即時',
 """
```
格式是{ (內文,類別), ...

之後就先用這些練習中文

參考資料:中央通訊社<https://www.cna.com.tw/>
