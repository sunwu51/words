# 单词本
# 1 背景
这是我的单词本，毕业之后英语学习就放下了，工作和生活中经常遇到一些明明拼写很简单但是就是想不起来是什么意思的单词。以及每天受我美丽的老婆熏陶，所以想继续学英语，每周都记录一下单词放到单词本中。

本来想就是搞个记事本，像OneNote这样的记录一下，但是觉得只把单词记下来，容易之后忘了他是啥意思，那就把意思也记下来吧。再一想，那得把例句和短语写上啊，不然不会用。然后发现这不就是词典的功能了，那算了，我做一个自己的词典库吧，然后每周学习的单词，单另出来组成一个页面，历史上学过的单词也是一个页面。
# 2 主要功能
- 页面1： 可以输入生词，输入之后会自动统计到本周的生词中。
- 页面2： 词典，查询单词的时候可以给出解释、词组和例句等。可以和页面1整合在一个页面中，生词、查词然后录入，用户逻辑通。
- 页面3： 本周生词列表，每到周末可以拿出来复习一下这周的单词。
- 页面4： 历史生词列表，这个可以和本周生词做到一个页面，分页展示每周的生词就好了，或者默认展示本周生词，下拉可以拉取更多数据。

# 3 准备工作
## 3.1 单词数据
在github中找到了一个单词库的[repo](https://github.com/mahavivo/english-wordlists)，里面有各种468级词汇GRE词汇等等，我看了下我也不是要考试，平时遇到的也比较常用。所以直接把美国当代语料库(COCA)2w个单词作为我的全部单词就行了，超出这个范围的单词很难见到。

## 3.2 词典数据
上述repo只有单词本身，没有解释，短语和例句等。我需要到网上词典中去爬取数据，可以使用有道的api，上来就给赠送了100块钱，算下来能查10w单词左右。

但是这个api有个小问题，就是单词只有解释和音标，没有词组和例句，最终觉得还是爬虫数据比较好，直接爬有道的数据，这里我对2w单词爬取了`美式音标`、`解释`、`词组`、`例句`这四项内容，每个单词是一个json文件，其中部分单词词典查不到，最终有1w7的数据量。同时对2w个单词爬取了`人声朗读的mp3`数据，也是1w7的结果。这些作为静态页面资源放到前端项目中，`/words-netlify`。

# 4 存储与服务
这个应用我想长期托管，所以不想使用任何付费的资源，Faas平台将会是首选。至于存储，数据库就更贵了，所以想白嫖github仓库作为数据存储了。上面的词库是存到了asset目录，但是单词本主要目的不是来记录2w个单词，而是每周我自己遇到的生词，所以需要有个地方来记录我每周的生词，大概就是记录每个单词啥时候录入的生词本就可以，单词的详细数据在词典数据中都能找到。

用子项目`words-db/words.json`文件来记录每周我学习的单词，一年才50周，所以1年也才50个json对象，其实还是很小的，到时候直接加载到服务里面格式大概是这样，id就是这一周的第一天咯，然后list就是这一周学了哪些单词的列表，然后就用json-server来提供这个单词本服务，这样数据存到json中，然后数据更新了可以push到github就好了，github就是我的持久化存储库。
```json
{
  "words":
  [
    {
      "id": "2022-04-24",
      "list": ["blame"]
    }
  ] 
}
```

## 4.1 heroku
因为普通的Faas服务都不提供上述这么灵活的文件操作和git指令操作，只有heroku是万能的神，但是缺点就是机器在美国，直接访问会比较慢，所以可以把一些api放到这，静态页面放到其他有离中国较近的机房的服务中。

![image](https://i.imgur.com/x8MzihC.png)

## 4.2 netlify
静态页面包括了`html/js/css`、`json文件`、`mp3`文件。

静态的页面还是放到netlify上面吧，一方面觉得netlify比较成熟，使用范围较广，所以不太会倒闭了。另一方面netlify使用非常简单，速度很快，最近的机房在新加坡。

![image](https://i.imgur.com/xYToOfO.png)

# 5 技术栈
涉及到的技术栈：
- 爬数据，使用`java`做了爬虫，用到了一个很好用的类似`jquery`的工具`jsoup`。
- 仓库，使用`github`做代码和数据存放的地方，多分支减少不同地方的数据拉取，按需拉取。
- 服务端，使用`nodejs`来写服务端的逻辑，包括`netlify`的简单转发逻辑和`heroku`的数据操作逻辑等。
- 前端，使用`tailwind`做样式，js框架还没太想好尝试了下`lit`，怕hold不住，到时候不行再换`remix`。
