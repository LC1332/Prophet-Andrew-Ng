# 【骆驼LangChain笔记7】PALChain解答中学数学联赛一试， LangChain中的长文本总结和复查, VisualGPT的langchain使用

因为最近额外的时间比较多都在[Chat凉宫春日](https://github.com/LC1332/Chat-Haruhi-Suzumiya) 这个项目上

虽然LangChain的视频我已经看到比较后面了，但是笔记还没来得及写。我决定压缩一下。把7-10都放在一篇里面去写。今天的笔记会包括下面几个

+ 7 PalChain
+ 8 LangChain的几种Summarization
+ 9 LangChain的SummarizationChecker机制
+ 10 微软VisualGPT中LangChain的使用

其实我拿天龙八部试了一下，LangChain的总结摘要设计让我有点不太满意。具体我们在这个笔记里面讨论吧。

笔记的所有内容都可以在 [骆驼先知](https://github.com/LC1332/Prophet-Andrew-Ng) 这个项目中看到

也可以直接访问骆驼项目的主页去跳转

[骆驼：开源中文大语言模型](https://github.com/LC1332/Luotuo-Chinese-LLM)

---

## PALChain

Sam还是比较喜欢这个PALChain的，所以专门开了一个视频来讲这个PALChain

其实PAL就做了一个事情，利用few shot的方法，可以把一个数学问题变为编程问题，然后再运行写出来的python代码

PALChain的使用非常简单，差不多这么用就可以

```python
from langchain.chains import PALChain

pal_chain = PALChain.from_math_prompt(llm, verbose=True)

question = "The cafeteria had 23 apples. \
If they used 20 for lunch and bought 6 more,\
how many apples do they have?"

pal_chain.run(question)
```

然后你就会发现输出是这样的

```python
> Entering new PALChain chain...
def solution():
    """The cafeteria had 23 apples. If they used 20 for lunch and bought 6 more,how many apples do they have?"""
    apples_initial = 23
    apples_used = 20
    apples_bought = 6
    apples_left = apples_initial - apples_used + apples_bought
    result = apples_left
    return result

> Finished chain.
9
```

所以我就在想中文行不行，结果是可以的

## 中文的测试

先找了个非常简单的进行测试

```python
question = "地上一个猴，树上七个猴，一共有几个猴？"

pal_chain.run(question)
```

他的输出是这样的

```python
> Entering new PALChain chain...
def solution():
    """地上一个猴，树上七个猴，一共有几个猴？"""
    total_monkeys = 1 + 7
    result = total_monkeys
    return result

> Finished chain.
8
```

因为OpenAI合并了code-davinci模型和turbo模型，所以现在turbo模型的代码实力实际上是很强的。

所以我们赶紧拿数学联赛的题试一下


## 2004高中数学联赛一试

```python
question = "设三位数n=abc, 若以a,b,c为三条边的长可构成一个等腰（含等边）三角形，则这样的三位数n有多少个？"

pal_chain.run(question)
```

这个题其实对人类来说不是特别简单。

```python
> Entering new PALChain chain...
def solution():
    """设三位数n=abc, 若以a,b,c为三条边的长可构成一个等腰（含等边）三角形，则这样的三位数n有多少个？"""
    count = 0
    for n in range(100, 1000):
        a = int(str(n)[0])
        b = int(str(n)[1])
        c = int(str(n)[2])
        if a == b or b == c or a == c:
            if a + b > c and b + c > a and a + c > b:
                count += 1
    result = count
    return result

> Finished chain.
165
```

被这玩意儿秒杀了。。

再放一道试试

```python
question = "某个货场有 1997 辆车排队等待装货，要求第一辆车必须装 9 箱货物，每相邻的 4辆车装货总数为 34 箱，为满足上述要求，至少应该有货物的箱数是多少？"

pal_chain.run(question)
```

他给解答也很快哈

```python
> Entering new PALChain chain...
def solution():
    """某个货场有 1997 辆车排队等待装货，要求第一辆车必须装 9 箱货物，每相邻的 4辆车装货总数为 34 箱，为满足上述要求，至少应该有货物的箱数是多少？"""
    num_cars = 1997
    first_car_cargo = 9
    cargo_per_group = 34
    cars_per_group = 4
    num_groups = (num_cars - 1) // cars_per_group  # total number of groups (excluding the first car)
    total_cargo = first_car_cargo + num_groups * cargo_per_group
    result = total_cargo
    return result

> Finished chain.
16975
```

因为我小时候是理科班的，我们数理化都是竞赛内容。对于我不参加数学竞赛来说，数学竞赛的考卷上的题大概就一半左右能做。

然后有一天万军老师发的卷子上有一道涉及讨论所有四位数的组合数学题。

我把卷子上其他看起来能做的题做了以后，就用TI计算机顺手编程把这题给解了。

然后分析考卷的时候，（似乎没几个人做完这题），万老师直接说，“李诚你这题拿TI计算机做的吧”

---

不过我们要说一下数学学科的学科哲学不是这样的。更重要的是公理体系出发去证明一些命题这样接近哲学的思维。

计算机在数学中往往只能用来做preliminary的验证。

如果你觉得计算机能解这些组合题 就等于 计算机能做数学竞赛，甚至学数学是没必要的了 那还是有点反智主义了。

有的同学建议我拿一试的几何题也试试，我就懒得实验了，我猜非证明的题有些题还是做的出来的，有空的同学可以试一试。

---

这个工作也吸引了我的同学韩京俊的兴趣，他问了论文的地址 论文地址在这儿

https://reasonwithpal.com/


## LangChain做长文本总结

这里我其实几天前就把视频看了，但是代码一直拖到昨天才实践。

他这里本来Sam用的文本是一个英文的，我给改成天龙八部中虚竹大战鸠摩智的片段了。

### 文本切块

因为GPT的输入是4096限制的，就算你使用claude也就是两倍，GPT4是32k，你总有用完的一天。

对于一个超长文本来说，总是要把文本切块儿的。这节课给了一个切块函数的例子

```python
from langchain.text_splitter import CharacterTextSplitter

text_splitter = CharacterTextSplitter(
    chunk_size = 1600,
    chunk_overlap  = 50)

texts = text_splitter.split_text(demi_gods)
```

这里demi_gods是我提前读取的长段的天龙八部文本。他这里的意思是会把文本切成

长度尽可能接近chunk_size，相互之间尽可能接近chunk_overlap的几个chunk

这里我试图打印了一下

```python
for text in texts:
  print(len(text))
  print(text[:10])
  print(text[-10:])
```

输出是这样的

```
1590
突然外面一个清朗的声
“大金刚拳”的秘要。
1540
鸠摩智不等铜鼎落下，
又焉有第二条路好走？
1483
殿上诸般事故，虚竹一
于他吐蕃国大有好处。
1595
只听他朗声说道：“小
和尚想必由此而知。”
1413
他天生睿智，自少年时
是般若掌的上乘功夫。
1519
韦陀掌是少林派的扎根
的面门相距已逾一尺。
1585
鸠摩智早已发觉，虚竹
困境，无不暗暗欢喜。
1151
神山上人自从鸠摩智一
袭，简直不成体统。”
```

可以看到在结尾的时候，CharacterTextSplitter会尽可能截取标点符号作为结尾。开头也应该尽可能是这样的。我能想到应该可能是一个贪心或者动态规划的算法在中间起作用。

### Map \& Reduce的总结策略

最简单的方法，就是对每个chunk进行总结之后，再对总的总结进行总结，于是代码是这样的。在一开始写笔记的时候我不知道prompt怎么额外设置，我就用了暴力设置的方法

```python
from langchain.chains.summarize import load_summarize_chain

chain = load_summarize_chain(llm, 
                             chain_type="map_reduce")

chain.llm_chain.prompt.template = """对下列文本编写一个简明的总结:\n\n\n"{text}"\n\n\n总结:"""

chain.combine_document_chain.llm_chain.prompt.template = """对下列文本编写一个简明的总结:\n\n\n"{text}"\n\n\n总结:"""

output_summary = chain.run(docs)
```

他给我输出是这样的

```
鸠摩智以超凡口才和精湛武功惊艳了少林寺的众僧。他展示了自己的绝世武功，让高僧们叹服。虚竹在少林寺遇到了种种怪事，展示了高超的轮指封穴技巧。鸠摩智与虚竹展开激烈的武术对决，虚竹使用逍遥派的武学技巧，而鸠 摩智使用火焰刀等危险技巧。最终虚竹以反击和抓住鸠摩智的手腕为契机，战胜了鸠摩智。
```

总感觉差那么一回事儿。。。

### Stiffing的方法

这个方法对于长文本是不适用的

我们直接跳入下一个方法使用bullets形式

### 使用自定义prompt将总结转化为bullet形式

这个好像是Sam自己搞出来的，也有可能是他借鉴别人的，总之这个prompt并不是langchain默认的，但是可以借助langchain来实现

不过还挺好用的，我们来看一下。

他这里新定义了一个prompt

```python
prompt_template = """对以下内容进行简明的要点总结，用bullet point形式:


{text}


使用BULLET POINTS形式总结:"""

BULLET_POINT_PROMPT = PromptTemplate(template=prompt_template, 
                        input_variables=["text"])
```

然后我们来用这个prompt模版来进行总结


```python
llm = ChatOpenAI(model_name='gpt-3.5-turbo')

## with intermediate steps
chain = load_summarize_chain(llm, 
                             chain_type="map_reduce", 
                             return_intermediate_steps=True, 
                             map_prompt=PROMPT, 
                             combine_prompt=PROMPT)

output_summary = chain({"input_documents": docs}, return_only_outputs=True)
```

这里可以看到有个return_intermediate_steps的开关被打开了，这个时候会返回一个字典，里面有intermediate_steps和output_text两个字段，让我们先看看前者。

```python
for inter_result in output_summary['intermediate_steps']:
    print(inter_result)
```

这个中间的输出还是很令人满意的，为了防止大家已经忘记了天龙八部这个桥段，我们都给打印出来吧。

```
• 一个清朗的声音传来，是吐蕃国山僧鸠摩智，参见少林寺方丈。
• 群僧对他的身手感到惊异，他坐在神山的上首。
• 鸠摩智指出少林派七十二门绝技中的问题，包括每年能成一项绝技的说法。
• 群僧不理解他的意思，玄生请他解释。
• 鸠摩智展示了大金刚拳法中的一招“洛钟东应”，展示出了深得“大金刚拳”的秘要。
• 群僧对他的能力感到惊叹，但有些人仍持怀疑态度。
- 鸠摩智使用般若掌中的“慑伏外道”掌法，将铜鼎上的一块手掌般的物件割落
- 众僧惊叹他的“三入地狱”摩诃指功，认为他的功力已超凡入圣的境地
- 落下的物件是一只黄铜手掌，袍袖一拂后，他展示了“袈裟伏魔功”，玄慈方丈对其赞叹不已
- 玄慈方丈认为鸠摩智武功高极，少林派技不如人，将波罗星请离寺院
- 众僧心生失落，感受到少林派在中土武学中的失位，玄慈方丈也深思熟虑，认为再扣留波罗星已无益
- 鸠摩智的绝世神功让整个大殿寂静无声，震慑众人
- 虚竹目睹鸠摩智的武功表演
- 鸠摩智使用“小无相功”掌控其他武功
- “小无相功”是道家武学，与佛教武功不同
- 众僧虽有疑虑，但不了解“小无相功”的威力和特点
- 虚竹想指出鸠摩智并非使用少林派绝技，但一直沉默不语
- 鸠摩智建议少林寺解散，引起众僧不满
- 鸠摩智意在瓦解少林寺，取得好处
- 一位僧人称赞大理天龙寺，玄渡大师受挫
- 虚竹救治玄渡大师，展示高超医术
- 群僧对虚竹的武功和行为产生疑问
- 庄聚贤要求少林派承认其为中原武林盟主，众高僧为此烦恼
- 虚竹师父慧轮在园中偷偷挑粪浇菜，众高僧不知
- 群僧对鸠摩智的指法产生质疑，鸠摩智恍然大悟
- 主人公是一个天生聪明、运气好的人，从小就遇到奇遇，从未败过。
- 他来到少林寺，想单枪匹马打败这座千年古刹。
- 他和虚竹进行了一场较量，虚竹使用了玄渡太师伯的拈花指和小无相功。
- 鸠摩智试了虚竹一招般若掌，虚竹用天山六阳掌将其化解。
- 鸠摩智要求虚竹使用少林派的功夫，虚竹只会罗汉拳和韦陀掌。
- 方丈让虚竹与鸠摩智过招，虚竹使用了韦陀掌的起手式灵山礼佛。
- 鸠摩智使用般若掌的上乘功夫，但最终不得不避战。
- 韦陀掌是少林派的基础武功
- 学韦陀掌后，可学习般若掌，需要数十年练习
- 般若掌是少林派七十二绝技之一，无穷无尽
- 鸠摩智与虚竹交手，使用多种少林派神功，虚竹只使用“黑虎偷心”一招
- “黑虎偷心”虽拙劣，但劲力不断增强，逼退鸠摩智
- 鸠摩智发现虚竹拳力中有小无相功，但未能发挥威力
- 鸠摩智使用少林绝技“龙爪功”抓住虚竹拳头
- 虚竹用“天山折梅手”反抓鸠摩智的左腕
- 鸠摩智用少林派武功攻击虚竹，虚竹用天山六阳掌化解
- 两人近身肉搏，掌力强劲，少林寺辈份较低的僧侣抵受不住，玄字辈高僧各运内力抗拒
- 虚竹用力抓住鸠摩智左腕，鸠摩智双掌连环变化、交互为用的诸般妙着便使不出来
- 两人僵持不下，众高僧生平从未见过这样的打法，群僧提心吊胆
- 虚竹渐渐掌握天山六阳掌的精妙，能在十招中反攻两三招
- 得到少林群僧的欢喜，但众人仍不想杀了鸠摩智，只为维护少林派的声誉
- 神山上人希望鸠摩智和虚竹两败俱伤，自己可以创立清凉寺的三门绝技
- 波罗星研习武功后感到不满足，想要学习更高阶的武学
- 虚竹使用逍遥派武学的狠辣招数，令旁观者惊惧
- 少林派历代均是走刚阳路子，制敌而非杀人
- 鸠摩智使用匕首偷袭虚竹，被旁观者谴责
```

可以看到这个中间层的输出还是非常准确的，问题主要出在进一步压缩上

进一步压缩之后，文本被总结成了


```
- 鸠摩智展示少林派七十二门绝技中的问题和自己的武功
- 虚竹目睹鸠摩智的武功表演，展示了自己的医术和武功
- 群僧对鸠摩智和虚竹的武功和行为产生疑问
- 主人公来到少林寺，与虚竹进行一场较量
- 两人使用少林派基础武功和七十二绝技进行交手
- 虚竹最终使用天山六阳掌化解鸠摩智的攻击
- 两人进行近身肉搏，众高僧生平从未见过这样的打法
- 虚竹最终掌握天山六阳掌的精妙，得到少林群僧的欢喜
- 神山上人和波罗星也有自己的武学追求
- 虚竹使用逍遥派武学的狠辣招数，引起旁观者惊惧
- 鸠摩智使用匕首偷袭虚竹，被谴责
```

其实我觉得这个bullet的方式是几个里面最好的了。


## Refine策略

这个refine策略是让我觉得最惊人（的差）的总结策略了

他的核心特点是把总结模版替换成了这个

```python
refine_template = (
    "您的工作是生成一个最终的总结\n"
    "我们已经提供了一个前半部分的现有的总结：{existing_answer}\n"
    "我们将通过增量的一些段落的上下文来补充现有的总结(只有在需要的情况下)。\n"
    "------------\n"
    "{text}\n"
    "------------\n"
    "根据新的上下文，完善原始的总结"
    "如果这个上下文不相关，可以返回原始的总结。"
)

refine_prompt = PromptTemplate(
    input_variables=["existing_answer", "text"],
    template=refine_template,
)
```

然后序贯得一段段总结。他最后给出的输出我非常吃惊。。。

```
在这场激烈的战斗中，虚竹和鸠摩智的武艺都得到了提升。虚竹通过领悟天山六
阳掌的精髓，开始有了反击的能力，而鸠摩智的攻势也因此略有顿挫。随着时间
的推移，虚竹越来越能够反击，少林群僧也因此欣喜。他们并不是想杀死鸠摩智
，而是为了维护少林派的声誉。虽然虚竹的天山六阳掌和鸠摩智的少林派武功形
成了僵持之局，但是虚竹逐渐掌握了进攻的主动权，让众人惊叹。在这个故事中
，武学、医学、宗教和政治相互交织，一个人的道德和信仰如何能够影响整个社
会。最终，虚竹成为了少林寺的一位神僧，而鸠摩智也为自己的行为所付出了代
价。然而，这场战斗也让人思考，对于武学的探索和研究，到底是为了何种目的
，是为了更高级的武功还是为了自己的私欲？这是值得深思的问题。
```

让你总结 谁让你给升华了。。。

其实这里面有几个点，我们从前面那个bullet可以看到

其实gpt对于4096限制内的总结是还可以的

但是对于小说这种比较难切段的内容，他不能很好的去切段

我觉得如果中间加一个大的parser，然后再总结可能会更好一些。

这个超长文本的总结看起来还有一段更长的路可以走。

---

## Summarization Checker



## Chat凉宫春日

对了，我最近在拿Chat凉宫春日交DataWhale举办的5月学习活动的作业

我们有一个针对Chat凉宫春日这个项目 招人的计划在这里 [招人计划](https://github.com/LC1332/Prophet-Andrew-Ng/blob/main/Hiring.md)

需要一些后端、前端和算法工程师，整个项目的ddl在6月6号和6月13号

如果你也正在参加DataWhale的课程或者这两天非常有空，可以联系我一起玩一下，做多做少都是缘，欢迎加入

另外昨天Chat凉宫春日在群里开放测了一下，大家反馈还都不错。不过有点费钱，如果你想测可以私信我 我发您链接。


## 关于骆驼

我们在积极寻求服务器资源（A100，A800的服务器）的捐赠，当然你也可以去我们的项目页找到[赞助链接](https://github.com/LC1332/Luotuo-Chinese-LLM#sponsorship)来对我们进行支持。所有的赞助资源将会用在服务器资源的购买、数据的获取、社区的正常运维和周边的发放。如果你有兴趣用中文复现上面的一些前沿工作，也欢迎和我们讨论。

[骆驼：开源中文大语言模型](https://github.com/LC1332/Luotuo-Chinese-LLM)

骆驼是我们的个人作业项目。如果你感觉这个文章对你有帮助，也欢迎到我们的骆驼项目主页为我们点上star。如果您没有github账号，也可以在知乎直接点赞。谢谢