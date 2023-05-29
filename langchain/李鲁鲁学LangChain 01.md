# 李鲁鲁学LangChain 01 LLMs + Prompting

这是李鲁鲁的学习笔记，原来是Sam Witteveen的笔记

项目地址 https://github.com/LC1332/Prophet-Andrew-Ng

原视频地址 https://www.youtube.com/watch?v=J_0qvRt4LNk

增加了李鲁鲁学习时候的吐槽

上这个课程的原因是Andrew的课程没有去照顾langchain的使用。而Sam的课看起来还挺好的，并且每节课都有colab

---

## 初识LangChain

第一次跑LangChain的代码是在实践 [Camel](https://github.com/LC1332/Chinese-Camel) 这个项目

这个项目是使用了LangChain的Agent去写的，如果你点开去看我录的视频，你会发现这中间其实发生了一次Request Error

但是被LangChain的询问接口自动重试了。然后在搭建 [Chat凉宫春日](https://github.com/LC1332/Prophet-Andrew-Ng/blob/main/prophet-code/haruhiLangChain.ipynb)的时候，一开始单纯使用了openai.ChatCompletion接口，让同学们测发现挺卡的，还会因为OpenAI服务器的原因

经常挂掉，后来换成LangChain的接口就好很多了。话说这个Chat凉宫春日我还在内测，稍后争取放到HuggingFace上面去，如果你是老二次元的话也可以私信我试一下。

---

## Sam老师的LangChain第一课

这里我先让GPT老师对着Sam的视频进行总结，给我总结出了一些标题

---

## LangChain: 利用 Few Shot Prompt Templates 构建基于大型语言模型的应用

- Few Shot Prompting 和 Prompt Templates
    - 用 Prompt Templates 解决问题
    - 问答系统
    - 分类化文本
- [餐厅命名案例](#餐厅命名案例)
    - Prompt Templates 和语言模型的连接
    - Few-shot prompt templates 的使用
- 利用 Few Shot Prompt Templates 创建词汇反义词查询服务
- 智能化设计和下一步计划

---

其实我觉得总结的还不错，考虑之后用LangChain来搭建一个总结youtube字幕的程序吧（作业+1）

## 为什么要LangChain

这里其实我觉得Sam的视频讲得挺好的，我们都知道语言模型只是一个条件概率的语言生成模型，

也就是根据P(输出|Prompt)，来对语言进行输出。这个时候语言模型对知识的记忆能力是比较局限的。

我们往往还要额外搭建知识体系。这个时候LangChain就出现了。

机器翻译和总结：LangChain 是一个新的 Python 库，可以让我们构建完整功能的应用程序，与普通软件堆栈进行交互，并管理使用大型语言模型和提示。大型语言模型非常适合从头开始或通过条件生成生成新的内容，但它们不能以正常方式访问传统软件堆栈。LangChain 允许我们与外部数据源和传统软件堆栈集成，使得使用大型语言模型构建应用程序更加容易。LangChain 中的所有内容都围绕提示进行构建，在大型语言模型的输出上具有巨大的影响。该库正在成为围绕大型语言模型构建应用程序的事实标准，并已经由一大群人快速开发。


## LangChain的安装和使用

安装，如果colab就加个惊叹号

```
pip -q install openai langchain huggingface_hub
```

设置token

```python
import os

os.environ['OPENAI_API_KEY'] = 'sk-'
os.environ['HUGGINGFACEHUB_API_TOKEN'] = 'hf_'
```

这里我要废话一下，和我之前代码习惯不一样，这里必须设置到OPENAI_API_KEY这个环境变量，而不能简单实用openai.api_key这个python动态环境中的变量。

这里我觉得有点麻烦，如果我以后单个端要多重token的话不知道是不是支持


### 使用GPT3来进行生成

这个我们之前已经试过很多次了，在Chat凉宫春日项目中，已经实践上了

```
from langchain.llms import OpenAI

llm = OpenAI(model_name='text-davinci-003', 
             temperature=0.9, 
             max_tokens = 256)

text = "Why did the duck cross the road?"

print(llm(text))
```

我的输出是

```
To get to the other side.
```

注意这里gpt-turbo-3.5是不能用这个方法简单初始化的。

### 使用T5-Flan-XL的模型

这里我有一些意外，因为我之前没有用过HuggingFaceHub

```python
from langchain.llms import HuggingFaceHub

llm_hf = HuggingFaceHub(
    repo_id="google/flan-t5-xl",
    model_kwargs={"temperature":0.9 }
)

text = "Why did the chicken cross the road?"

print(llm_hf(text))
```

我这里运行了一直没有出结果，我觉得应该是因为我的HuggingFace不是收费账号

如果HF是收费账号应该是会有返回的。

## 餐厅命名案例

```python
from langchain import PromptTemplate


restaurant_template = """
我希望你能充当一个新餐厅的命名顾问。

返回一个餐厅名称列表。每个名称都应该简短，易记，与你正在命名的餐厅类型相关。

如果是{restaurant_description}类型的餐厅，有哪些好的命名方式？
"""

prompt = PromptTemplate(
    input_variables=["restaurant_description"],
    template=restaurant_template,
)

# An example prompt with one input variable
prompt_template = PromptTemplate(input_variables=["restaurant_description"], template=restaurant_template)

description = "一个供应新鲜的羊肉肉串和其他希腊食物的希腊餐厅"
description_02 = "一个以棒球纪念品为主题的汉堡店"
description_03 = "一个咖啡馆，有现场重金属音乐和纪念品"

## to see what the prompt will be like
prompt_template.format(restaurant_description=description)
```

到这一步会输出

```
\n我希望你能充当一个新餐厅的命名顾问。\n\n返回一个餐厅名称列表。每个名称都应该简短，易记，与你正在命名的餐厅类型相关。\n\n如果是一个供应新鲜的羊肉肉串和其他希腊食物的希腊餐厅类型的餐厅，有哪些好的命名方式？\n
```

这里langchain实际上实现了prompt template和实际变量的分离。

继续运行后面的代码

```python
## querying the model with the prompt template
from langchain.chains import LLMChain


chain = LLMChain(llm=llm, prompt=prompt_template)

# Run the chain only specifying the input variable.
print(chain.run(description_03))
```

模型的输出

```
1. 重金属咖啡：重音咖啡、重金属咖啡馆。 
2. 音乐咖啡屋：音乐之路、音乐花园、乐林咖啡。 
3. 纪念品咖啡：忆印咖啡、收藏馆咖啡、宝藏咖啡。
```

## Few Shot Learning

在GPT使用的时候，Few shot (in context) learning是非常重要的

（即给问题的时候给两个例子答案）

GPT老师对这一段的总结

```
Few Shot Prompt Template可以在模型中加入多组示例数据，并在每组示例数据中对输入和输出进行标注，这样模型就可以利用这些数据进行学习，并实现更加个性化的数据处理。

作者通过一个示例，展示了如何使用Few Shot Prompt Template来创建一个词汇的反义词查询服务。通过传入多组示例数据和设置相应的前缀和后缀，作者最终得到了一个完整的查询模型，并演示了如何输入一个单词来查询它的反义词。
```

其实FewShotPromptTemplate就是一个可以用来组织prompt的一个范式

```python
from langchain import PromptTemplate, FewShotPromptTemplate

# First, create the list of few shot examples.
examples = [
    {"word": "happy", "antonym": "sad"},
    {"word": "tall", "antonym": "short"},
]

# Next, we specify the template to format the examples we have provided.
# We use the `PromptTemplate` class for this.
example_formatter_template = """
Word: {word}
Antonym: {antonym}\n
"""

example_prompt = PromptTemplate(
    input_variables=["word", "antonym"],
    template=example_formatter_template,
)
```

这样其实就完成了对example_prompt的组织。然后后面这个设计其实还挺细心的，让我们让GPT老师来翻译后面

```python
few_shot_prompt = FewShotPromptTemplate(
    # 这是我们要插入到提示中的样例。
    examples=examples,
    # 这是我们在将样例插入到提示时想要格式化样例的方式。
    example_prompt=example_prompt,
    # 前缀是在样例之前放置的一些文本。通常，它包含说明信息。
    prefix="给出每个输入的反义词",
    # 后缀是在样例之后放置的一些文本。通常，用户的输入会被放在这里。
    suffix="单词：{input}\n反义词：",
    # input_variables 是整个提示想要的变量。
    input_variables=[“input”],
    # example_separator 是我们将用来连接前缀、样例和后缀的字符串。
    example_separator="\n\n",
)

# We can now generate a prompt using the `format` method.
print(few_shot_prompt.format(input="big"))
```

这里的输出

```
给出每个输入的反义词

Word: happy
Antonym: sad

Word: tall
Antonym: short

单词：big
反义词：
```

这里我们发现一个问题，也就是你example_formatter_template 和 suffix 是要一致

另外从这个输出我们可以看到他这个本质上是把数据组织成了

```
prefix [example_separator example_prompt]*N example_separator suffix
```

这个形式，当组织完这些，你再去要一个新的词的反义词，你就可以用这个指令

```python
from langchain.chains import LLMChain

chain = LLMChain(llm=llm, prompt=few_shot_prompt)

# Run the chain only specifying the input variable.
print(chain.run("Big"))
```

这样他输出就会是

```
Small
```

其实如果我们单看最后三行代码，整体还是比较简洁的，并且他直接把整体的prompt，又实现成为一个LLMChain，这样只要LLMChain.run(输入)，就可以要到对应的输出。

---

我感觉相比Andrew的课程来说，Sam一节课的内容是偏少的。所以一次笔记打算记两节课的内容

## Tools and Chains

第二节课主要会有关Tools和Chains

Tools就是LangChain的Chain中的某个单个的环节

就好像一个单句的python语句

他这里Sam说Tool还支持像GPT4能用的那些比如沃尔夫勒姆 或者 搜索引擎那些工具

Chain = PromptTemplate + LLM

Chain支持LLM->API->LLM->PAL->LLM的应用

（鲁鲁的comments：

```python
from langchain.chains import SimpleSequentialChain, SequentialChain

full_chain = SimpleSequentialChain(chains=[fact_extraction_chain, investor_update_chain], verbose=True)

response = full_chain.run(article)
```

其实就是为了引出这样一个用法，Sam讲得还是比较细的）

其实就是能够吧一个tool的输出 组织成下一个tool的输入

有三种不同类型的Chain

+ Generic Chain

    + 第一类最常见的是LLMChain，前面已经见过了

    + 第二常见的是Transformation Chain，这个可以理解为组织字符串输入，运行一些python代码再给输出了

    + 第三Sequential Chain，其实就是组合模式，一个chain也可以看成工具

+ Utility Chain

    + Pal Chain 将一个Reasoning问题转化为python代码

    + SQL Database Chain 顾名思义

    + Bash 可以运行Bash命令

    + Request Chain

+ Asyncronous Chain

其实我看他第二节课的Notebook主要还是这些Chain的单独使用。例子里面包括了简单的LLMChain，SequentialChain和PALChain

## 使用一个LLMChain做要点提取

他这里还是先使用了GPT3的接口，没有去用turbo

```python
from langchain.prompts import PromptTemplate
from langchain.llms import OpenAI
from langchain.chains import LLMChain

llm = OpenAI(model_name='text-davinci-003', 
             temperature=0, 
             max_tokens = 256)

article = '''Coinbase, the second-largest crypto exchange by trading volume, released its Q4 2022 earnings on Tuesday, giving shareholders and market players alike an updated look into its financials. In response to the report, the company's shares are down modestly in early after-hours trading.In the fourth quarter of 2022, Coinbase generated $605 million in total revenue, down sharply from $2.49 billion in the year-ago quarter. Coinbase's top line was not enough to cover its expenses: The company lost $557 million in the three-month period on a GAAP basis (net income) worth -$2.46 per share, and an adjusted EBITDA deficit of $124 million.Wall Street expected Coinbase to report $581.2 million in revenue and earnings per share of -$2.44 with adjusted EBITDA of -$201.8 million driven by 8.4 million monthly transaction users (MTUs), according to data provided by Yahoo Finance.Before its Q4 earnings were released, Coinbase's stock had risen 86% year-to-date. Even with that rally, the value of Coinbase when measured on a per-share basis is still down significantly from its 52-week high of $206.79.That Coinbase beat revenue expectations is notable in that it came with declines in trading volume; Coinbase historically generated the bulk of its revenues from trading fees, making Q4 2022 notable. Consumer trading volumes fell from $26 billion in the third quarter of last year to $20 billion in Q4, while institutional volumes across the same timeframe fell from $133 billion to $125 billion.The overall crypto market capitalization fell about 64%, or $1.5 trillion during 2022, which resulted in Coinbase's total trading volumes and transaction revenues to fall 50% and 66% year-over-year, respectively, the company reported.As you would expect with declines in trading volume, trading revenue at Coinbase fell in Q4 compared to the third quarter of last year, dipping from $365.9 million to $322.1 million. (TechCrunch is comparing Coinbase's Q4 2022 results to Q3 2022 instead of Q4 2021, as the latter comparison would be less useful given how much the crypto market has changed in the last year; we're all aware that overall crypto activity has fallen from the final months of 2021.)There were bits of good news in the Coinbase report. While Coinbase's trading revenues were less than exuberant, the company's other revenues posted gains. What Coinbase calls its "subscription and services revenue" rose from $210.5 million in Q3 2022 to $282.8 million in Q4 of the same year, a gain of just over 34% in a single quarter.And even as the crypto industry faced a number of catastrophic events, including the Terra/LUNA and FTX collapses to name a few, there was still growth in other areas. The monthly active developers in crypto have more than doubled since 2020 to over 20,000, while major brands like Starbucks, Nike and Adidas have dived into the space alongside social media platforms like Instagram and Reddit.With big players getting into crypto, industry players are hoping this move results in greater adoption both for product use cases and trading volumes. Although there was a lot of movement from traditional retail markets and Web 2.0 businesses, trading volume for both consumer and institutional users fell quarter-over-quarter for Coinbase.Looking forward, it'll be interesting to see if these pieces pick back up and trading interest reemerges in 2023, or if platforms like Coinbase will have to keep looking elsewhere for revenue (like its subscription service) if users continue to shy away from the market.
'''

fact_extraction_prompt = PromptTemplate(
    input_variables=["text_input"],
    template="Extract the key facts out of this text. Don't include opinions. Give each fact a number and keep them short sentences. :\n\n {text_input}"
)

fact_extraction_chain = LLMChain(llm=llm, prompt=fact_extraction_prompt)

facts = fact_extraction_chain.run(article)

print(facts)
```

这里对一个相对比较长（3000个字符）的文本去做一个信息提取，输出如下

```
1. Coinbase released its Q4 2022 earnings on Tuesday.
2. Coinbase generated $605 million in total revenue in Q4 2022.
3. Coinbase lost $557 million in the three-month period on a GAAP basis.
4. Wall Street expected Coinbase to report $581.2 million in revenue.
5. Coinbase's stock had risen 86% year-to-date before its Q4 earnings were released.
6. Consumer trading volumes fell from $26 billion in Q3 2022 to $20 billion in Q4 2022.
7. Institutional volumes across the same timeframe fell from $133 billion to $125 billion.
8. The overall crypto market capitalization fell about 64%, or $1.5 trillion during 2022.
9. Coinbase's total trading volumes and transaction revenues fell 50% and 66% year-over-year, respectively.
10. Trading revenue at Coinbase fell from $365.9 million in Q3 2022 to $322.1 million in Q4 2022.
11. Coinbase's "subscription and services revenue" rose from $210.5 million in Q3 2022 to $282.8 million in Q4 2022.
12. Monthly active developers in crypto have more than doubled since
```

这是一个最简单的Chain的形式。简单连接了输入和语言模型。 PromptTemplate在第一节课已经实践了，你可以理解成这是一种简单的去组合格式化字符串的方式。

这里我不敢直接把文本全翻译成中文，是因为他这个程序后面都是连接的 facts变量很快要作为后面的输入，所以我们先跑一遍他基础的程序。再来看怎么改。


## 第二步

在第二步中，会把facts再作为输入

``` python
investor_update_prompt = PromptTemplate(
    input_variables=["facts"],
    template="You are a Goldman Sachs analyst. Take the following list of facts and use them to write a short paragrah for investors. Don't leave out key info:\n\n {facts}"
)

investor_update_chain = LLMChain(llm=llm, prompt=investor_update_prompt)

investor_update = investor_update_chain.run(facts)

print(investor_update)
```

输出为

```
Coinbase released its Q4 2022 earnings on Tuesday, reporting total revenue of $605 million, which was slightly above Wall Street's expectations of $581.2 million. Despite the positive revenue figure, Coinbase reported a GAAP loss of $557 million in the three-month period. Coinbase's stock had risen 86% year-to-date before its Q4 earnings were released. Consumer trading volumes fell from $26 billion in Q3 2022 to $20 billion in Q4 2022, while institutional volumes across the same timeframe fell from $133 billion to $125 billion. The overall crypto market capitalization fell about 64%, or $1.5 trillion during 2022, leading to a 50% and 66% year-over-year decline in Coinbase's total trading volumes and transaction revenues, respectively. Trading revenue at Coinbase fell from $365.9 million in Q3 2022 to $322.1 million in Q4 2022, while its "subscription and services revenue" rose from $210.5 million in Q3 2022 to $282.8 million in Q4 2022. Despite the overall decline in the crypto market, monthly active developers in crypto have more than doubled since the start of 2022.
```

当然这里让我感到有点无聊的是现在这个例子里面 传递的都是text

这样即使我们不使用LangChain的范式也可以很方便的自己写出来。

当然后面的课程会慢慢复杂起来，让我们事无巨细地一点点继续刷这个教程吧。


## 将事实转化为知识图谱的三元组

使用一个其他的prompt可以把刚才的facts变成一个三元组

```python
triples_prompt = PromptTemplate(
    input_variables=["facts"],
    template="Take the following list of facts and turn them into triples for a knowledge graph:\n\n {facts}"
)

triples_chain = LLMChain(llm=llm, prompt=triples_prompt)

triples = triples_chain.run(facts)

print(triples)
```

输出是这样的

```
1. (Coinbase, released, Q4 2022 earnings)
2. (Coinbase, generated, $605 million total revenue)
3. (Coinbase, lost, $557 million GAAP basis)
4. (Wall Street, expected, $581.2 million revenue)
5. (Coinbase's stock, risen, 86% year-to-date)
6. (Consumer trading volumes, fell, $26 billion Q3 2022)
7. (Institutional volumes, fell, $133 billion)
8. (Overall crypto market capitalization, fell, $1.5 trillion)
9. (Coinbase's total trading volumes, fell, 50% year-over-year)
10. (Coinbase's transaction revenues, fell, 66% year-over-year)
11. (Trading revenue, fell, $365.9 million Q3 2022)
12. (Coinbase's subscription and services revenue, rose, $210.5 million Q3 2022)
13. (Monthly active developers in crypto, more than doubled, since start of 2022)
```

## 用SequentialChain连接多个Chain

```python
from langchain.chains import SimpleSequentialChain, SequentialChain

full_chain = SimpleSequentialChain(chains=[fact_extraction_chain, investor_update_chain], verbose=True)

response = full_chain.run(article)
```

这实际上就把刚才的关键信息提取和转写 合并成一个chain了

输出的时候，Chain上的每个tool会用不同的颜色输出。

接着response会给最终的输出

其实这里就是这个视频的核心思想。

## PALChain

Sam对这个PAL Chain很感兴趣，还说要额外做一个视频

PALChain会用到code-davinci-002的模型（我估计StarCoder也有办法接）。这个除了copilot以外我也没有用过这个模型

不过不幸的消息是Codex这个接口已经被OpenAI弃用了，根据官网的提示我们仍然去使用text-davinci-003

```python
from langchain.chains import PALChain

llm = OpenAI(model_name='text-davinci-003', 
             temperature=0, 
             max_tokens=512)

pal_chain = PALChain.from_math_prompt(llm, verbose=True)             
```

这里使用了一个LangChain在PALChain里面预置的一个prompt

如果你打印pal_chain这个对象，可以看到这个prompt，里面都是

```
Q: 应用题
solution in Python: 代码
```

格式的例子

然后输入问题（这个问题和PAL paper中的一致）

```python
question_02= "The cafeteria had 23 apples. If they used 20 for lunch and bought 6 more, how many apples do they have?"
pal_chain.run(question_02)
```

```python
> Entering new PALChain chain...
def solution():
    """The cafeteria had 23 apples. If they used 20 for lunch and bought 6 more, how many apples do they have?"""
    apples_initial = 23
    apples_used = 20
    apples_bought = 6
    apples_total = apples_initial + apples_bought - apples_used
    result = apples_total
    return result

> Finished chain.
9
```

Sam在这里给了另一个问题

```python
question = "Jan has three times the number of pets as Marcia. Marcia has two more pets than Cindy. If Cindy has four pets, how many total pets do the three have?"
```

给的答案是

```python
> Entering new PALChain chain...
def solution():
    """Jan has three times the number of pets as Marcia. Marcia has two more pets than Cindy. If Cindy has four pets, how many total pets do the three have?"""
    cindy_pets = 4
    marcia_pets = cindy_pets + 2
    jan_pets = marcia_pets * 3
    total_pets = cindy_pets + marcia_pets + jan_pets
    result = total_pets
    return result

> Finished chain.
28
```

