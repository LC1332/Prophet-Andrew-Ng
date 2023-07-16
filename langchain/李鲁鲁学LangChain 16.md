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

---


