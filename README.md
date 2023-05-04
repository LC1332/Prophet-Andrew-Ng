
# 吴恩达老师的Prompt Engineering课程学习

本项目由李鲁鲁老师在DataWahle翻译的中文版本的吴恩达老师与 OpenAI 合作推出的 《ChatGPT Prompt Engineering for Developers》基础上，进行了一些自己的实践思考。

原来项目的链接是 [https://github.com/datawhalechina/prompt-engineering-for-developers](https://github.com/datawhalechina/prompt-engineering-for-developers)

每一课都在colab上进行了改造，并将链接放在了readme，可以直接用colab进行学习。

## 快速开始

|  | Colab链接 | 细节 |
| --- | --- | :--- |
| 2. 提示原则 | <a href="https://colab.research.google.com/github/LC1332/prompt-ng-andrew/blob/main/content/2.%20%E6%8F%90%E7%A4%BA%E5%8E%9F%E5%88%99%20Guidelines.ipynb" target="_parent"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/></a> | 基础的课程 |
| 3. 迭代优化 | <a href="https://colab.research.google.com/github/LC1332/prompt-ng-andrew/blob/main/content/3.%20%E8%BF%AD%E4%BB%A3%E4%BC%98%E5%8C%96%20Iterative.ipynb" target="_parent"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/></a> | 针对一个椅子商品描述的迭代优化 |
| 4. 文本概括 | <a href="https://colab.research.google.com/github/LC1332/prompt-ng-andrew/blob/main/content/4.%20%E6%96%87%E6%9C%AC%E6%A6%82%E6%8B%AC%20Summarizing.ipynb" target="_parent"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/></a> | 对文本进行针对性的概括 |
| 3. 迭代优化 | <a href="https://colab.research.google.com/github/LC1332/prompt-ng-andrew/blob/main/content/3.%20%E8%BF%AD%E4%BB%A3%E4%BC%98%E5%8C%96%20Iterative.ipynb" target="_parent"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/></a> | 针对一个椅子商品描述的迭代优化 |
| 2. 提示原则 | <a href="https://colab.research.google.com/github/LC1332/prompt-ng-andrew/blob/main/content/2.%20%E6%8F%90%E7%A4%BA%E5%8E%9F%E5%88%99%20Guidelines.ipynb" target="_parent"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/></a> | 基础的课程 |
| 3. 迭代优化 | <a href="https://colab.research.google.com/github/LC1332/prompt-ng-andrew/blob/main/content/3.%20%E8%BF%AD%E4%BB%A3%E4%BC%98%E5%8C%96%20Iterative.ipynb" target="_parent"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/></a> | 针对一个椅子商品描述的迭代优化 |
| 2. 提示原则 | <a href="https://colab.research.google.com/github/LC1332/prompt-ng-andrew/blob/main/content/2.%20%E6%8F%90%E7%A4%BA%E5%8E%9F%E5%88%99%20Guidelines.ipynb" target="_parent"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/></a> | 基础的课程 |
| 3. 迭代优化 | <a href="https://colab.research.google.com/github/LC1332/prompt-ng-andrew/blob/main/content/3.%20%E8%BF%AD%E4%BB%A3%E4%BC%98%E5%8C%96%20Iterative.ipynb" target="_parent"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/></a> | 针对一个椅子商品描述的迭代优化 |


## 有趣的例子

**总结修仙斗法过程**

```python
# 有步骤的文本
text_1 = f"""
韩立叹了一口气，原本还想费些口舌，看看能否糊弄过去，但如今看对方的神态和凭对方的心计，是一点迂回的余地都没有了，他和自己肯定只能有一个还可活在世上，还是不要白费口舌，先下手为强算了。\
想到这里，韩立二话不说，一扬左手一个精钢环就发出怪啸之声，直向“陆师兄”冲去，随后又把右手一亮，青黑色的葫芦出现在了手中，并从葫芦嘴中喷出了五六个黑糊糊的圆球，紧跟在钢环后而去。\
做完这一切韩立并未罢手，他空出的左手在虚空中略一比划，刹那间浮现了数个红色火球，袖子略微一动，把这些火球卷在其中，紧接着再冲“陆师兄”猛然一甩，口中低吐一个“去”字。\
顿时，夹带着一股炎热之气，火球们一窝蜂的四行散开，不同角度砸向了陆师兄。\
这一番出手，韩立几乎动用了未得到新法器之前，不使用符宝的一切进攻手段，特别是最后这手数弹并发的瞬发手法，更是韩立费了好大工夫才从吴风那里学到手的，为的就是想要打对方一个措手不及，一举闪电击毙对手。
"""
prompt = f"""
您将获得由三个引号括起来的文本。\
如果它包含一系列的打斗的片段，则需要按照以下回合制描述的格式重新编写这些片段：

第一回合 - <A> 使用 <A's skill>, <skill result description>
第二回合 - <B> 使用 <B's skill>, <skill result description>
…
第N回合 - …

如果文本中不包含一系列的打斗，则直接写“未提供打斗细节”。"
\"\"\"{text_1}\"\"\"
"""
response = get_completion(prompt)
print("Text 1 的总结:")
print(response)
```

结果

```
Text 1 的总结:
第一回合 - 韩立 使用 精钢环, 击中“陆师兄”
第二回合 - 韩立 使用 葫芦中的黑糊糊圆球, 击中“陆师兄”
第三回合 - 韩立 使用 火球术, 击中“陆师兄”

未提供打斗细节。
```

**先知谈“时间”**

```
你的任务是以一致的风格回答问题。

<市民>: 请和我们讨论一下"孩子"

<先知>: """你的孩子，其实不是你的孩子，
他们是生命对于自身渴望而诞生的孩子，
他们通过你来到这世界，却非因你而来，
他们在你身边，却并不属于你。
你可以给予他们的是你的爱，却不是你的想法，
因为他们自己有自己的思想。
你可以庇护的是他们的身体，却不是他们的灵魂，
因为他们的灵魂属于明天，属于你做梦也无法达到的明天。
你可以拼尽全力，变得象他们一样，
却不要让他们变得和你一样，
因为生命不会后退，也不在过去停留。
你是弓，儿女是从你那里射出的箭。
弓箭手望着未来之路上的箭靶，
他用尽力气将你拉开，使他的箭射得又快又远。
怀着快乐的心情，在弓箭手的手里弯曲吧，
因为他爱一路飞翔的箭，也爱无比稳定的弓。"""

<市民>:请和我们讨论一下"衣服"

<先知>: 你们的衣服遮掩了你们许多的美，却不能遮盖住丑。
尽管你们借衣服寻求隐私的自由，但你们找到的却是羁绊和束缚。
但愿你们用自己的肌肤而不是衣服去迎接阳光和清风，
因为阳光中有生命的气息，而风中有生命之手。
你们中一些人说：“是北风织造了我们所穿的衣服。”
我说，是的，的确是北风，
但它以羞怯为织机，以纤弱的肌肉为纱线。
它一旦完成工作，便会在林中大笑。
不要忘记，羞怯原是抵挡不洁目光的盾牌。
若无邪侪，那羞怯除了是精神上的束缚和污垢外还能是什么？
也不要忘记，大地乐于感觉体赤裸的双脚，风儿渴望与你的头发嬉戏。

<市民>:请和我们讨论一下"时间"
```

输出


```
<先知>: 时间是一条河流，它汇聚着每个生命的流动和消逝。
每个人都在时间的河流中漂泊，不断地向前推进。
我们无法停止或掌控时间，但可以选择如何利用它。
时间是宝贵的，它可以让我们成长、学习、改变和创造。
但若我们不珍惜时间，它也会成为我们生命中的遗憾和遗失。
无论过去、现在还是未来，时间都是无情的，它会不断流逝，
我们唯一能做的就是珍惜它，珍惜每分每秒的生命，让我们的时间变得更加有意义和宝贵。
```



![figures/readme.png](https://github.com/datawhalechina/prompt-engineering-for-developers/blob/main/figures/readme.jpeg)

# prompt-engineering-for-developers

### 项目简介

吴恩达《ChatGPT Prompt Engineering for Developers》课程中文版，主要内容为指导开发者如何构建 Prompt 并基于 OpenAI API 构建新的、基于 LLM 的应用，包括：

> 书写 Prompt 的原则;
> 文本总结（如总结用户评论）；
> 文本推断（如情感分类、主题提取）；
> 文本转换（如翻译、自动纠错）；
> 扩展（如书写邮件）;

**英文原版地址：[ChatGPT Prompt Engineering for Developers](https://learn.deeplearning.ai)**

### 项目意义

LLM 正在逐步改变人们的生活，而对于开发者，如何基于 LLM 提供的 API 快速、便捷地开发一些具备更强能力、集成LLM 的应用，来便捷地实现一些更新颖、更实用的能力，是一个急需学习的重要能力。由吴恩达老师与 OpenAI 合作推出的 《ChatGPT Prompt Engineering for Developers》教程面向入门 LLM 的开发者，深入浅出地介绍了对于开发者，如何构造 Prompt 并基于 OpenAI 提供的 API 实现包括总结、推断、转换等多种常用功能，是入门 LLM 开发的经典教程。因此，我们将该课程翻译为中文，并复现其范例代码，支持国内中文学习者直接使用，以帮助中文学习者更好地学习 LLM 开发。

### 项目受众

适用于所有具备基础 Python 能力，想要入门 LLM 的开发者。

### 项目亮点

《ChatGPT Prompt Engineering for Developers》作为由吴恩达老师与 OpenAI 联合推出的官方教程，在可预见的未来会成为 LLM 的重要入门教程，但是目前还只支持英文版且国内访问受限，打造中文版且国内流畅访问的教程具有重要意义。

### 内容大纲

**目录：**
1. 简介 Introduction @邹雨衡
2. Prompt 的构建原则 Guidelines @邹雨衡
3. 如何迭代优化 Prompt Itrative @邹雨衡
4. 文本总结 Summarizing @玉琳
5. 文本推断 Inferring @长琴
6. 文本转换 Transforming @玉琳
7. 文本扩展 Expanding @邹雨衡
8. 聊天机器人 Chatbot @长琴
9. 总结 @长琴

### 致谢

**核心贡献者**

- [邹雨衡-项目负责人](https://github.com/nowadays0421)
- [长琴](https://yam.gift/)（内容创作者-Datawhale成员-AI算法工程师）
- [玉琳](https://github.com/Sophia-Huang)（内容创作者-Datawhale成员）


**其他**

1. 特别感谢 [@Sm1les](https://github.com/Sm1les)、[@LSGOMYP](https://github.com/LSGOMYP) 对本项目的帮助与支持；

### 关注我们

<div align=center>
<p>扫描下方二维码关注公众号：Datawhale</p>
<img src="figures/qrcode.jpeg" width = "180" height = "180">
</div>
&emsp;&emsp;Datawhale，一个专注于AI领域的学习圈子。初衷是for the learner，和学习者一起成长。目前加入学习社群的人数已经数千人，组织了机器学习，深度学习，数据分析，数据挖掘，爬虫，编程，统计学，Mysql，数据竞赛等多个领域的内容学习，微信搜索公众号Datawhale可以加入我们。

## LICENSE
<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://img.shields.io/badge/license-CC%20BY--NC--SA%204.0-lightgrey" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。
