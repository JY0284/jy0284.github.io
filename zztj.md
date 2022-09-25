---
title: "偷书能算偷？《资治通鉴》文白对照版爬取记录"
date: 2019-09-17T21:52:27+08:00
draft: false
tags: ["爬虫", "BeautifulSoup", "python"]
categories: ["记录", "爬虫"]
---


# 前言

最近几个月都在读《资治通鉴》，有时候一些地方看文言文不明白，就会翻书或用搜索引擎查找对应的翻译。但无论是翻书还是用搜索引擎，还是有些麻烦，要么需要特定版本的书在手边，要么就是需要在许多搜索结果中选择。另一方面，我也想要之后能够搜索一句就能直接看到翻译好的整段，以提高阅读效率和体验。

恰好，最近在查翻译时发现了一个很棒的网站展示了《资治通鉴》比较好的文言文白话文对照文本，索性利用一点爬虫技术将完整的对照文本取到本地，便于之后的阅读。

这篇文章是用`jupyter notebook`边想边做边写的，所以思路比较连贯。汇总的`python`程序在文章的最后，大概60行左右。希望读者不要恶意使用爬虫相关的技术，并在符合当地法律的情形下使用相关的技术，以免给他人和自己带来不便。

“偷书怎么能算偷呢？”

# 搞起

## 1.工具库准备、目录分析、卷链接提取

如果没有安装过beautifulsoup库，需要先行安装
```shell
pip install beautifulsoup4 --user
```


```python
# 引入网络请求和页面解析所需要的库
from urllib.request import urlopen
from bs4 import BeautifulSoup
```


```python
# 目录链接
content_url = 'http://www.shuzhai.org/gushi/tongjian/'
```

目录页（也就是上面的`content_url`）如下图所示。可以看到《资治通鉴》全部294卷的链接都在这个页面上。但与此同时，该页面上也存在大量其他链接。

![目录页](https://s2.ax1x.com/2019/09/17/nIUcTJ.md.png)

读者也可以自己访问目录页面 [目录页](http://www.shuzhai.org/gushi/tongjian/)。

首先使用`beautifulsoup`来读取页面，使得页面的元素易于提取。


```python
html = urlopen(content_url)
bs = BeautifulSoup(html.read(), "html.parser")
```

紧接着查看一下所有的链接（`html`中的`<a>`标签)元素，有一个宏观的认识。


```python
bs.findAll("a")
```




    [<a href="http://www.shuzhai.org/gushi/" title="古诗">古诗赏析</a>,
     <a href="http://www.shuzhai.org/gushi/guwen/" title="古文典籍">古文典籍</a>,
     <a href="http://www.shuzhai.org/gushi/wenyanwen/" title="文言文">文言翻译</a>,
     <a href="http://www.shuzhai.org/gushi/tang/">唐诗三百</a>,
     
     …………（省略若干链接）

     <a href="http://www.shuzhai.org/gushi/tongjian/7580.html" target="_blank" title="资治通鉴第二百七十一卷(后梁纪)">资治通鉴第二百七十一卷(后梁纪)</a>,
     <a href="http://www.shuzhai.org/gushi/tongjian/7579.html" target="_blank" title="资治通鉴第二百七十卷(后梁纪)">资治通鉴第二百七十卷(后梁纪)</a>,
     <a href="http://www.shuzhai.org/gushi/tongjian/7578.html" target="_blank" title="资治通鉴第二百六十九卷(后梁纪)">资治通鉴第二百六十九卷(后梁纪)</a>,
     <a class="bds_tsina">新浪微博</a>,
     <a class="bds_qzone">QQ空间</a>,
     <a class="bds_tqq">腾讯微博</a>,
     <a class="bds_baidu">百度搜藏</a>,
     <a class="bds_douban">豆瓣网</a>,
     <a class="shareCount"></a>,
     <a href="http://www.shuzhai.org/gushi/tongjian/6134.html" target="_blank">第一卷</a>,
     <a href="http://www.shuzhai.org/gushi/tongjian/6135.html" target="_blank">第二卷</a>,
     <a href="http://www.shuzhai.org/gushi/tongjian/6136.html" target="_blank">第三卷</a>,
     <a href="http://www.shuzhai.org/gushi/tongjian/6137.html" target="_blank">第四卷</a>,
     <a href="http://www.shuzhai.org/gushi/tongjian/6138.html" target="_blank">第五卷</a>,
     <a href="http://www.shuzhai.org/gushi/tongjian/6139.html" target="_blank">第六卷</a>,
     <a href="http://www.shuzhai.org/gushi/tongjian/6140.html" target="_blank">第七卷</a>,
     <a href="http://www.shuzhai.org/gushi/tongjian/6141.html" target="_blank">第八卷</a>,
     <a href="http://www.shuzhai.org/gushi/tongjian/6142.html" target="_blank">第九卷</a>,
     <a href="http://www.shuzhai.org/gushi/tongjian/6143.html" target="_blank">第十卷</a>,
     <a href="http://www.shuzhai.org/gushi/tongjian/6144.html" target="_blank">第十一卷</a>,
     
     …………（省略若干链接）

     <a href="http://www.shuzhai.org/gushi/soushenji/" target="_blank">搜神记</a>,
     <a href="http://www.shuzhai.org/gushi/xiaojing/" target="_blank">孝经</a>,
     <a href="http://www.shuzhai.org/gushi/liutao/" target="_blank">六韬</a>,
     <a href="http://www.mindhave.com/dushubiji/" target="_blank">读书笔记</a>,
     <a href="http://www.shuzhai.org/" target="_blank"> www.shuzhai.org</a>,
     <a href="http://www.shuzhai.org" target="_blank" title="书摘天下">书摘天下</a>]



可见，有大量链接充斥在这个页面上，第一时间考虑使用**正则表达式**匹配需求的全部294卷所指向的链接。


```python
# 引入正则表达式库
import re
```


```python
target_links = bs.findAll("a", {"href": re.compile(".*gushi\/tongjian\/.*html"), "title":None})
len(target_links)
```




    294



好！果真提取出了恰好294个链接，为了稳妥起见，还是抽查一下最后5个链接是不是所求。


```python
target_links[-5:]
```




    [<a href="http://www.shuzhai.org/gushi/tongjian/7612.html" target="_blank">第二百九十卷</a>,
     <a href="http://www.shuzhai.org/gushi/tongjian/7613.html" target="_blank">第二百九十一卷</a>,
     <a href="http://www.shuzhai.org/gushi/tongjian/7614.html" target="_blank">第二百九十二卷</a>,
     <a href="http://www.shuzhai.org/gushi/tongjian/7615.html" target="_blank">第二百九十三卷</a>,
     <a href="http://www.shuzhai.org/gushi/tongjian/7616.html" target="_blank">第二百九十四卷</a>]



正是二百九十卷至二百九十四卷，稳。

顺便提取链接文本看一下，之前都是`html`的`<a>`标签。


```python
target_links[0].attrs['href']
```




    'http://www.shuzhai.org/gushi/tongjian/6134.html'



## 2.卷页面分析、卷页面上有用链接提取

顺手取第一个链接作为分析模板，一般情况下，这样的网站不同卷的页面应该不会有结构差别。


```python
target_url = target_links[0].attrs['href']
```

和之前一样的处理：


```python
html = urlopen(target_url)
bs = BeautifulSoup(html.read())
```

看一下卷页面的样子，如下图。

![卷页面上部](https://s2.ax1x.com/2019/09/17/nIBOQ1.md.png)
![卷页面下部](https://s2.ax1x.com/2019/09/17/nIDkQI.md.png)

看看发现了什么！从卷页面的下部可以看到，每一卷还分了多个页面，也就是说，我们在下面的工作中需要将每一卷的分页链接全部提取出来。

简单看一下卷页面上全部链接的情况：


```python
bs.findAll("a")
```




    [<a href="http://www.shuzhai.org/gushi/" title="古诗">古诗赏析</a>,
     <a href="http://www.shuzhai.org/gushi/guwen/" title="古文典籍">古文典籍</a>,
     <a href="http://www.shuzhai.org/gushi/wenyanwen/" title="文言文">文言翻译</a>,
     <a href="http://www.shuzhai.org/gushi/tang/">唐诗三百</a>,
     <a href="http://www.shuzhai.org/gushi/song/">宋词三百</a>,
     <a href="http://www.shuzhai.org/gushi/yuan/">元曲精选</a>,
     <a href="http://www.shuzhai.org/gushi/mingju/">诗词名句</a>,
     <a href="http://www.shuzhai.org/dict/zaoju/">词语大全</a>,
     <a href="http://yuedu.shuzhai.org/" title="语文阅读">语文阅读</a>,
     <a href="http://www.shuzhai.org/duhougan/" title="读后感">读后感</a>,
     <a href="http://www.shuzhai.org/dict/chengyu/" title="成语大全">成语大全</a>,

     …………（省略若干链接）

     <a href="http://www.shuzhai.org/gushi/qiu/" style=" width:80px;">秋天诗句</a>,
     <a href="http://www.shuzhai.org/gushi/yu/" style=" width:80px;">写雨诗句</a>,
     <a href="http://www.shuzhai.org/gushi/xue/" style=" width:80px;">写雪诗句</a>,
     <a href="http://www.shuzhai.org/gushi/feng/" style=" width:80px;">写风诗句</a>,
     <a href="http://www.shuzhai.org/gushi/hua/" style=" width:80px;">写花诗句</a>,

     …………（省略若干链接）

     <a href="http://www.shuzhai.org/gushi/tongjian/6134.html" target="_blank">http://www.shuzhai.org/gushi/tongjian/6134.html</a>,
     <a>共11页: </a>,
     <a href="#">上一页</a>,
     <a href="#">1</a>,
     <a href="6134_2.html">2</a>,
     <a href="6134_3.html">3</a>,
     <a href="6134_4.html">4</a>,
     <a href="6134_5.html">5</a>,
     <a href="6134_6.html">6</a>,
     <a href="6134_7.html">7</a>,
     <a href="6134_8.html">8</a>,
     <a href="6134_9.html">9</a>,
     <a href="6134_10.html">10</a>,
     <a href="6134_11.html">11</a>,
     <a href="6134_2.html">下一页</a>,
     <a href="http://www.shuzhai.org/gushi/tongjian/6135.html">资治通鉴第二卷(周纪)</a>,
     <a class="bds_tsina">新浪微博</a>,
     <a class="bds_qzone">QQ空间</a>,

     …………（省略若干链接）

     <a href="http://www.shuzhai.org/gushi/tongjian/7613.html" target="_blank" title="资治通鉴第二百九十一卷(后周纪)">资治通鉴第二百九十一卷(后周纪)</a>,
     <a href="http://www.shuzhai.org/" target="_blank"> www.shuzhai.org</a>,
     <a href="http://www.shuzhai.org" target="_blank" title="书摘天下">书摘天下</a>]



类似上面的套路，页面链接的模式很明显（由数字和下划线组成），正则表达式走起：


```python
page_links = bs.findAll("a", {"href": re.compile("\d+_\d+\.html")})
page_links
```




    [<a href="6134_2.html">2</a>,
     <a href="6134_3.html">3</a>,
     <a href="6134_4.html">4</a>,
     <a href="6134_5.html">5</a>,
     <a href="6134_6.html">6</a>,
     <a href="6134_7.html">7</a>,
     <a href="6134_8.html">8</a>,
     <a href="6134_9.html">9</a>,
     <a href="6134_10.html">10</a>,
     <a href="6134_11.html">11</a>,
     <a href="6134_2.html">下一页</a>]



为了防止出现重复链接导致的无限递归查找，使用**集合**以保证每个链接只会出现1次：


```python
pages = set()
for link in page_links:
    pages.add(content_url + link.attrs["href"])

```

好！每一卷里所有页的链接提取完毕。

接下来只要把所有294卷对应的所有页的链接都做相同的处理就好。具体提取正文的处理，就在下一节。

## 3.提取正文！

让我们看一下这个页面的`body`是什么样子：


```python
text = bs.body.text
text
```




    '\n古诗赏析 | 古文典籍 | 文言翻译 |  | 唐诗三百 | 宋词三百 |  | 元曲精选 |  | 诗词名句 |  | 词语大全 | 语文阅读 | 读后感 | 成语大全 | 词典 | 故事会 | 思想汇报 | 工作总结\n\n\n \n\n \n\n\n\n\n\n\n首页\n唐代\n宋代\n元代\n明代\n清代\n五代\n金朝\n诗人\n全宋词\n名句\n诗词试题\n\n\n\n\n\n\n\n\n书摘天下 > 古诗宋词 > 古文典籍 > 资治通鉴 > \n资治通鉴第一卷(周纪)\n\n作者 ： 司马光\u3000\u3000\u3000\u3000网址 ： www.shuzhai.org 时间 ： 2015-12-12    整理 ： 古诗文网\n\n\n\n\n\n类型\n\n\n写景诗句\n写黄河诗句\n写儿童诗句\n写鸟诗句\n写马诗句\n爱国诗句\n励志古诗词\n忧国忧民\n哲理诗\n田园诗\n送别诗\n爱情诗\n友情的古诗\n古诗十九首\n咏物诗\n元宵节诗句\n写长江诗句\n写水诗句\n春天诗句\n春节诗句\n夏天诗句\n冬天诗句\n秋天诗句\n写雨诗句\n写雪诗句\n写风诗句\n写花诗句\n写梅花诗句\n写荷花诗句\n写菊花诗句\n写柳树诗句\n写月亮诗句\n写山诗句\n清明诗句\n\n\n\n\n\n朝代\n\n\n唐代古诗\n宋代古诗\n金朝古诗\n元代古诗\n明代古诗\n清代古诗\n文言翻译\n\n\n\n\r\n资治通鉴第一卷(周纪)\n\r\n周纪一 威烈王二十三年（戊寅、前403）\r\n\u3000\u3000周纪一 周威烈王二十三年（戊寅，公元前403年）\r\n\u3000\u3000[1]初命晋大夫魏斯、赵籍、韩虔为诸侯。\r\n\u3000\u3000[1]周威烈王姬午初次分封晋国大夫魏斯、赵籍、韩虔为诸侯国君。\r\n\u3000\u3000臣光曰：臣闻天子之职莫大于礼，礼莫大于分，分莫大于名。何谓礼？纪纲是也。何谓分？君、臣是也。何谓名？公、侯、卿、大夫是也。\r\n\u3000\u3000臣司马光曰：我知道天子的职责中最重要的是维护礼教，礼教中最重要的是区分地位，区分地位中最重要的是匡正名分。什么是礼教？就是法纪。什么是区分地位？就是君臣有别。什么是名分？就是公、侯、卿、大夫等官爵。\r\n\u3000\u3000夫以四海之广，兆民之众，受制于一人，虽有绝伦之力，高世之智，莫不奔走而服役者，岂非以礼为之纪纲哉！是故天子统三公，三公率诸侯，诸侯制卿大夫，卿大夫治士庶人。贵以临贱，贱以承贵。上之使下犹心腹之运手足，根本之制支叶，下之事上犹手足之卫心腹，支叶之庇本根，然后能上下相保而国家治安。故曰天子之职莫大于礼也。\r\n\u3000\u3000四海之广，亿民之众，都受制于天子一人。尽管是才能超群、智慧绝伦的人，也不能不在天子足下为他奔走服务，这难道不是以礼作为礼纪朝纲的作用吗！所以，天子统率三公，三公督率诸侯国君，诸侯国君节制卿、大夫官员，卿、大夫官员又统治士人百姓。权贵支配贱民，贱民服从权贵。上层指挥下层就好像人的心腹控制四肢行动，树木的根和干支配枝和叶；下层服侍上层就好像人的四肢卫护心腹，树木的枝和叶遮护根和干，这样才能上下层互相保护，从而使国家得到长治久安。所以说，天子的职责没有比维护礼制更重要的了。\r\n\u3000\u3000文王序《易》，以乾、坤为首。孔子系之曰：“天尊地卑，乾坤定矣。卑高以陈，贵贱位矣。”言君臣之位犹天地之不可易也。《春秋》抑诸侯，尊王室，王人虽微，序于诸侯之上，以是见圣人于君臣之际未尝不也。非有桀、纣之暴，汤、武之仁，人归之，天命之，君臣之分当守节伏死而已矣。是故以微子而代纣则成汤配天矣，以季札而君吴则太伯血食矣，然二子宁亡国而不为者，诚以礼之大节不可乱也。故曰礼莫大于分也。\r\n\u3000\u3000周文王演绎排列《易经》，以乾、坤为首位。孔子解释说：“天尊贵，地卑微，阳阴于是确定。由低至高排列有序，贵贱也就各得其位。”这是说君主和臣子之间的上下关系就像天和地一样不能互易。《春秋》一书贬低诸侯，尊崇周王室，尽管周王室的官吏地位不高，在书中排列顺序仍在诸侯国君之上，由此可见孔圣人对于君臣关系的关注。如果不是夏桀、商纣那样的暴虐昏君，对手又遇上商汤、周武王这样的仁德明主，使人民归心、上天赐命的话，君臣之间的名分只能是作臣子的恪守臣节，矢死不渝。所以如果商朝立贤明的微子为国君来取代纣王，成汤创立的商朝就可以永配上天；而吴国如果以仁德的季札做君主，开国之君太伯也可以永享祭祀。然而微子、季札二人宁肯国家灭亡也不愿做君主，实在是因为礼教的大节绝不可因此破坏。所以说，礼教中最重要的就是地位高下的区分。\r\n\u3000\u3000夫礼，辨贵贱，序亲疏，裁群物，制庶事，非名不著，非器不形；名以命之，器以别之，然后上下粲然有伦，此礼之大经也。名器既亡，则礼安得独在哉！昔仲叔于奚有功于卫，辞邑而请繁缨，孔子以为不如多与之邑。惟名与器，不可以假人，君之所司也；政亡则国家从之。卫君待孔子而为政，孔子欲先正名，以为名不正则民无所措手足。夫繁缨，小物也，而孔子惜之；正名，细务也，而孔子先之：诚以名器既乱则上下无以相保故也。夫事未有不生于微而成于著，圣人之虑远，故能谨其微而治之，众人之识近，故必待其著而后救之；治其微则用力寡而功多，救其著则竭力而不能及也。《易》曰：“履霜坚冰至，”《书》曰：“一日二日万几，”谓此类也。故曰分莫大于名也。\r\n\u3000\u3000所谓礼教，在于分辨贵贱，排比亲疏，裁决万物，处理日常事物。没有一定的名位，就不能显扬；没有器物，就不能表现。只有用名位来分别称呼，用器物来分别标志，然后上下才能井然有序。这就是礼教的根本所在。如果名位、器物都没有了，那么礼教又怎么能单独存在呢！当年仲叔于奚为卫国建立了大功，他谢绝了赏赐的封地，却请求允许他享用贵族才应有的马饰。孔子认为不如多赏赐他一些封地，惟独名位和器物，绝不能假与他人，这是君王的职权象征；处理政事不坚持原则，国家也就会随着走向危亡。卫国国君期待孔子为他崐处理政事，孔子却先要确立名位，认为名位不正则百姓无所是从。马饰，是一种小器物，而孔子却珍惜它的价值；正名位，是一件小事情，而孔子却要先从它做起，就是因为名位、器物一紊乱，国家上下就无法相安互保。没有一件事情不是从微小之处产生而逐渐发展显著的，圣贤考虑久远，所以能够谨慎对待微小的变故及时予以处理；常人见识短浅，所以必等弊端闹大才来设法挽救。矫正初起的小错，用力小而收效大；挽救已明显的大害，往往是竭尽了全力也\r\n\u3000\u3000不能成功。《易经》说：“行于霜上而知严寒冰冻将至。”《尚书》说：“先\r\n\u3000\u3000王每天都要兢兢业业地处理成千上万件事情。”就是指这类防微杜渐的例子。\r\n \r\n来源栏目：  http://www.shuzhai.org/gushi/tongjian/'



简单分析一波，发现正文总是以**文言翻译**和几个特殊符号开始，以**来源栏目**结束，再一次拿出正则表达式：


```python
text_pattern = r"文言翻译(\n)+(\n|\r)*(.*)来源"
result = re.findall(text_pattern, text, re.DOTALL)[0][2]
print(result)
```

    资治通鉴第一卷(周纪)
    
    周纪一 威烈王二十三年（戊寅、前403）
    　　周纪一 周威烈王二十三年（戊寅，公元前403年）
    　　[1]初命晋大夫魏斯、赵籍、韩虔为诸侯。
    　　[1]周威烈王姬午初次分封晋国大夫魏斯、赵籍、韩虔为诸侯国君。
    　　臣光曰：臣闻天子之职莫大于礼，礼莫大于分，分莫大于名。何谓礼？纪纲是也。何谓分？君、臣是也。何谓名？公、侯、卿、大夫是也。
    　　臣司马光曰：我知道天子的职责中最重要的是维护礼教，礼教中最重要的是区分地位，区分地位中最重要的是匡正名分。什么是礼教？就是法纪。什么是区分地位？就是君臣有别。什么是名分？就是公、侯、卿、大夫等官爵。
    　　夫以四海之广，兆民之众，受制于一人，虽有绝伦之力，高世之智，莫不奔走而服役者，岂非以礼为之纪纲哉！是故天子统三公，三公率诸侯，诸侯制卿大夫，卿大夫治士庶人。贵以临贱，贱以承贵。上之使下犹心腹之运手足，根本之制支叶，下之事上犹手足之卫心腹，支叶之庇本根，然后能上下相保而国家治安。故曰天子之职莫大于礼也。
    　　四海之广，亿民之众，都受制于天子一人。尽管是才能超群、智慧绝伦的人，也不能不在天子足下为他奔走服务，这难道不是以礼作为礼纪朝纲的作用吗！所以，天子统率三公，三公督率诸侯国君，诸侯国君节制卿、大夫官员，卿、大夫官员又统治士人百姓。权贵支配贱民，贱民服从权贵。上层指挥下层就好像人的心腹控制四肢行动，树木的根和干支配枝和叶；下层服侍上层就好像人的四肢卫护心腹，树木的枝和叶遮护根和干，这样才能上下层互相保护，从而使国家得到长治久安。所以说，天子的职责没有比维护礼制更重要的了。
    　　文王序《易》，以乾、坤为首。孔子系之曰：“天尊地卑，乾坤定矣。卑高以陈，贵贱位矣。”言君臣之位犹天地之不可易也。《春秋》抑诸侯，尊王室，王人虽微，序于诸侯之上，以是见圣人于君臣之际未尝不也。非有桀、纣之暴，汤、武之仁，人归之，天命之，君臣之分当守节伏死而已矣。是故以微子而代纣则成汤配天矣，以季札而君吴则太伯血食矣，然二子宁亡国而不为者，诚以礼之大节不可乱也。故曰礼莫大于分也。
    　　周文王演绎排列《易经》，以乾、坤为首位。孔子解释说：“天尊贵，地卑微，阳阴于是确定。由低至高排列有序，贵贱也就各得其位。”这是说君主和臣子之间的上下关系就像天和地一样不能互易。《春秋》一书贬低诸侯，尊崇周王室，尽管周王室的官吏地位不高，在书中排列顺序仍在诸侯国君之上，由此可见孔圣人对于君臣关系的关注。如果不是夏桀、商纣那样的暴虐昏君，对手又遇上商汤、周武王这样的仁德明主，使人民归心、上天赐命的话，君臣之间的名分只能是作臣子的恪守臣节，矢死不渝。所以如果商朝立贤明的微子为国君来取代纣王，成汤创立的商朝就可以永配上天；而吴国如果以仁德的季札做君主，开国之君太伯也可以永享祭祀。然而微子、季札二人宁肯国家灭亡也不愿做君主，实在是因为礼教的大节绝不可因此破坏。所以说，礼教中最重要的就是地位高下的区分。
    　　夫礼，辨贵贱，序亲疏，裁群物，制庶事，非名不著，非器不形；名以命之，器以别之，然后上下粲然有伦，此礼之大经也。名器既亡，则礼安得独在哉！昔仲叔于奚有功于卫，辞邑而请繁缨，孔子以为不如多与之邑。惟名与器，不可以假人，君之所司也；政亡则国家从之。卫君待孔子而为政，孔子欲先正名，以为名不正则民无所措手足。夫繁缨，小物也，而孔子惜之；正名，细务也，而孔子先之：诚以名器既乱则上下无以相保故也。夫事未有不生于微而成于著，圣人之虑远，故能谨其微而治之，众人之识近，故必待其著而后救之；治其微则用力寡而功多，救其著则竭力而不能及也。《易》曰：“履霜坚冰至，”《书》曰：“一日二日万几，”谓此类也。故曰分莫大于名也。
    　　所谓礼教，在于分辨贵贱，排比亲疏，裁决万物，处理日常事物。没有一定的名位，就不能显扬；没有器物，就不能表现。只有用名位来分别称呼，用器物来分别标志，然后上下才能井然有序。这就是礼教的根本所在。如果名位、器物都没有了，那么礼教又怎么能单独存在呢！当年仲叔于奚为卫国建立了大功，他谢绝了赏赐的封地，却请求允许他享用贵族才应有的马饰。孔子认为不如多赏赐他一些封地，惟独名位和器物，绝不能假与他人，这是君王的职权象征；处理政事不坚持原则，国家也就会随着走向危亡。卫国国君期待孔子为他崐处理政事，孔子却先要确立名位，认为名位不正则百姓无所是从。马饰，是一种小器物，而孔子却珍惜它的价值；正名位，是一件小事情，而孔子却要先从它做起，就是因为名位、器物一紊乱，国家上下就无法相安互保。没有一件事情不是从微小之处产生而逐渐发展显著的，圣贤考虑久远，所以能够谨慎对待微小的变故及时予以处理；常人见识短浅，所以必等弊端闹大才来设法挽救。矫正初起的小错，用力小而收效大；挽救已明显的大害，往往是竭尽了全力也
    　　不能成功。《易经》说：“行于霜上而知严寒冰冻将至。”《尚书》说：“先
    　　王每天都要兢兢业业地处理成千上万件事情。”就是指这类防微杜渐的例子。
     
    


很稳，正是想要的。

## 4.存入文件、编写完整程序、检查输出

为了后面存储方便，提取一下当前页的标题。


```python
bs.title.text
```




    '资治通鉴第一卷(周纪)原文_全文_翻译_白话文_书摘天下'




```python
title_pattern = r".*\)"
title = re.findall(title_pattern, bs.title.text)[0]
title
```




    '资治通鉴第一卷(周纪)'



用提取的页面标题`title`命名文件，将正文`result`写入文件，检查是否符合需求。


```python
f = open(title + ".txt", "w")
f.write(result)
f.close()
```

看一下文件内容：


```python
f = open(title + ".txt", "r")
print(f.read())
```

    资治通鉴第一卷(周纪)
    
    周纪一 威烈王二十三年（戊寅、前403）
    　　周纪一 周威烈王二十三年（戊寅，公元前403年）
    　　[1]初命晋大夫魏斯、赵籍、韩虔为诸侯。
    　　[1]周威烈王姬午初次分封晋国大夫魏斯、赵籍、韩虔为诸侯国君。
    　　臣光曰：臣闻天子之职莫大于礼，礼莫大于分，分莫大于名。何谓礼？纪纲是也。何谓分？君、臣是也。何谓名？公、侯、卿、大夫是也。
    　　臣司马光曰：我知道天子的职责中最重要的是维护礼教，礼教中最重要的是区分地位，区分地位中最重要的是匡正名分。什么是礼教？就是法纪。什么是区分地位？就是君臣有别。什么是名分？就是公、侯、卿、大夫等官爵。
    　　夫以四海之广，兆民之众，受制于一人，虽有绝伦之力，高世之智，莫不奔走而服役者，岂非以礼为之纪纲哉！是故天子统三公，三公率诸侯，诸侯制卿大夫，卿大夫治士庶人。贵以临贱，贱以承贵。上之使下犹心腹之运手足，根本之制支叶，下之事上犹手足之卫心腹，支叶之庇本根，然后能上下相保而国家治安。故曰天子之职莫大于礼也。
    　　四海之广，亿民之众，都受制于天子一人。尽管是才能超群、智慧绝伦的人，也不能不在天子足下为他奔走服务，这难道不是以礼作为礼纪朝纲的作用吗！所以，天子统率三公，三公督率诸侯国君，诸侯国君节制卿、大夫官员，卿、大夫官员又统治士人百姓。权贵支配贱民，贱民服从权贵。上层指挥下层就好像人的心腹控制四肢行动，树木的根和干支配枝和叶；下层服侍上层就好像人的四肢卫护心腹，树木的枝和叶遮护根和干，这样才能上下层互相保护，从而使国家得到长治久安。所以说，天子的职责没有比维护礼制更重要的了。
    　　文王序《易》，以乾、坤为首。孔子系之曰：“天尊地卑，乾坤定矣。卑高以陈，贵贱位矣。”言君臣之位犹天地之不可易也。《春秋》抑诸侯，尊王室，王人虽微，序于诸侯之上，以是见圣人于君臣之际未尝不也。非有桀、纣之暴，汤、武之仁，人归之，天命之，君臣之分当守节伏死而已矣。是故以微子而代纣则成汤配天矣，以季札而君吴则太伯血食矣，然二子宁亡国而不为者，诚以礼之大节不可乱也。故曰礼莫大于分也。
    　　周文王演绎排列《易经》，以乾、坤为首位。孔子解释说：“天尊贵，地卑微，阳阴于是确定。由低至高排列有序，贵贱也就各得其位。”这是说君主和臣子之间的上下关系就像天和地一样不能互易。《春秋》一书贬低诸侯，尊崇周王室，尽管周王室的官吏地位不高，在书中排列顺序仍在诸侯国君之上，由此可见孔圣人对于君臣关系的关注。如果不是夏桀、商纣那样的暴虐昏君，对手又遇上商汤、周武王这样的仁德明主，使人民归心、上天赐命的话，君臣之间的名分只能是作臣子的恪守臣节，矢死不渝。所以如果商朝立贤明的微子为国君来取代纣王，成汤创立的商朝就可以永配上天；而吴国如果以仁德的季札做君主，开国之君太伯也可以永享祭祀。然而微子、季札二人宁肯国家灭亡也不愿做君主，实在是因为礼教的大节绝不可因此破坏。所以说，礼教中最重要的就是地位高下的区分。
    　　夫礼，辨贵贱，序亲疏，裁群物，制庶事，非名不著，非器不形；名以命之，器以别之，然后上下粲然有伦，此礼之大经也。名器既亡，则礼安得独在哉！昔仲叔于奚有功于卫，辞邑而请繁缨，孔子以为不如多与之邑。惟名与器，不可以假人，君之所司也；政亡则国家从之。卫君待孔子而为政，孔子欲先正名，以为名不正则民无所措手足。夫繁缨，小物也，而孔子惜之；正名，细务也，而孔子先之：诚以名器既乱则上下无以相保故也。夫事未有不生于微而成于著，圣人之虑远，故能谨其微而治之，众人之识近，故必待其著而后救之；治其微则用力寡而功多，救其著则竭力而不能及也。《易》曰：“履霜坚冰至，”《书》曰：“一日二日万几，”谓此类也。故曰分莫大于名也。
    　　所谓礼教，在于分辨贵贱，排比亲疏，裁决万物，处理日常事物。没有一定的名位，就不能显扬；没有器物，就不能表现。只有用名位来分别称呼，用器物来分别标志，然后上下才能井然有序。这就是礼教的根本所在。如果名位、器物都没有了，那么礼教又怎么能单独存在呢！当年仲叔于奚为卫国建立了大功，他谢绝了赏赐的封地，却请求允许他享用贵族才应有的马饰。孔子认为不如多赏赐他一些封地，惟独名位和器物，绝不能假与他人，这是君王的职权象征；处理政事不坚持原则，国家也就会随着走向危亡。卫国国君期待孔子为他崐处理政事，孔子却先要确立名位，认为名位不正则百姓无所是从。马饰，是一种小器物，而孔子却珍惜它的价值；正名位，是一件小事情，而孔子却要先从它做起，就是因为名位、器物一紊乱，国家上下就无法相安互保。没有一件事情不是从微小之处产生而逐渐发展显著的，圣贤考虑久远，所以能够谨慎对待微小的变故及时予以处理；常人见识短浅，所以必等弊端闹大才来设法挽救。矫正初起的小错，用力小而收效大；挽救已明显的大害，往往是竭尽了全力也
    　　不能成功。《易经》说：“行于霜上而知严寒冰冻将至。”《尚书》说：“先
    　　王每天都要兢兢业业地处理成千上万件事情。”就是指这类防微杜渐的例子。
     
    


除了最后一行是页面本来的格式问题外，其余部分正是所求。剩下的工作就是写好循环，并适当输出一些信息，以完成对全部294卷正文的提取。

下面即为正式提取所用的代码，附图为终端上完成后的输出截图。最后一行输出`page`和`chapter`数目当时写反了，已改正。

```python
from urllib.request import urlopen
from bs4 import BeautifulSoup
import re

content_url = 'http://www.shuzhai.org/gushi/tongjian/'

text_pattern = r"文言翻译(\n)+(\n|\r)*(.*)来源"
title_pattern = r".*\)"

def extract_title_text_from_page(page_url):
    html = urlopen(page_url)
    bs_obj = BeautifulSoup(html.read())
    body = bs_obj.body.text
    text = re.findall(text_pattern, body, re.DOTALL)[0][2]
    title = re.findall(title_pattern, bs_obj.title.text)[0]

    return (title, text)

def extract_page_links(chapter_url):
    html = urlopen(chapter_url)
    bs_obj = BeautifulSoup(html.read())
    page_links = bs_obj.findAll("a", {"href": re.compile("\d+_\d+\.html")})
    pages = set()
    pages.add(chapter_url)
    
    for link in page_links:
        pages.add(content_url + link.attrs["href"])

    return pages

def generate_all_chapter_links(content_url=content_url):
    html = urlopen(content_url)
    bs = BeautifulSoup(html.read())
    links = bs.findAll("a", {"href": re.compile(".*gushi\/tongjian\/.*html"), "title":None})
    target_links = [link.attrs['href'] for link in links]

    return target_links

if __name__ == "__main__":
    chapter_links = generate_all_chapter_links()
    page_count = 0
    chapter_count = len(chapter_links)
    processed_chapter = 0

    print("[INFO] %d chapters to be processed." % chapter_count)

    for chapter_link in chapter_links:
        print('[INFO] Extracting page links from: %s' % chapter_link)
        page_links = extract_page_links(chapter_link)

        page_count += len(page_links)

        for page_link in page_links:
            print('[INFO] Extracting page text from: %s' % page_link)
            title, text = extract_title_text_from_page(page_link)
            f = open("%s.txt" % (title), "w")
            f.write(text)
            f.close()

        processed_chapter += 1
        print('\n[INFO] Processed: %d/%d \n' % (processed_chapter, chapter_count))

    print('[INFO] Processed %d pages from %d chapters.' \
        % (page_count, chapter_count))
    
 ```

![终端输出](https://s2.ax1x.com/2019/09/17/nIyxcd.md.png)

`tree`工具看一下产物，前言所述的提取任务完成，之后有时间继续建立检索相关的工作。


```python
!tree zztj
```

    [34;42mzztj[00m
    ├── zztj.py
    ├── 资冶通鉴第五十二卷(汉纪)(10).txt
    ├── 资冶通鉴第五十二卷(汉纪)(11).txt
    ├── 资冶通鉴第五十二卷(汉纪)(12).txt
    ├── 资冶通鉴第五十二卷(汉纪)(13).txt
    ├── 资冶通鉴第五十二卷(汉纪)(14).txt
    ├── 资冶通鉴第五十二卷(汉纪)(2).txt
    ├── 资冶通鉴第五十二卷(汉纪)(3).txt
    ├── 资冶通鉴第五十二卷(汉纪)(4).txt
    ├── 资冶通鉴第五十二卷(汉纪)(5).txt
    ├── 资冶通鉴第五十二卷(汉纪)(6).txt
    ├── 资冶通鉴第五十二卷(汉纪)(7).txt

     …………（省略若干条目）

    ├── 资治通鉴第四十四卷(汉纪).txt
    ├── 资治通鉴第四卷(周纪)(10).txt
    ├── 资治通鉴第四卷(周纪)(11).txt
    ├── 资治通鉴第四卷(周纪)(12).txt
    ├── 资治通鉴第四卷(周纪)(2).txt
    ├── 资治通鉴第四卷(周纪)(3).txt
    ├── 资治通鉴第四卷(周纪)(4).txt
    ├── 资治通鉴第四卷(周纪)(5).txt
    ├── 资治通鉴第四卷(周纪)(6).txt
    ├── 资治通鉴第四卷(周纪)(7).txt
    ├── 资治通鉴第四卷(周纪)(8).txt
    ├── 资治通鉴第四卷(周纪)(9).txt
    └── 资治通鉴第四卷(周纪).txt
    
    0 directories, 3614 files

