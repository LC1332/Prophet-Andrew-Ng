# 【LangChain笔记11】如何打造属于自己的GPT4，利用LangChain和GPT最新模型构建工具组合助手

我原来想的标题叫做**跨时代的Agent机制，Reason+Act实现原理, 支持函数调用的OpenAI0613模型**

今天这是一篇比较重要的笔记，会涉及LangChain中的Agent机制，

以及对应的这个Agent的实现原理，即Reasoning+Act的原理

去讨论OpenAI的0613-turbo模型如何官方实现了这个Agent的机制

---

对于这方面接触不深的同学来说，你完全可以把今天的内容理解成

“如何打造属于自己的GPT4，利用LangChain的Agent和GPT最新模型构建工具组合助手”

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

