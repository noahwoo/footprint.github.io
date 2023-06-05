# 文案生成
## 产品流程（一期，一次生成多次修改）
### Use-case：运营
1. 运营选择菜品
2. 根据选择菜品属性指定营销卖点
3. 选择文案生成
4. 阅读并修改文案
5. 上线开播

## 数据抓取与标注
1.直播间文案脚本转录解析，并标注

- 逐句标注标签(Label)：`开场白`,`衔接文案`,`下播文案`,`菜品特性(CFS)`,`互动召唤(CTA)`及`其他`，`菜品特性`包括`价格`、`开放时段`、`优惠券`、`$菜品名称$`、`$产地$`等，`互动召唤`包括`求赞`、`求关注`、`求评论`等。标注结果为`{标签(Label)，直播脚本文案}`的监督对；
- 每个直播场次一个文件，按照时间顺序得到一个tsv的多列标注文档
   - 示例：`username<tab>content<tab>label<tab>start<tab>end`

2.抓取大众点评针对菜品评论的文本，得到`{菜品名称, 评论文案}`的监督对，细节格式待定

## Finetune大语言模型（OpenAI Finetune API - gpt-3.5-turbo）

### Round1: 单菜品特性或行动召唤直播话术学习

1. 用大众点评自助餐相关菜品数据预训练 `gpt-3.5-turbo` 
2. 在#1基础上，用标注直播间文案脚本Finetune 对应的模型
3. 实现细节
   - 调研满足时间和样本规模约束的预训练环境（如用开源方案）
   - 调研短文本的高效Finetune实现方案

### Round2: 文案组织逻辑学习
- 一期：Prompt实现固定模版
  - Part1: {开场白}
  - Part2: {核心卖点文案}
  - Part3: {其他卖点文案}
  - Part3: {互动召唤文案}
  - Part4: {衔接文案}
  - 重复Part1，或者{下播文案}

- 二期：文案模版学习

### 指令提示文案
根据运营选择，整体文案生成通过类似如下Prompt模版驱动（示例，待优化，产品上程序自动生成模版）：
> `以下是{{开场白}}的示例：宝宝们刷到可别错过哈，优惠就在这里\n\n走过路过不要错过，发券到手软\n\n以下是{{下播文案}}的示例\n\n宝宝们下播咯，记得关注哟\n\n 今天福利发到这里了，更多优惠请点关注哟。\n\n 你是一个直播营销文案创作者，请按照如下描述写一段自助餐直播营销的文案：首先生成一段{{开场白}}，[接着介绍{{自助餐厅}}的核心优点: [{{CFS}}]+，然后完成产品其他特性的介绍，包括[{{CFS}}]+，生成{{衔接文案}}]+，最后请生成{{下播文案}}。`

单品示例：

**Prompt**: `以下是{{开场白}}的示例：宝宝们刷到可别错过哈，优惠就在这里\n\n 走过路过不要错过，发券到手软\n\n以下是{{下播结束语}}的示例：\n\n宝宝们下播咯，记得关注哟\n\n今天福利发到这里了，更多优惠请点关注\n\n 你是一个直播营销文案创作者，请按照上面信息和如下描述写一篇自助餐直播营销的文案。首先写一段{{开场白}}，接着介绍自助餐的核心优点: "价格低"，“食材新鲜”，然后完成产品其他特性的介绍，包括“交通方便”，“营业时间长”，最后请写一段{{下播结束语}}。`

**Completion**: 

开场白：宝宝们刷到可别错过哈，优惠就在这里！欢迎来到我们的直播间！今天我要向大家推荐一家绝对让你欲罢不能的自助餐厅！无论你是吃货还是追求性价比的小伙伴，这里都是你的天堂！

自助餐厅直播营销文案：

优点一：价格低！在我们的自助餐厅，你将享受到超实惠的价格。无论你是品尝海鲜还是追求传统美食，我们提供丰富的选择，让你尽情享受美食的同时，也能轻松掌控预算，满足你的胃口和钱包！

优点二：食材新鲜！我们的自助餐厅注重食材的新鲜度，每一样菜肴都精心挑选，确保每一口都能让你感受到最纯粹的美味。新鲜的海鲜、新鲜的蔬菜，每一样都是源自大自然的馈赠，为你带来口感与营养的双重享受！

特性一：交通方便！我们的自助餐厅地理位置优越，交通便利。无论你是乘坐公共交通还是自驾前来，都能轻松抵达。不用担心迷路，跟随导航指引，就能找到这个美食的天堂！

特性二：营业时间长！我们的自助餐厅为你提供长时间的用餐时段，早晨的早餐、中午的午餐、晚上的晚餐，甚至还有宵夜时段等着你来品尝。无论你是忙碌的上班族还是夜猫子，我们都会满足你不同时间段的美食需求！

下播结束语：宝宝们下播咯，记得关注哟！感谢大家今天的收看与参与，希望我们的直播能为你带来无限食欲！别忘了关注我们的直播间，我们将定期推出更多优惠和惊喜！期待与你再次相聚，共同探索美食世界的无限可能！

# 附录：FABE模版 ChatGPT测试

**Prompt** :

`假设你在做营销推广，请用FABE(Feature, Advantage, Benefit, Experience)话术模版介绍一下推荐的小度学习机？\n Feature：支持制定学习计划，并自动生成学习报告，内置中小学各个科目的学习材料；\n\n Advantage：轻便易携，低辐射护眼； \n\n Benefit：价格便宜，包邮，3年售后服务； \n\n Experience：购买的用户都给朋友做了推荐；`

**Completion**:

Feature：小度学习机具有支持制定学习计划的功能，同时能够自动生成学习报告。它内置了中小学各个科目的学习材料，为学生提供全面的学习资源。

Advantage：小度学习机的轻便易携特性使得学生可以随时随地进行学习。它采用低辐射设计，有效护眼，减少眼部疲劳，让学习更加舒适和安全。

Benefit：购买小度学习机的价格相对较低，并且享受包邮的便利。此外，它提供长达3年的售后服务，为用户提供全方位的支持和保障，让用户无后顾之忧。

Experience：许多购买了小度学习机的用户都对它赞不绝口，并且纷纷将这款学习工具推荐给自己的朋友。他们通过小度学习机获得了高效、便捷的学习体验，进一步提升了学习成果。

使用小度学习机，你将体验到个性化的学习计划和全方位的学习支持。它的轻便设计和低辐射护眼特性将使学习过程更加舒适和安全。而且，它的价格实惠，包邮服务和3年的售后保障，让你无后顾之忧。加入众多推荐它的用户行列，享受高效学习的乐趣吧！


请参示例风格写一段直播催单的文案，示例是如下分割的markdown格式

```
- 下播以后就没有啊, 再播以后就没有, 关注主播相当于关注一个流动的优惠券, 一定要先下单先囤, 一定要先下单先囤。
- 先囤先囤, 没有用的话随时退、过时退, 就这一单就一单了, 对这个价格我跟你说马上没瞬间没啊。
- 走过路过的家人们，别错过啊别错过，囤就完事了，咱就直接拍起来完事了啊。
- 走过路过咱们别错过，错过找不到啊。拍起来，先拍先得，后拍后得，拍完了咱们真的找不到了啊。 过了这个村，没有这个店了啊。
```

文案中不能有emoji，你写的催单文案如下：


```
请参示例话术和风格写一段直播催单的文案，如下JSON格式的示例:

{'copywrite-type' : ‘催单文案’, 'copywrites' : ['下播以后就没有啊, 再播以后就没有, 关注主播相当于关注一个流动的优惠券, 一定要先下单先囤, 一定要先下单先囤。', '先囤先囤, 没有用的话随时退、过时退, 就这一单就一单了, 对这个价格我跟你说马上没瞬间没啊。', '走过路过的家人们，别错过啊别错过，囤就完事了，咱就直接拍起来完事了啊。', '走过路过咱们别错过，错过找不到啊。拍起来，先拍先得，后拍后得，拍完了咱们真的找不到了啊。 过了这个村，没有这个店了啊。']}

文案中不能有emoji，你写的20字以上的催单文案如下：
```

ChatGPT response: 
{
"copywrite-type": "催单文案",
"copywrites": [
"主播马上下播，别错过优惠，快下单囤货！",
"先下单，后悔莫及！关注主播，就是关注超值优惠！",
"别犹豫，现在下单立享优惠，错过就没有了！",
"走过路过的朋友们，抓紧时间下单吧，机会稍纵即逝！",
"这个价格只有一次机会，马上下单，别等到下次再后悔！"
]
}


请参示例话术和风格写一段直播催单的文案，如下JSON格式的示例:

{'#催单\n': ['让我抢到吧我中午就去喝', '"直播间宝们抓紧手速抓紧网速, 秒排秒付了啊"', '"所以我是希望大家现在还没有去拍的, 现在还想要去拍一波的抓紧手速去拍, 抓紧手速给我去捅一波就可以了"', '"今天没有去下单宝呢, 那抓紧下单哦"', '如果还没有去拍的还想要去拍一波的'], '#催单#库存提醒\n': ['"如果说你还没有拍下的, 想拍的一定要赶紧去拍了啊"', '先把您喜欢的东西都给他囤下来', '先把您喜欢的东西都给他囤下来', '所以趁着现在赶紧去囤货', '"你可以直接去拍一下, 直接去抢一下都是没有问题的"'], '#催单#商品#菜品\n': ['没抢到薯条[流泪][流泪][流泪][流泪][流泪][流泪][流泪][流泪][流泪][流泪][流泪][流泪][流泪][流泪][流泪][流泪][流泪][流泪]', '"如果说今天你有需求一定要赶紧去拍啊, 比如说你是想要去哎, 想要去说带着一家三口晚上去吃个饭啊"', '"我也希望大家如果还没有去拍的, 想要去吃咱们后期五花肉的, 来抓紧时间了, 抓紧时间"', '"您还等啥呢, 咱们吃完这一顿小龙虾呀, 烦恼呢, 真的是会一扫而光的啊"', '开心，今天抢了好多薯条'], '#催单#限时活动\n': ['"超级超级给力, 好棒棒的一个活动专场, 一定要趁现在抓紧时间拼手, 走平行走"', '"宝贝们准备好手速再抢一次, 2号连接拼手速再抢一次倒计时5秒钟"', '刷新去拍抓你手速抓你时间到', '今天最不吃醋宝宝一定要赶去', '刷新去拍抓你手速抓你时间到'], '#互动#催单\n': ['你没有抢到你回来打继续', '"来到我这啊抓紧手步啊我想不出名字, 然后的话呢还在我们直播间啊你们可以赶紧赶紧去冲啊"', '"但是你说你们都慌慌下单, 你知不知道"', '"但是你说你们都慌慌下单, 你知不知道"', '"但是你说你们都慌慌下单, 你知不知道"'], '#库存提醒#催单\n': ['"马上没马上没啊, 马上没瞬间没啊马上没瞬间没, 拼手速拼手速啊拼手速"', '"然后直播间有没有还在犹豫没付款的, 没付款抓紧付款 等一下我们都库存都会踢掉的"', '还没有拍没有抢到宝贝抓紧时间拼手速拼网速了手快有手慢就真的没有了先到先得啊', '"怎么没有了, 有了赶紧抢, 没有了, 太快了, 我跟你说"', '"我都给大家延长到这个一个小时了啊, 所以说呢这个赶紧拍赶紧囤"']}


文案中不能有emoji，你写3个菜品、库存、互动类型的20字以上的催单文案如下：