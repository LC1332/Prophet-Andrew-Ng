# 【LangChain笔记11】打造属于自己的AutoGPT，利用LangChain和GPT最新模型构建工具组合助手

我原来想的标题叫做**跨时代的Agent机制，Reason+Act实现原理, 支持函数调用的OpenAI0613模型**，后来觉得起更大一点好像也很合理。

今天这是一篇比较重要的笔记，会涉及LangChain中的Agent机制，

以及对应的这个Agent的实现原理，即Reasoning+Act的原理

去讨论OpenAI的0613-turbo模型如何官方实现了这个Agent的机制

- [啰嗦的引言](#啰嗦的引言)
- [Agent机制初探](#Agent机制初探)
- [LangChain中Agent的初始化](#LangChain中Agent的初始化)
- [什么是Reasoning_And_Acting](#什么是Reasoning_And_Acting)
- [Zero_Shot_ReAct](#Zero_Shot_ReAct)


## 啰嗦的引言

这份笔记默认是[github的版本](https://github.com/LC1332/Prophet-Andrew-Ng/blob/main/langchain/%E6%9D%8E%E9%B2%81%E9%B2%81%E5%AD%A6LangChain%2011.md)，你去github看的话会看到所有的完整链接，知乎或者微信的版本可能链接是不全的。

对于这方面接触不深的同学来说，你完全可以把今天的内容理解成

“打造属于自己的AutoGPT，利用LangChain的Agent和GPT最新模型构建工具组合助手”

而且这么说也一点都没有高估这件事儿。首先我们之前已经提到LangChain里面有Tools这个概念

工具指的就是比如打开一个网页，计算一个公式，查询一个地区的天气，搜索wiki百科等原本不属于语言模型本身的功能

LangChain的Agent（智能体）允许把多种工具组合在一起，形成一个智能助手。

并且由语言模型，来对这些工具进行调度。这个事情背后的意义非常大，你可以认为大量的智能体工作

包括和人类沟通的同时操控一个机器人，或者由语言作为入口帮你做微信助手，这些应用，都可以被覆盖在这个框架下

这个框架很有可能是产生革命性应用的一个起点。而且让语言模型去调度和调用外部接口，显然超过了原有的

应用程序或者基于状态机的智能体设计的模式。几乎所有的智能应用都可以去重做一遍，并且会产生一些新的应用。

---

过往看过我们笔记的同学可能知道，我们正常情况下笔记会随着Sam Witteveen视频的顺序进行记录

不过这次笔记覆盖的内容是跳跃的，统一选取了和Agent比较相关的4个视频，分别是

9 - LangChain Agents - Joining Tools and Chains with Decisions https://www.youtube.com/watch?v=ziu87EXZVUE

20 - Building Custom Tools and Agents with LangChain (gpt-3.5-turbo) https://www.youtube.com/watch?v=biS8G8x8DdA

29 - Understanding ReACT with LangChain https://www.youtube.com/watch?v=Eug2clsLtFs

30 - OpenAI Functions + LangChain : Building a Multi Tool Agent https://www.youtube.com/watch?v=4KXK6c6TVXQ

这些视频暂时没有找到B站的搬运，我也懒得搬运了，同学们可以自己去源地址去看。

Sam在每个视频下面，都留有colab链接。大家也可以在读这个笔记的前后去看一看那些colab代码

这个笔记实际上是对着四个视频和对应代码内容的重组和实验。

---

## Agent机制初探

我们先来感受一下Agent的能力，这是一个结合了两个工具serpapi搜索引擎和llm-math作为工具的agent，我们使用一句话进行调用

```python
agent.run("中国所有省级行政区，如果每个省都能和广东产出一样的GDP，那么中国一年的GDP是多少亿？")
```

Agent的输出是这样的

<p align="center">
    <img src="https://github.com/LC1332/Prophet-Andrew-Ng/blob/main/figures/langchaiAgentGDPresult.jpg">
</p>

或者我们在加入`["serpapi", "llm-math","wikipedia","terminal"]`四个工具之后，如果我们去询问

```python
agent.run("钢之炼金术师中，最基本的法则是什么？")
```

Agent的输出是这样的

<p align="center">
    <img src="https://github.com/LC1332/Prophet-Andrew-Ng/blob/main/figures/langchaiAgentAlchemist.jpg">
</p>

最终的回答是

```
The basic law in Fullmetal Alchemist is the law of equivalent exchange: "Humankind cannot gain anything without first giving something in return."
```

虽然说是英语的，但可以说是非常正确了。

我们可以看到，我们可以把数学工具、wiki和搜索引擎这些都集成到Agent中，这样语言模型就可以获得大大超越本来ChatGPT的能力。

在这里我要加一个吐槽。很多非从业者总觉得ChatGPT不会数学，或者ChatGPT回答自己某个方向的专业问题，结果发现是错的。因为这些原因就觉得ChatGPT在短期不会有很大的应用发展。这种看法在我看来是肤浅的。尽管在测评大语言模型综合能力的时候，数学测评或者某些方向的专业测评，可以从一些角度上去评估大语言模型的部分能力。但是在实际应用的时候，语言模型完全可以通过这些方法来集成调用不同的工具，来克服数学或者逻辑上的一些弱点，甚至在短期（指的是你看完这个笔记之后），就可以立刻通过API方式连接上专业知识库，来覆盖这部分的短板。所以靠ChatGPT在问答中无法解决专业问题或者无法解特定难度的数学，就去推断语言模型的应用能力有限，是一种肤浅的推断。

## LangChain中Agent的初始化

让我们绕回我们的笔记，前面我们直接给出了call Agent的部分。这里我们来展示一下初始化这么一个agent有多么简单，你差不多要做的就是这样

```python
from langchain.agents import load_tools
from langchain.agents import initialize_agent
from langchain.chat_models import ChatOpenAI

llm = ChatOpenAI(model_name='gpt-3.5-turbo')
tools = load_tools(["serpapi", "llm-math"], llm=llm)

agent = initialize_agent(tools,
                         llm,
                         agent="zero-shot-react-description",
                         verbose=True)
```

这样就初始化完成了一个agent。这个agent中我们引入了两个工具 一个是serpapi，实际上这个就是谷歌搜索的API，对于免费用户来说，一天好像有几千次可以的调用。我看后面Sam用duckduckGo来做搜索也比较多。另一个工具是llm-math，顾名思义就是结合语言模型去做数学的。

在这两个工具的结合下，语言模型构成的智能体就结合了搜索引擎和数学的能力。

前面提到的第二个例子里面还额外使用了Terminal和wiki百科。原则上你也可以授权这东西对你的文档进行管理，应该都是有办法实现的。

所有LangChain默认支持的Tools在这个网页是可以看到的。

https://python.langchain.com/docs/modules/agents/tools/integrations/apify

包括了DuckDuck搜索、文件管理、知识图谱、wolframalpha、hugging face调用等非常多功能

甚至还有一个叫什么Human as a tool的。另外显然也支持自己的函数定义进去。

## 什么是Reasoning_And_Acting

这个使用简单到不可思议，这意味着我只要在前面提到的页面随便选取一些工具，就可以用GPT来组成一个智能体，

这个智能体会根据你的需求来组合你之前注册的工具。在我第一次看到这些例子的时候我其实挺震惊的。

这背后的技术实际上来自于这篇paper， ReAct: Synergizing Reasoning and Acting in Language Models。这篇论文的作者主要都来自于Google Brain。

<p align="center">
    <img src="https://github.com/LC1332/Prophet-Andrew-Ng/blob/main/figures/agentDiagram.png">
</p>

看他的图还是有一点抽象的。本质上ReAct是Reasoning和Acting两个词的组合。前者其实来自于CoT，类似于让GPT做事情的时候总是要“让我想一想”，然后把操作步骤先给说清楚。Acting是让语言模型去调度外部API的过程。

这里我们不如直接去看例子，Sam在21课这里给出了4个人工的例子，我们这里放出两个

```
Question: 东科罗拉多造山运动扩展到的区域的海拔高度范围是多少？
Thought: 我需要搜索科罗拉多造山运动，找到东部区域，然后找到该区域的海拔高度范围。
Action: Search[科罗拉多造山运动]
Observation: 科罗拉多造山运动是科罗拉多州及周边地区的一次山脉建造活动（造山运动）。
Thought: 没有提到东部区域。我需要查找东部区域。
Action: Lookup[东部区域]
Observation: （结果 1/1）东部区域扩展到高原地带，被称为中央平原造山运动。
Thought: 东科罗拉多造山运动扩展到了高原地带。因此，需要搜索高原并找到它的海拔高度范围。
Action: Search[高原]
Observation: 高原是指两个不同的地区之一。
Thought: 我需要搜索“高原（美国）”。
Action: Search[高原（美国）]
Observation: 高原是大平原的一个分区。从东向西，高原的海拔高度从约1,800英尺上升到约7,000英尺（550米至2,130米）。[3]
Thought: 高原的海拔高度从约1,800英尺上升到约7,000英尺，因此答案是1,800到7,000英尺。
Action: Finish[1,800到7,000英尺]
```

另一个是

```
Question: Nicholas Ray和Elia Kazan有什么共同的职业？
Thought: 我需要搜索Nicholas Ray和Elia Kazan，找到他们的职业，然后找到他们共同的职业。
Action: Search[Nicholas Ray]
Observation: Nicholas Ray（1911年8月7日-1979年6月16日）是一位美国电影导演，编剧和演员，以1955年的电影《无因的反叛》而闻名。
Thought: Nicholas Ray的职业是导演，编剧和演员。我需要接下来搜索Elia Kazan并找到他的职业。
Action: Search[Elia Kazan]
Observation: Elia Kazan是一位美国电影和戏剧导演，制片人，编剧和演员。
Thought: Elia Kazan的职业是导演，制片人，编剧和演员。因此，Nicholas Ray和Elia Kazan共同的职业是导演，编剧和演员。
Action: 完成[导演，编剧，演员]
```

看到这里我们其实有一些感觉了。其实就是让语言模型去习惯Throught-Action-Observation这样一个循环。并且在Action的步骤，可以去使用Action:函数名[参数]的形式，对外部的API进行调用。

并且在这里其实，调用之后会把返回的结果作为Observation，语言模型会进一步观察Observation中的结果，总结自己看到了什么，并且在Thought步骤决定自己后面一步的行动。明白了这个之后我们再看论文中的例子就会觉得很顺利。作者在主页上每个benchmark放了一个例子，这个是HotpotQA的例子

<p align="center">
    <img src="https://github.com/LC1332/Prophet-Andrew-Ng/blob/main/figures/langchainHotpotqa.png">
</p>

这是一个Agent玩文字游戏的benchmark的例子

<p align="center">
    <img src="https://github.com/LC1332/Prophet-Andrew-Ng/blob/main/figures/langchainAlfworld.png">
</p>


另外Google Brain他们也在4个benchmark上做了实验，结果当然是显著提升

<p align="center">
    <img src="https://github.com/LC1332/Prophet-Andrew-Ng/blob/main/figures/langchainReactResult.png">
</p>

话说能用搜索和不能用搜索也差太大了，AI也学会开卷考了嗨。你们之前哪个说ChatGPT考试相当于开卷考来着的，要不用这个策略重新试试看？

当然我们这里注意到，在给出例子的情况下，这个策略会非常有效，那么我们前面设计的`zero-shot-react-description`是什么意思呢？

## Zero_Shot_ReAct

根据[LangChain的官方文档](https://www.pinecone.io/learn/langchain-agents/#:~:text=Zero%2Dshot%20means%20the%20agent,solely%20on%20the%20tool's%20description%20.) 

```
Zero-shot means the agent functions on the current action only — it has no memory. 
It uses the ReAct framework to decide which tool to use, based solely on the tool’s description.
```

哦，原来这个zero-shot-react是之前ReACT标准论文的一个弱化版本，即不使用例子，仅考虑Tools的描述。同时，这个东西有三个变体，分别是

+ If you're using a text LLM, first try zero-shot-react-description, aka. the MRKL agent for LLMs.
+ If you're using a Chat Model, try chat-zero-shot-react-description, aka. the MRKL agent for Chat Models.
+ If you're using a Chat Model and want to use memory, try chat-conversational-react-description, the Conversational agent.

也就是如果你考虑用turbo模型的时候，你可以考虑去用chat-zero-shot-react-description。而如果你希望他对过往的记录有记忆，可以使用chat-conversational-react-description来进行。

在这几节课里面，Sam解释了

