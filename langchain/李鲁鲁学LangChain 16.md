# 【LangChain笔记16】BabyAGI, LangChain实现ChatPDF, 鸭鸭Go和Wiki

这周休假在家，周一先写个笔记吧 这篇笔记会包括Sam的后面4个视频

+ 15 BabyAGI: Discover the Power of Task-Driven Autonomous Agents!
+ 16 LangChain重构BabyAGI
+ 17 LangChain实现ChatPDF
+ 18 LangChain调用DuckduckGo和Wikipedia

这次的BabyAGI会稍微好玩一丢丢，相当于是一个AutoGPT的某类实现。为了简单起见我们先讲ChatPDF吧

---

## 用LangChain实现ChatPDF

这节课的额外实验放在了[这个脚本](https://colab.research.google.com/github/LC1332/Prophet-Andrew-Ng/blob/main/langchain/%E6%9D%8E%E9%B2%81%E9%B2%81%E5%AD%A6LangChain17_ChatPDF.ipynb)

其实这节课我很早很早就看了，差不多在开始骆驼QA和骆驼嵌入两个项目之后就看了这个视频。所以写这个笔记就当复习了吧

<p align="center">
    <img src="https://github.com/LC1332/Prophet-Andrew-Ng/blob/main/figures/langchainChatPDF1.png">
</p>

实现ChatPDF的原理异常简单，总之就是先把PDF文件进行切片，然后求向量放到向量数据库中


<p align="center">
    <img src="https://github.com/LC1332/Prophet-Andrew-Ng/blob/main/figures/langchainChatPDF2.png">
</p>

然后对于每个问题，检索几个片段组成上下文，进行回答。

在这里我自己试的时候选择了GPT4的tech report。因为众所周知那个tech report里面给的GPT4的实现信息非常少，我看看用ChatPDF的方式他会不会乱回答问题。

```python
!wget -q https://cdn.openai.com/papers/gpt-4.pdf

from PyPDF2 import PdfReader

doc_reader = PdfReader('/content/gpt-4.pdf')
```

用这个简单的指令来读取pdf文件。他是分页存储的，把每一页的内容合并到一起

```python
# read data from the file and put them into a variable called raw_text
raw_text = ''
for i, page in enumerate(doc_reader.pages):
    text = page.extract_text()
    if text:
        raw_text += text
```

这里总的raw_text的长度是283701。

然后Sam给他做了简单的切分

```python
from langchain.text_splitter import CharacterTextSplitter

text_splitter = CharacterTextSplitter(
    separator = "\n",
    chunk_size = 1000,
    chunk_overlap  = 200, #striding over the text
    length_function = len,
)
texts = text_splitter.split_text(raw_text)
```

这里说明每个chunk的大小大约是1000，chunk和chunk之间的重叠略小于200。这里的两个size都不是精确值，其实实际测试的时候这个函数会根据分隔符做一定的长度调整。

---

另外根据Andrew Ng新的LangChain的课程，其实我们更建议使用这个函数

```python
#需要提前安装tiktoken
text_splitter = CharacterTextSplitter.from_tiktoken_encoder(
    chunk_size=100, chunk_overlap=0
)
```

这个函数会根据token数量来切，实际上应该更准确一些。

---

我们回到程序当中，这个text_splitter把GPT4的技术报告切成了357份，其中某一份是这样

```
test takers (Table 1, Figure 4).
The model’s capabilities on exams appear to stem primarily from the pre-training process and are not
signiﬁcantly affected by RLHF. On multiple choice questions, both the base GPT-4 model and the
RLHF model perform equally well on average across the exams we tested (see Appendix B).
We also evaluated the pre-trained base GPT-4 model on traditional benchmarks designed for evaluating
language models. For each benchmark we report, we ran contamination checks for test data appearing
in the training set (see Appendix D for full details on per-benchmark contamination).5We used
few-shot prompting [1] for all benchmarks when evaluating GPT-4.6
GPT-4 considerably outperforms existing language models, as well as previously state-of-the-art
(SOTA) systems which often have benchmark-speciﬁc crafting or additional training protocols
(Table 2).
5During our contamination check we discovered that portions of BIG-bench [48] were inadvertently mixed
```

或者编号10的一份是这样

```
normalized so that GPT-4 is 1.
Observed
Prediction
gpt-4
1µ 10µ 100µ 0.001 0.01 0.1 1
Compute012345– Mean Log Pass RateCapability prediction on 23 coding problems
Figure 2. Performance of GPT-4 and smaller models. The metric is mean log pass rate on a subset of
the HumanEval dataset. A power law ﬁt to the smaller models (excluding GPT-4) is shown as the dotted
line; this ﬁt accurately predicts GPT-4’s performance. The x-axis is training compute normalized so that
GPT-4 is 1.
3wherekand are positive constants, and Pis a subset of problems in the dataset. We hypothesize
that this relationship holds for all problems in this dataset. In practice, very low pass rates are difﬁcult
or impossible to estimate, so we restrict to problems Pand models Msuch that given some large
sample budget, every problem is solved at least once by every model.
We registered predictions for GPT-4’s performance on HumanEval before training completed, using
```

然后就是求embedding，建立向量数据库

```python
from langchain.embeddings.openai import OpenAIEmbeddings
from langchain.vectorstores import FAISS

embeddings = OpenAIEmbeddings(disallowed_special=())
docsearch = FAISS.from_texts(texts, embeddings)
```

这里其实是把OpenAI的函数打到一个管线里面，OpenAI这个也可以替换成本地

话说Sam后面2x视频里面有非常大量怎么把这些模型换成本地模型的，所以我们回头再说换到本地这个事儿吧。

群里有人反应直接用HuggingFaceInferenceEmbeddings + LuotuoBert是可以使用的

这个可以我们在后面的课程里面去试一下

这里我们尝试去运行

```python
query = "GPT-4在哪些任务上显著超过了GPT-3?"
docs = docsearch.similarity_search(query)
```

这里sam的notebook先检查了搜索出来的文档

```
in such scenarios, GPT-4 was evaluated on a variety of exams originally designed for humans. In\nthese evaluations it performs quite well and often outscores the vast majority of human test takers.\nFor example, on a simulated bar exam, GPT-4 achieves a score that falls in the top 10% of test takers.\nThis contrasts with GPT-3.5, which scores in the bottom 10%.\nOn a suite of traditional NLP benchmarks, GPT-4 outperforms both previous large language models\nand most state-of-the-art systems (which often have benchmark-speciﬁc training or hand-engineering).\nOn the MMLU benchmark [ 35,36], an English-language suite of multiple-choice questions covering\n57 subjects, GPT-4 not only outperforms existing models by a considerable margin in English, but\nalso demonstrates strong performance in other languages. On translated variants of MMLU, GPT-4\nsurpasses the English-language state-of-the-art in 24 of 26 languages considered. We discuss these
```

看起来基本是对的。那么我们就可以进一步使用LangChain里面的QA Chain，把这些docs合并起来 让GPT做问答

```python
from langchain.chains.question_answering import load_qa_chain
from langchain.chat_models import ChatOpenAI

llm = ChatOpenAI(model_name='gpt-3.5-turbo')

chain = load_qa_chain(llm,
                      chain_type="stuff") # we are going to stuff all the docs in at once
```

话说我时候才理解到他这里面总是说stuff就是全部粘起来的意思。。之前summarizeChain里面也有个stuff让我困惑了一下。。。

```python
query = "GPT-4在哪些任务上显著超过了GPT-3?"
docs = docsearch.similarity_search(query)
chain.run(input_documents=docs, question=query)
```

输出是

```
根据上下文提供的信息，GPT-4在多项任务上显著超过了GPT-3。具体来说，GPT-4在人类设计的各种考试中表现优异，
在大多数考试中超过了GPT-3。例如，在模拟的律师资格考试中，GPT-4的得分位居前10%。此外，在传统的自然语言
处理基准测试中，GPT-4不仅超过了之前的大型语言模型，还超过了大多数最先进的系统。在MMLU基准测试中，GPT-4
在英语领域的表现远远超过现有模型，并在其他语言中也展现出强大的性能。总之，GPT-4在学术和专业考试以及自然
语言处理任务中的表现优于GPT-3。
```

看起来还是不错的，接着我们拿两个GPT4报告中没有提到的问题来问这个系统

```python
query = "训练GPT-4使用了多少计算资源?"
docs = docsearch.similarity_search(query)
chain.run(input_documents=docs, question=query)
```

输出是

```
根据提供的上下文信息，无法确定GPT-4训练时使用了多少计算资源。
```

靠谱，再问一个文章没提到的问题

```python
query = "GPT-4使用了多少训练数据?"
docs = docsearch.similarity_search(query)
chain.run(input_documents=docs, question=query)
```

输出是

```
GPT-4的技术报告中没有提到具体的训练数据量。
```

问个有模糊信息的呀，因为GPT4的技术报告里面提到用了transformer-like的结构

```python
query = "GPT-4使用了什么模型架构?"
docs = docsearch.similarity_search(query)
chain.run(input_documents=docs, question=query)
```

输出是

```
GPT-4使用了Transformer-based模型架构。
```

看起来还可以，让我们问个泛化一点的问题吧

```python
query = "GPT-4相比于过往的语言模型，有哪些新的特性和提升？"
docs = docsearch.similarity_search(query)
chain.run(input_documents=docs, question=query)
```

输出是这样的

```
根据文本中的描述，GPT-4相较于之前的语言模型有如下的新特性和提升：

1. 在人类设计的各种考试中，GPT-4的表现非常出色，通常能在绝大多数考生中获得前10%的成绩，而之前的GPT-3.5则在最后10%的考生中。

2. 在传统的自然语言处理基准测试中，GPT-4不仅超越了之前的大型语言模型，还超过了大多数最先进的系统。这些系统通常需要进行基准测试特定的训练或手工工程。

3. 在英语的MMLU基准测试中，GPT-4不仅在英语方面比现有模型表现出色，而且在其他语言中也表现出很强的性能。在翻译版本的MMLU中，GPT-4在26种考虑的语言中有24种超越了英语的最先进水平。

4. GPT-4在用户意图追踪方面明显提高。在ChatGPT和OpenAI API的数据集中，GPT-4生成的回答在70.2%的提示中优于GPT-3.5生成的回答。

5. GPT-4接受由图像和文本组成的提示，能够处理图像和文本混合的输入。在各种领域中，包括包含文本和照片、图表或屏幕截图的文档，GPT-4表现出与仅文本输入相似的能力。

需要注意的是，以上是根据文本中提供的信息推测得出的特性和提升，具体细节可能需要参考原始文献以获取更确切的信息。
```

感觉还挺不错的。另外QA除了staff模式，还有一个map-reduce模式，是逐个reference回答再总结，感觉实用意义一般就不展示了。

Sam到后面有大量的怎么把这里的模型换成本地、或者换成其他向量库的视频。我们应该会在后面的笔记中看到。

另外有一点是这个prompt肯定做了一定的grading，你看他都是严格根据上下文进行描述，不会乱说话。上下文没有的时候，就会说这个信息没有。

---

## BabyAGI: 一个简易的任务驱动智能体

这个和CAMEL或者autoGPT一样都是对GPT的一个加强，就是他们会自己列任务、给任务排优先级然后执行。

我注意到CAMEL没有去用向量数据库，但是BabyAGI用了，感觉最显著的区别就在这里。

repo在

https://github.com/yoheinakajima/babyagi

这已经16k星了？？？


<p align="center">
    <img src="https://github.com/LC1332/Prophet-Andrew-Ng/blob/main/figures/langchainBabyAGI.png">
</p>

他的想法其实挺简单，就是每次从队列里面抽取一个没完成的任务。

然后看看这个任务是不是可以细化。然后他优先级排序最需要完成的任务

去解决，还有一个评估的prompt来评估任务是不是结束。

我们可以先看看使用的效果，这里我给出了一个我觉得不是很简单的任务

```
Calculate the GDP growth rate of China from 2018 to 2022 and summarize the reasons.
计算中国从2018年到2022年的GDP增长率并总结原因。
```

这里我们看BabyAGI对于这个主任务的分解

```
1. 收集数据：
   - 收集2018年至2022年中国的GDP数据。
   - 获取每年的季度或年度GDP数据。
   - 确保数据来源可靠，来自世界银行、国际货币基金组织（IMF）或中国国家统计局等权威机构。

2. 计算GDP增长率：
   - 确定计算GDP增长率的公式：[(当前年份的GDP - 前一年的GDP) / 前一年的GDP] * 100。
   - 应用公式计算2018年至2022年每年的GDP增长率。
   - 计算季度和年度增长率以进行全面分析。

3. 分析影响GDP增长的因素：
   - 识别影响指定时期中国GDP增长的关键因素。
   - 考虑政府政策、贸易关系、国内消费、投资、技术进步和全球经济状况等因素。
   - 进行研究并审阅经济报告，了解这些因素对中国GDP增长的影响。

4. 总结GDP增长的原因：
   - 分析计算得到的GDP增长率，并与确定的因素进行比较。
   - 确定观察到的增长率背后的主要原因。
   - 清晰简明地总结这些原因，重点突出2018年至2022年中国GDP增长的最重要因素。
   - 提供支持证据或数据以加强总结的可信度。

5. 准备报告或演示文稿：
   - 将计算得到的GDP增长率、因素分析和总结的原因编写成全面的报告或演示文稿。
   - 在报告/演示文稿中设置引言、方法、研究结果和结论部分。
   - 包括相关的图表、图形或视觉辅助工具以增强理解。
   - 确保报告/演示文稿组织良好，易于理解，并具有视觉吸引力。

6. 审核和验证：
   - 审查报告/演示文稿的准确性、连贯性和逻辑流程。
   - 通过与其他可靠来源的交叉参考来验证研究结果和原因。
   - 寻求同事或领域专家的反馈，以确保分析全面而可靠。

7. 呈现和沟通：
   - 向相关利益相关者（如客户、同事或上级）展示研究结果和原因。
   - 清晰传达计算得到的GDP增长率及其背后的原因。
   - 在演示过程中回答提出的任何问题或关注点，如有需要，提供额外的信息或澄清。

8. 更新和监测：
   - 持续监测2022年后中国的GDP增长和相关因素，以跟上最新动态。
   - 随着新数据的发布，定期更新对GDP增长的分析和原因。
   - 了解可能影响中国未来GDP增长的经济趋势、政策变化和全球事件。
```

这里5后面的步骤让我觉得很搞笑

并且这里他还输出了一个思考：

```
根据政府政策、贸易关系、国内消费、投资、技术进步和全球经济状况等各种因素的影响，预计中国从2018年到2022年的国内生产总值 (GDP) 增长率将在6.5%至7.5%之间。
```

然后他根据这个结论，进一步创造任务

```
你是一个任务创建的人工智能，使用执行代理的结果创建新的任务，目标如下: 计算中国2018年至2022年的国内生产总值 (GDP) 增长率，并总结原因。最后完成的任务结果为: 中国2018年至2022年的国内生产总值 (GDP) 增长率预计在6.5%至7.5%之间，具体取决于政府政策、贸易关系、国内消费、投资、技术进步和全球经济状况等各种因素的影响。这个结果是基于以下任务描述的：制作一个待办事项清单。以下是未完成的任务：根据结果，创建新的任务，由AI系统完成，并且不与未完成的任务重叠。以数组形式返回任务。
```



然后他这里还有个任务优先级排序的Chain，输入是这样的

```
Entering new TaskPrioritizationChain chain...
Prompt after formatting:
你是一款任务优先级排序的人工智能，任务是清理以下任务的格式并重新确定优先级：['任务：'，'1. 分析政府政策对中国2018年至2022年国内生产总值 (GDP) 增长率的影响。'，'2. 分析贸易关系对中国2018年至2022年国内生产总值 (GDP) 增长率的影响。'，'3. 分析国内消费对中国2018年至2022年国内生产总值 (GDP) 增长率的影响。'，'4. 分析投资对中国2018年至2022年国内生产总值 (GDP) 增长率的影响。'，'5. 分析技术进步对中国2018年至2022年国内生产总值 (GDP) 增长率的影响。'，'6. 分析全球经济状况对中国2018年至2022年国内生产总值 (GDP) 增长率的影响。'，'7. 总结分析结果并提供改善2018年至2022年中国国内生产总值 (GDP) 增长率的建议。']。请考虑你团队的最终目标：计算中国2018年至2022年国内生产总值 (GDP) 增长率并总结原因。请不要删除任何任务。以数字列表的形式返回结果，如：#. 第一个任务 #. 第二个任务。请以数字2开始任务列表。
```

排序完的任务是这样的

```
2: 分析政府政策对中国2018年至2022年的国内生产总值 (GDP) 增长率的影响。
3: 分析贸易关系对中国2018年至2022年的国内生产总值 (GDP) 增长率的影响。
4: 分析国内消费对中国2018年至2022年的国内生产总值 (GDP) 增长率的影响。
5: 分析投资对中国2018年至2022年的国内生产总值 (GDP) 增长率的影响。
6: 分析技术进步对中国2018年至2022年的国内生产总值 (GDP) 增长率的影响。
7: 分析全球经济状况对中国2018年至2022年的国内生产总值 (GDP) 增长率的影响。
8: 总结分析结果，并提出改善中国2018年至2022年的国内生产总值 (GDP) 增长率的建议。
9: 计算中国2018年至2022年的国内生产总值 (GDP) 增长率，并总结原因。
```

到这里还有一些套话，然后他开始执行第二个任务

```
思考：我需要了解中国2018年至2022年的国内生产总值 (GDP) 增长率，并总结原因。
行动：搜索
行动输入：中国2018年至2022年的国内生产总值 (GDP) 增长率
观察：https://www.macrotrends.net/countries/CHN/china/gdp-growth-rate
思考：我需要总结中国2018年至2022年的国内生产总值 (GDP) 增长率的原因
行动：搜索
行动输入：中国2018年至2022年的国内生产总值 (GDP) 增长率的原因
观察：在中国2022年底新冠疫情爆发后迅速重新开放后，预计GDP增长率将从2023年的3.0%反弹至5.1% 。
思考：我现在知道最终答案了
最终答案：中国2018年至2022年的国内生产总值 (GDP) 增长率预计为3.0%。这一增长率的原因包括在2022年底新冠疫情爆发后经济迅速重新开放，以及政府政策，如增加对基础设施和技术的投资。
```

这里我其实有一点震惊。。。你去看这个页面，他其实是没有重新开放的信息的

然后他这里重新评估了顺序之后，又觉得要去看贸易的影响

```
Thought: I need to search for data on the GDP growth rate of China from 2018 to 2022.
Action: Search
Action Input: GDP growth rate of China from 2018 to 2022
Observation: https://www.macrotrends.net/countries/CHN/china/gdp-growth-rate
Thought: I need to analyze the impact of trade relations on the GDP growth rate of China from 2018 to 2022.
Action: Search
Action Input: Impact of trade relations on the GDP growth rate of China from 2018 to 2022
Observation: Following China's swift reopening after the COVID-19 outbreaks in late 2022, GDP growth is expected to rebound to 5.1 percent in 2023, from 3 percent in 2022.
Thought: I now know the final answer
Final Answer: China's GDP growth rate is expected to rebound to 5.1 percent in 2023, from 3 percent in 2022, due to the swift reopening after the COVID-19 outbreaks in late 2022.
```

他还知道因为开放了之后，今年的GPT增长预计会从去年的3调整到5.1？

接着重新排序任务之后，他居然能够意识到国内消费是下一个重要的议题。。

```
Thought: I need to search for information about the impact of domestic consumption on the GDP growth rate of China from 2018 to 2022.
Action: Search
Action Input: Impact of domestic consumption on the GDP growth rate of China from 2018 to 2022
Observation: Following China's swift reopening after the COVID-19 outbreaks in late 2022, GDP growth is expected to rebound to 5.1 percent in 2023, from 3 percent in 2022.
Thought: I need to analyze the impact of government policies on the GDP growth rate of China from 2018 to 2022.
Action: Search
Action Input: Impact of government policies on the GDP growth rate of China from 2018 to 2022
Observation: Since China began to open up and reform its economy in 1978, GDP growth has averaged over 9 percent a year, and more than 800 million people have lifted ...
Thought: I now know the final answer
Final Answer: The GDP growth rate of China from 2018 to 2022 is expected to be around 5.1 percent, due to the impact of domestic consumption and government policies.
```

接着他又重新评估了任务

```
Final Answer: Based on the analysis of trade relations, government policies, and domestic consumption, China's GDP growth rate is expected to rebound to 5.1 percent in 2023, from 3 percent in 2022. The main drivers of this growth are expected to be increased domestic consumption, improved trade relations, and supportive government policies.
```

---

这里也可以贴一下Sam原来的例子，他的任务要和谐和简单一些

```
Find the cheapest price and site to buy a Yubikey 5c online and give me the URL.
```

这个简单例子的todo list就会比较简短

```
1. Research online retailers that sell Yubikey 5c (e.g. Amazon, Best Buy, etc.).

2. Compare prices of Yubikey 5c across different retailers.

3. Check for any discounts or promotions available for Yubikey 5c.

4. Read customer reviews for each retailer to determine the best site to buy from.

5. Once you have identified the cheapest price and site, copy the URL and provide it to the customer.
```

在这个例子中最后Agent确实比价输出了一些网站

```
The cheapest price and site to buy a Yubikey 5c online is YubiKey 5 Series for $50 from YubiKey. The URL is https://www.yubico.com/products/yubikey-hardware/
```

具体的代码我放在

[这个链接](https://colab.research.google.com/github/LC1332/Prophet-Andrew-Ng/blob/main/langchain/%E6%9D%8E%E9%B2%81%E9%B2%81%E5%AD%A6LangChain16_BabyAGI.ipynb)

这个其实是Sam加入了tool的，原来的babyAGI代码应该不带这个搜索的功能。

---

+ 可以看到这个BabyAGI对于一些任务，确实能够启发出一些合理的子任务，并且结合搜索工具，可以给出一些结论

+ 对于比较简单的任务，比如比价，BabyAGI是有可能成功迭代完成的

+ 对于更复杂的任务，其实任务队列很容易越来越长，无法完结

+ 我觉得这里任务的分解和完结可以好好做一下文章。现在靠LLM来分解任务和确认是否完结不是特别靠谱，应该引入更多的约束

---

我觉得比较有意思的是这里有一页

https://github.com/yoheinakajima/babyagi/blob/main/docs/inspired-projects.md

讲的是projects inspired by BabyAGI

<p align="center">
    <img src="https://github.com/LC1332/Prophet-Andrew-Ng/blob/main/figures/projectsInspiredByBaby.png">
</p>

这个有空可以回头再捞一下。

---

然后18课的内容实际上之前Agents的笔记都已经覆盖了

可以简单说一下

18课其实就是介绍了一些LangChain里面现成的tools的使用。比如wiki


```python
from langchain.utilities import WikipediaAPIWrapper

wikipedia = WikipediaAPIWrapper()
wikipedia.run('Langchain')
```

```
Page: LangChain
Summary: LangChain is a framework designed to simplify the creation of applications using large language models (LLMs). As a language model integration framework, LangChain's use-cases largely overlap with those of language models in general, including document analysis and summarization, chatbots, and code analysis.

Page: Sentence embedding
Summary: In natural language processing, a sentence embedding refers to a numeric representation of a sentence in the form of a vectors of real numbers which encodes meaningful semantic information.State of the art embeddings are based on the learned hidden layer representation of dedicated sentence transformer models. BERT pioneered an approach involving the use of a dedicated [CLS] token preprended to the beginning of each sentence inputted into the model; the final hidden state vector of this token encodes information about the sentence and can be fine-tuned for use in sentence classification tasks. In practice however, BERT's sentence embedding with the [CLS] token achieves poor performance, often worse than simply averaging non-contextual word embeddings. SBERT later achieved superior sentence embedding performance by fine tuning BERT's [CLS] token embeddings through the usage of a siamese neural network architecture on the SNLI dataset. 
Other approaches are loosely based on the idea of distributional semantics applied to sentences. Skip-Thought trains an encoder-decoder structure for the task of neighboring sentences predictions. Though this has been shown to achieve worse performance than approaches such as InferSent or SBERT. 
An alternative direction is to aggregate word embeddings, such those returned by Word2vec, into sentence embeddings. The most straightforward approach is to simply compute the average of word vectors, known as continuous bag-of-words (CBOW). However, more elaborate solutions based on word vector quantization have also been proposed. One such approach is the vector of locally aggregated word embeddings (VLAWE), which demonstrated performance improvements in downstream text classification tasks.
```

比如REPL

```python
from langchain.utilities import PythonREPL

python_repl = PythonREPL()
python_repl.run("print(17*2)")
```

输出是34哈

比如DuckduckGo

```python
from langchain.tools import DuckDuckGoSearchRun
search = DuckDuckGoSearchRun()
search.run("Tesla stock price?")
```

输出是

```
Get the latest Tesla Inc (TSLA) real-time quote, historical performance, charts, and other financial information to help you make more informed trading and investment decisions. MarketWatch IBD Tesla Inc. TSLA (U.S.: Nasdaq) AT CLOSE 4:00 PM EDT 07/10/23 $269.61USD -4.82 -1.76% AFTER HOURS 7:59 PM EDT 07/10/23 $268.40 -1.21 -0.45% AFTER HOURS Vol 1,175,976 Volume... Discover historical prices for TSLA stock on Yahoo Finance. View daily, weekly or monthly format back to when Tesla, Inc. stock was issued. Tesla Inc. historical stock charts and prices, analyst ratings, financials, and today's real-time TSLA stock price. Overview Key Ratios Trading Information Last Close 271.99 Sector Consumer Cyclical Industry Auto Manufacturers Investment Style Large Growth Day Range 271.46 - 276.52 Year Range 101.81 - 314.67...
```

这里我要夸一下duckduckGo，虽然搜得不一定对，但是他完全不要钱

然后google的Serp API是要钱的，bing的似乎也要钱。而且这两个都不便宜

这导致我们最近想试验一些搜索agent相关的工作碰到了经济阻碍>_<

如果有哪个大佬知道可以通过页面代理的方式去调用，有好的项目可以使用，可以介绍给我一下


这三个tools的介绍其实是为了后面agents做铺垫

不过agents的笔记我们已经提前写啦哈哈哈


今天的笔记就到这里。

笔记实际可以在[骆驼先知]的页面(https://github.com/LC1332/Prophet-Andrew-Ng/blob/main/langchain/%E6%9D%8E%E9%B2%81%E9%B2%81%E5%AD%A6LangChain%2016.md)直接看到

知乎的版本是从md顺便自动转化的。langchain的笔记看起来还有4-5篇，写完之后可以再出个目录总结篇。目测7月份搞不完。。

## 关于骆驼

我们在积极寻求服务器资源（A100，A800的服务器）的捐赠，当然你也可以去我们的项目页找到[赞助链接](https://github.com/LC1332/Luotuo-Chinese-LLM#sponsorship)来对我们进行支持。所有的赞助资源将会用在服务器资源的购买、数据的获取、社区的正常运维和周边的发放。如果你有兴趣用中文复现上面的一些前沿工作，也欢迎和我们讨论。

[骆驼：开源中文大语言模型](https://github.com/LC1332/Luotuo-Chinese-LLM)

骆驼是我们的个人作业项目。如果你感觉这个文章对你有帮助，也欢迎到我们的骆驼项目主页为我们点上star。如果您没有github账号，也可以在知乎直接点赞。谢谢




