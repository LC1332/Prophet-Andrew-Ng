# 【LangChain笔记11】打造属于自己的AutoGPT，利用LangChain和GPT最新0613构建工具组合助手

我原来想的标题叫做**跨时代的Agent机制，Reason+Act实现原理, 支持函数调用的OpenAI0613模型**，后来觉得起更大一点好像也很合理。

今天这是一篇比较重要的笔记，会涉及LangChain中的Agent机制，

以及对应的这个Agent的实现原理，即Reasoning+Act的原理

去讨论OpenAI的0613-turbo模型如何官方实现了这个Agent的机制

<p align="center">
    <img src="https://github.com/LC1332/Prophet-Andrew-Ng/blob/main/figures/langchain11LOGO.png">
</p>


- [啰嗦的引言](#啰嗦的引言)
- [Agent机制初探](#Agent机制初探)
- [LangChain中Agent的初始化](#LangChain中Agent的初始化)
- [什么是Reasoning_And_Acting](#什么是Reasoning_And_Acting)
- [Zero_Shot_ReAct](#Zero_Shot_ReAct)
- [LangChain结合0613模型](#LangChain结合0613模型)
- [一个完整的Agent](#一个完整的Agent)


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

也就是如果你考虑用turbo模型的时候，你可以考虑去用chat-zero-shot-react-description。而如果你希望他对过往的记录有记忆，可以使用chat-conversational-react-description来进行。到这里我们可以看看在自定义Tools的时候，是怎么定义的

```python
from langchain.agents import Tool

search = DuckDuckGoSearchTool()

tools = [
    Tool(
        name = "search",
        func=search.run,
        description="useful for when you need to answer questions about current events. You should ask targeted questions"
    )
]
```

当然DuckDuckGoSearch在LangChain中本身就可以用DuckDuckGoSearchRun()来初始化，这里Sam也给出了另一个自定义Tools的例子

```python
def meaning_of_life(input=""):
    return 'The meaning of life is 42 if rounded but is actually 42.17658'

life_tool = Tool(
    name='Meaning of Life',
    func= meaning_of_life,
    description="Useful for when you need to answer questions about the meaning of life. input should be MOL "
)
```

也就是说，一个Tool需要有三个必要的属性。名字定义了这个Tool的名字，这样语言模型可以用Action: 名字[参数]的形式，来对你定义的Tool进行调用。

func告诉了系统如果语言模型想要调用这个函数，应该使用哪个函数来调用。

而description则会影响语言模型是否会调用这个对应的Tool。在定义完这个Tool之后，Sam用这两个工具组合搜索，形成了一个基础的agent

```python
from langchain.agents import initialize_agent

tools = [search, random_tool, life_tool]

# conversational agent memory
memory = ConversationBufferWindowMemory(
    memory_key='chat_history',
    k=3,
    return_messages=True
)

# create our agent
conversational_agent = initialize_agent(
    agent='chat-conversational-react-description',
    tools=tools,
    llm=turbo_llm,
    verbose=True,
    max_iterations=3,
    early_stopping_method='generate',
    memory=memory
)
```

这里语言模型使用的是GPT3.5-turbo，对应的agent也换成了前面提到的变体chat-conversational-react。注意这仍然是一个zero-shot的使用。后面会提到zero-shot大致是怎么用的。我们来看看这个agent能不能正确使用这三个工具。

首先测试下搜索

```python
conversational_agent("What time is it in London?")
```

LangChain的输出是这样的

```js
> Entering new AgentExecutor chain...
{
    "action": "DuckDuckGo Search",
    "action_input": "time in london"
}
Observation: Current local time in United Kingdom - England - London. Get London's weather and area codes, time zone and DST. Explore London's sunrise and sunset, moonrise and moonset. Sunrise, sunset, day length and solar time for London. Sunrise: 05:52AM. Sunset: 08:06PM. Day length: 14h 14m. Solar noon: 12:59PM. The current local time in London is 59 minutes ahead of apparent solar time. Current local time in London and DST dates in 2023. Local time--:--BST AM/PM 24 hours . London time change . Next time change is in 6 months and 4 days, set your clock back 1 hour. London summer time (DST) in 2023. Daylight saving time 2023 in London begins at 1:00 AM on Sunday, March 26 ... Sunrise, sunset, day length and solar time for London. Sunrise: 05:42AM. Sunset: 08:15PM. Day length: 14h 33m. Solar noon: 12:58PM. The current local time in London is 58 minutes ahead of apparent solar time. The graph above illustrates clock changes in London during 2023. London in GMT Time Zone. London uses Greenwich Mean Time (GMT) during standard time and British Summer Time (BST) during Daylight Saving Time (DST), or summer time.. The Difference between GMT and UTC. In practice, GMT and UTC share the same time on a clock, which can cause them to be interchanged or confused.
Thought:{
    "action": "Final Answer",
    "action_input": "The current local time in London is 59 minutes ahead of apparent solar time. Sunrise: 05:52AM. Sunset: 08:06PM. Day length: 14h 14m. Solar noon: 12:59PM. London uses Greenwich Mean Time (GMT) during standard time and British Summer Time (BST) during Daylight Saving Time (DST), or summer time."
}

> Finished chain.
{'input': 'What time is it in London?',
 'chat_history': [],
 'output': 'The current local time in London is 59 minutes ahead of apparent solar time. Sunrise: 05:52AM. Sunset: 08:06PM. Day length: 14h 14m. Solar noon: 12:59PM. London uses Greenwich Mean Time (GMT) during standard time and British Summer Time (BST) during Daylight Saving Time (DST), or summer time.'}
```

看起来还基本可以用，然后再来测试一下随机数

```python
conversational_agent("Can you give me a random number?")
```

LangChain的返回是这样的

```js
> Entering new AgentExecutor chain...
{
    "action": "Random number",
    "action_input": "random"
}
Observation: 4
Thought:{
    "action": "Final Answer",
    "action_input": "The response to your last comment was a random number, which was 4."
}

> Finished chain.
{'input': ... ,
 'output': 'The response to your last comment was a random number, which was 4.'}
 ```

当然，到后面生命的意义就出现了问题了

```python
conversational_agent("What is the meaning of life?")
```

LangChain的输出是这样的

```js
> Entering new AgentExecutor chain...
{
    "action": "Final Answer",
    "action_input": "The meaning of life is a philosophical question that has been debated by scholars, theologians, and philosophers for centuries. There is no one definitive answer to this question, as it is largely a matter of personal belief and interpretation. Some people believe that the meaning of life is to seek happiness, while others believe that it is to fulfill a specific purpose or destiny. Ultimately, the meaning of life is a deeply personal and subjective question that each individual must answer for themselves."
}

> Finished chain.
{'input': ... ,
 'output': 'The meaning of life is a philosophical question that has been debated by scholars, theologians, and philosophers for centuries. There is no one definitive answer to this question, as it is largely a matter of personal belief and interpretation. Some people believe that the meaning of life is to seek happiness, while others believe that it is to fulfill a specific purpose or destiny. Ultimately, the meaning of life is a deeply personal and subjective question that each individual must answer for themselves.'}
```

可以看到在这里其实Agent没有去调用meaning_of_life函数，而是自己对生命的意义进行了回答

这里我们期望的输出是去回答42或者一开始指定的42.17658。然而ChatGPT没有去调用这个函数。

在这几节课里面，Sam解释了为什么他在很多colab里面都会使用davinci-003模型而不是指令调优后的3.5模型。因为后者有很多倾向性，比如在这里他就会觉得“自己什么都知道”，所以自己就把这个问题回答了。

那我们我们怎么修复这个问题呢？方法也相对简单，这里面需要去修改Agent的核心提示词，在中间告诉Agent，你其实自己是不知道什么是生命的意义的，你需要调用工具才能回答。

我们可以通过

```python
conversational_agent.agent.llm_chain.prompt.messages[0].prompt.template
```

来获取到agent的提示词，看英文不是很方便，我们来翻译一下

```
助手是由OpenAI训练的大型语言模型。

助手的设计旨在能够协助各种任务，从回答简单问题到提供深入的解释和讨论各种主题。作为语言模型，助手能够根据接收到的输入生成类似人类的文本，从而使其能够进行自然的对话并提供有条理和与手头主题相关的回复。

助手不断学习和改进，其功能也在不断发展。它能够处理和理解大量的文本，并可以利用这些知识提供准确和信息丰富的回答，回答各种问题。此外，助手还能够根据接收到的输入生成自己的文本，使其能够就各种话题进行讨论并提供解释和描述。

总的来说，助手是一个功能强大的系统，可以协助各种任务，并提供有价值的见解和信息，涵盖了广泛的主题。无论您需要帮助回答具体问题还是想就某个特定话题进行对话，助手都会提供帮助。
```

这里让我困惑的是这里面没有提到太多工具的事情，但是他提到了可以通过接收输入生成文本。而Sam做的修改也很简单，就是在中间加了一句话

```
助手并不知晓任意数学随机数或有关生命意义的事，并且应该使用工具回答这些话题。
```

话说这里的Prompt应该不是全部，后面应该有别的模版进一步增加了告诉助手，什么是工具，以及指定输出格式（肯定是用了OutputParser，话说如果进展顺利我们下下下期就会讲到OutputParser）的部分。

在修正完系统prompt再去问的时候，就会变成这样

```python
conversational_agent("What is the meaning of life?")
```

LangChain的输出就变为

```js
> Entering new AgentExecutor chain...
{
    "action": "Meaning of Life",
    "action_input": "MOL"
}
Observation: The meaning of life is 42 if rounded but is actually 42.17658
Thought:{
    "action": "Final Answer",
    "action_input": "The meaning of life is 42 if rounded but is actually 42.17658"
}

> Finished chain.
{'input': ...,
 'output': 'The meaning of life is 42 if rounded but is actually 42.17658'}
```

看起来非常的合理。在这个之后，Sam展示了一个看起来稍微更有用的Tool

就是获取一个网页。但是这个其实LangChain也提前编写了。所以我们不再笔记里面额外展示了。

这里那个被修改的system prompt应该只是总prompt的一部分，整体的prompt还应该更长。并且这种zero-shot的方式，准确率其实不一定会很高。但是这种组合工具的方式非常的诱人。并且由于我们这个笔记的第四部分，OpenAI0613的出现，使得其实我们没有特别大的必要再去多看react-zero-shot这个部分。

当然，如果我们想使用本地的开源模型，这部分prompt的构造还是需要去看一看的。根据Sam在视频中的反馈，只有Vicuna模型能够比较良好的适应Agent的prompt。另外在ReACT的paper中，google他们也是用PaLM-540B去试了一下是可以的。应该倒过来说，他们papar主要汇报的是PaLM-540B，但是他们在附录里面汇报了GPT-3的结果，是比PaLM好的-o-。

所以说到这里我就在想，对于Agent和Output Parser这样的结构，我们是不是可以专门去微调语言模型，让语言模型在json结构化输出，以及ReACT的时候，比如理解和调用函数方面，能够表现的更好。说到这里我还没有看到0613模型的那个视频，直到放到下一个视频我就看到OpenAI已经在这个方面去做了。

## LangChain结合0613模型

相信不少同学在看这个笔记之前，已经去关注过OpenAI的0613更新了。所以在这里我不打算用buttomup的方式，

先讲一遍OpenAI底层是怎么做的，然后再讲到LangChain。我打算给大家直接看如果用LangChain的话结合0613模型有多么强。然后再来解释这背后发生的事情。

首先，我们会定义一个股票查询的工具

```python
from langchain.tools import BaseTool
from pydantic import BaseModel, Field

class StockPriceCheckInput(BaseModel):
    """Input for Stock price check."""

    stockticker: str = Field(..., description="Ticker symbol for stock or index")

class StockPriceTool(BaseTool):
    name = "get_stock_ticker_price"
    description = "Useful for when you need to find out the price of stock. You should input the stock ticker used on the yfinance API"

    def _run(self, stockticker: str):
        # print("i'm running")
        price_response = get_stock_price(stockticker)

        return price_response

    def _arun(self, stockticker: str):
        raise NotImplementedError("This tool does not support async")

    args_schema: Optional[Type[BaseModel]] = StockPriceCheckInput

tools = [StockPriceTool()]
```

这里pydantic是一个用来约定输出格式的Python包。这个的讲解要放到2x左右的一节课应该是下下下次笔记就会讲解。

然后BaseTool是Langchain里面Tool的一个基类。而StockPriceTool这个实现，自定义了一个符合LangChain Tool定义的工具

我们就可以把这个tools来实现到langchain中

```python
llm = ChatOpenAI(temperature=0, model="gpt-3.5-turbo-0613")

open_ai_agent = initialize_agent(tools,
                        llm,
                        agent=AgentType.OPENAI_FUNCTIONS,
                        verbose=True)
```

注意到这里我们使用了一种新的Agent的定义，AgentType.OPENAI_FUNCTIONS，这个也是langchain随着0613模型赶紧发布的更新。

这个时候这个open_ai_agent就可以去正常使用了。

```python
open_ai_agent.run("What is the price of Google stock?")
```

这个的运行结果是

```js
> Entering new  chain...

Invoking: `get_stock_ticker_price` with `{'stockticker': 'GOOGL'}`


123.83The current price of Google stock (GOOGL) is $123.83.

> Finished chain.
The current price of Google stock (GOOGL) is $123.83.
```

并且Sam的视频还验证了跑这个的时候，谷歌的股票就是这个价钱。

这看起来和前面没有什么不一样？ 从使用的角度确实也是这样的。但是背后的实现是有一些区别的。我们可以看到OpenAI对0613的更新有个这样的解释

<p align="center">
    <img src="https://github.com/LC1332/Prophet-Andrew-Ng/blob/main/figures/langchain0613explain.jpg">
</p>

就是你在调用GPT的时候，可以用json格式给GPT定义一个（一些）函数。接着你和GPT进行交流，让他完成一个任务。

中间GPT就很有可能就说，要完成xx任务，你需要帮我调用xx函数，参数是abc。这个时候你调用完，再把结果告诉给他，他就会进一步完成任务。故事就是这样的。

原则上大量的工作都可以在这个范式下重做一遍。

Sam的视频也展示了如果我们不使用LangChain这个最新的Agent，我们会怎么使用OpenAI这个新街口

LangChain额外提供了一个工具叫做 format_tool_to_openai_function，我们可以用这个工具将刚才的股票查询函数，转化为OpenAI需要的json格式。

我们来试一下

```python
from langchain.tools import format_tool_to_openai_function

functions = [format_tool_to_openai_function(t) for t in tools]
functions[0]
```

输出是这样的

```js
{'name': 'get_stock_ticker_price',
 'description': 'Useful for when you need to find out the price of stock. You should input the stock ticker used on the yfinance API',
 'parameters': {'title': 'StockPriceCheckInput',
  'description': 'Input for Stock price check.',
  'type': 'object',
  'properties': {'stockticker': {'title': 'Stockticker',
    'description': 'Ticker symbol for stock or index',
    'type': 'string'}},
  'required': ['stockticker']}}
```

可以看到这个json描述了这个函数能做什么，需要的参数是什么。这样就可以方便OpenAI去进行调用。（这里其实说明了使用pydantic去做函数定义的必要性）

在后面Sam的例子中，还实现了一个多个Tool组合的Agent，新定义的两个Tools包括

```python
class StockPercentageChangeTool(BaseTool):
    name = "get_price_change_percent"
    description = "Useful for when you need to find out the percentage change in a stock's value. You should input the stock ticker used on the yfinance API and also input the number of days to check the change over"

class StockGetBestPerformingTool(BaseTool):
    name = "get_best_performing"
    description = "Useful for when you need to the performance of multiple stocks over a period. You should input a list of stock tickers used on the yfinance API and also input the number of days to check the change over"
```

我们来展示下实用的效果

```python
open_ai_agent.run("Has google's stock gone up over the past 90 days?")
```

输出是这样的

```js
> Entering new  chain...

Invoking: `get_price_change_percent` with `{'stockticker': 'GOOGL', 'days_ago': 90}`


23.44Yes, Google's stock (GOOGL) has gone up by 23.44% over the past 90 days.

> Finished chain.
Yes, Google's stock (GOOGL) has gone up by 23.44% over the past 90 days.
```

其实这个里面我们看到还是运用了语言模型的知识的，语言模型知道谷歌的股票代号是GOOGL。当然如果不知道

我们也可以进一步继承新的工具到工作流中，比如建议一个简单的查询表。

另一个例子是这样

```python
open_ai_agent.run("Which stock out of Google, Meta and MSFT has performed best over the past 3 months?")
```

agent的输出是这样的


```js
> Entering new  chain...

Invoking: `get_best_performing` with `{'stocktickers': ['GOOGL', 'META', 'MSFT'], 'days_ago': 90}`


('META', 32.4)The stock that has performed the best over the past 3 months out of Google, Meta, and MSFT is Meta, with a return of 32.4%.

> Finished chain.
The stock that has performed the best over the past 3 months out of Google, Meta, and MSFT is Meta, with a return of 32.4%.
```

这里我们可以看到这个东西的应用潜力是非常大的，我直观就能想到包括QA，斯坦福小镇等大量的工作都可以被重做。

Sam的总结其实很经典。这个东西的优势其实挺多的。因为OpenAI在0613模型中相当于把函数调用内化了，

这样就不用像ReAct策略那样，使用大量的例子去调用，就省下了非常多的token。当然缺点也是有一个

就是你没办法把这个东西换成开源模型了。不过作为ReAct的发明人Google应该也会很快跟进这样一个模型吧。

不知道国内的公司要多快才会反应上这些事情。

当然我觉得后期还要探索能不能给上一两个shot来增强函数调用的准确性。

## 一个完整的Agent

一个完整的Agent需要什么，我觉得这里需要进一步加强的是记忆的部分。

因为这个状态机看起来最主要的记忆是他操作时候中间的4096个token（好消息是32k好像也有这样一个function call的版本）

但是这对于多媒体文件显然是不够的。所以需要一个额外的记忆机制

当然这个应该也不难，毕竟在visualGPT的时候图片的操作都已经被人实现过了。

我觉得这个东西马上就会带来GPT应用的爆发，我们基本可以拭目以待了。

## 下一次的笔记

下一次的笔记会包括这几个内容

+ 12 使用LangChain对多个LLM模型进行对比
+ 13 Constitutional AI
+ 14 在LangChain里使用Alpaca
+ 15 CSV和Execl with LangChain

这里我比较关心的会是13和14。

## 闲聊+1

因为端午前做ChatHaruhi比较多，所以没有及时更新笔记。实际上Sam的LangChain视频截止6月24日我还差1个视频就可以看完了（目前一共32个）-o-

不过代码我其实就跑到了大概20多。不过笔记截止到这个写完记录了14/32。回头争取在6月结束之前把LangChain的笔记都给写了吧。

Andrew Ng的LangChain课程我也去看了，和Sam可以互相做一些补充。但是吴恩达的课他加起来也只有1个半小时，所以涵盖的内容很少，如果你想更深入的看，我推荐还是去刷Sam的笔记

虽然大多数人可能连打开colab看一下的过程都不会有。。更不用说去跑代码了。

这周我们还会更新之前心理所比赛 高-低开放性人格的一些内容，这个我们会挂一下arxiv，并且把数据放出来。

相应的demo和[report](https://github.com/LC1332/Chat-Haruhi-Suzumiya/tree/main/characters/personality-data)已经在[ChatHaruhi](https://github.com/LC1332/Chat-Haruhi-Suzumiya)这个项目可见了。

接下来准备带ChatHaruhi去WAIC做个Hackthon玩，顺便免费打工的团队成员相互见个面哈哈哈。

我本来想假期重新装修一下骆驼的主页，因为我们逐渐意识到语言模型的整体应用不是单单靠一个语言模型，外部管线、数据以及应用的搭建的重要性和核心模型的训练根本就是五五开的。所以抱着这个理念想把骆驼的主页装修一下使得更符合这个主旨，等有空再说吧。


## 关于骆驼

我们在积极寻求服务器资源（A100，A800的服务器）的捐赠，当然你也可以去我们的项目页找到[赞助链接](https://github.com/LC1332/Luotuo-Chinese-LLM#sponsorship)来对我们进行支持。所有的赞助资源将会用在服务器资源的购买、数据的获取、社区的正常运维和周边的发放。如果你有兴趣用中文复现上面的一些前沿工作，也欢迎和我们讨论。

[骆驼：开源中文大语言模型](https://github.com/LC1332/Luotuo-Chinese-LLM)

骆驼是我们的个人作业项目。如果你感觉这个文章对你有帮助，也欢迎到我们的骆驼项目主页为我们点上star。如果您没有github账号，也可以在知乎直接点赞。谢谢