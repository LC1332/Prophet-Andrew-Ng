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