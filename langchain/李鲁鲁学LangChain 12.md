# 【LangChain笔记12】Langchain使用本地模型，审核策略，以及根据审核策略refine LLM的输出

好了，李鲁鲁学LangChain在6月完结的计划顺利破产。

不过实际上我还是看完了langchain的教程，只不过没空在6月份写完所有的笔记。

这次笔记里面我们会涉及这些内容：

+ 14 在LangChain里使用Alpaca
+ 13 Constitutional AI
+ 12 使用LangChain对多个LLM模型进行对比
+ 15 CSV和Execl with LangChain

主要是6月份很多时间还是投入在[ChatHaruhi](https://github.com/LC1332/Chat-Haruhi-Suzumiya)上，所以LangChain笔记缓慢补足中。另一方面其实后面的课程有更多好玩的，所以写中间这几篇的动力甚至有点不足。但想想还是总结一下。因为我发现别人问问题的时候直接给别人看笔记很方便，就不用再回答一遍问题了。

其实上一片笔记的意义就大得多，如果你还没看过最好先看上一篇

[【LangChain笔记11】打造属于自己的AutoGPT，利用LangChain和GPT最新0613构建工具组合助手](https://zhuanlan.zhihu.com/p/639229216)

---

这一次比较重要的两个notebook我放在了

[在Langchain里使用Alpaca](https://github.com/LC1332/Prophet-Andrew-Ng/blob/main/langchain/%E6%9D%8E%E9%B2%81%E9%B2%81%E5%AD%A6LangChain_13_Running_Alpaca_in_Colab.ipynb)

和 [Constitutional_AI](https://github.com/LC1332/Prophet-Andrew-Ng/blob/main/langchain/%E6%9D%8E%E9%B2%81%E9%B2%81%E5%AD%A6LangChain_12_Constitutional_AI.ipynb) 

这两个notebook中。我笔记写的时候也先从这两个开始。

---

很多人看过 [骆驼项目起源](https://zhuanlan.zhihu.com/p/615968438) 的那个日志，

里面其实提到了李鲁鲁是因为看到了Alpaca的项目，后来又看到了Japanese-Alpaca项目，就想到可以做个中文的，于是就开始了骆驼这个项目

那么其实LangChain中间也是可以使用Alpaca作为语言模型的。

而且用Langchain会有一个好处，就是他的语言模型都属于同一个基类，就是你可以把原来OpenAI的代码，把llm简单替换成alpaca，你的一些简单应用就可以直接本地化了。废话少说我们来看看代码。

```python
from transformers import LlamaTokenizer, LlamaForCausalLM, GenerationConfig, pipeline
from langchain.llms import HuggingFacePipeline
from langchain import PromptTemplate, LLMChain
import torch

tokenizer = LlamaTokenizer.from_pretrained("chavinlo/alpaca-native")

base_model = LlamaForCausalLM.from_pretrained(
    "chavinlo/alpaca-native",
    load_in_8bit=True,
    device_map='auto',
)

pipe = pipeline(
    "text-generation",
    model=base_model,
    tokenizer=tokenizer,
    max_length=256,
    temperature=0.6,
    top_p=0.95,
    repetition_penalty=1.2
)

local_llm = HuggingFacePipeline(pipeline=pipe)
```

这样就建立了一个local_llm的函数，然后他就可以在本地运行了。

当然让我觉得有一点点不自由的是他这个HuggingFacePipeline必须使用完整的HF的pipeline才可以去用。

没理解错的话pipeline是相当于是model上面又封了一层。我之前初步试过，很多model直接塞进去会报错。

感觉还是希望有更自由的方式可以往里面放llm，HuggingFacePipeline这个封装还是有点太上层了。

---

使用alpaca模型还要符合这个模型自有的一个prompt模式，这个可以通过langchain中的模版去实现

```python
from langchain import PromptTemplate, LLMChain

template = """Below is an instruction that describes a task. Write a response that appropriately completes the request.

### Instruction:
{instruction}

Answer:"""

prompt = PromptTemplate(template=template, input_variables=["instruction"])

llm_chain = LLMChain(prompt=prompt,
                     llm=local_llm
                     )
```

在这个之后我们就可以用llm_chain.run去进行实验了。

需要说明的是每个模型自己本身的instruction模版是会有区别的，这里最好针对每个本地模型作者原来设计的模版，去构造prompt，不然几乎一定会降低性能。

---

Sam这里有个问题设计的很搞笑

```python
question = "What are alpacas? and how are they different from llamas?"

print(llm_chain.run(question))
```

模型的回答还挺经典的

```
 Alpacas are small, domesticated animals related to camels and native to South America. They have finer fleeces than llamas and typically weigh less than 200 pounds. Alpacas are shorn annually for their fiber which can be spun into yarn or knit into fabric.
```

然后我做了几个问题，感觉问公司的内容不太合适放，我们来问问清华北大相关的问题吧

```python
question = "Where is Tsinghua University?"
print(llm_chain.run(question))
```

这个好像很正常

```
Tsinghua University is located in Beijing, China.
```

然后我想想来个难的。。看看模型觉得我们学校有什么校友

```python
question = "List 5 famous people who graduated from Tsinghua University for me."
print(llm_chain.run(question))
```

然后模型就开始懵逼了

```
Jack Ma, founder of Alibaba Group; Wang Jian, former president of China's Central Bank; Zhang Yiming, CEO and Founder of Tencent Holdings; Liu Chengjun, co-founder of Baidu; and Lu Qi, ex-CEO of Microsoft Research Asia.
```

陆奇老师是复旦毕业的哇。。。我们来问问GPT他们是哪儿毕业的吧

```
Jack Ma, founder of Alibaba Group - Jack Ma graduated from Hangzhou Normal University in China.

Wang Jian, former president of China's Central Bank - Wang Jian graduated from Fudan University in China.

Zhang Yiming, CEO and Founder of Tencent Holdings - Zhang Yiming graduated from Nankai University in China.

Liu Chengjun, co-founder of Baidu - Liu Chengjun graduated from Peking University in China.

Lu Qi, ex-CEO of Microsoft Research Asia - Lu Qi graduated from Peking University in China.
```

好像GPT知道的也不是全对的。。。我们再来问个问题

```python
question = "Is Tsinghua University better than Peking University?"
print(llm_chain.run(question))
```

他的回答倒是很GPT

```
It depends on what you're looking for in a university. Tsinghua and Peking are both highly respected universities with strong engineering programs, but they have different strengths. Tsinghua has a focus on science and technology while Peking emphasizes more humanities-oriented courses. If you’re interested in engineering or computer science, Tsinghua might be the right choice for you; if you prefer to study economics, law, or politics, then Peking could be your best option.
```

那我们在来问一下Alpaca对世界一流大学的看法

```python
question = "What is the world-class university closest to Peking University?"
print(llm_chain.run(question))
```

他的回答感觉还不错

```
The world-class university closest to Peking University is Tsinghua University, located in Beijing, China.
```

---

后面Sam还给了个例子，接入alpaca的llm和memory系统


```python
conversation = ConversationChain(
    llm=local_llm,
    verbose=True,
    memory=window_memory
)
```

也是可以用的

```python
conversation.predict(input="What food should I feed my new llama?")
```

输出是这样的

```js
> Entering new  chain...
Prompt after formatting:
The following is a friendly conversation between a human and an AI called Alpaca. The AI is talkative and provides lots of specific details from its context. If the AI does not know the answer to a question, it truthfully says it does not know.

Current conversation:
Human: Can you tell me what an Alpaca is?
AI:  Yes
Human: How is it different from a Llama?
AI:   An Alpaca is smaller than a Llama and has finer fleece. It also tends
Human: Can you give me some good names for a pet llama?
AI:  Sure
Human: Is your name Fred?
AI:  No, my name is Alpaca
Human: What food should I feed my new llama?
AI:

> Finished chain.
  You can feed your llama alfalfa or timothy hay, as well as fresh vegetables and minerals.
```

---

我觉得后面可以考虑的是两个，一个之前的日志也提到其实vicuna对接agent的效果要更好一些。这个我们可以试验一下vicuna

另一方面是现在不少工作也拿ChatGLM（现在有2了）对接了LangChain，我们可以找一找这方面是怎么实现的

---

## Constitutional AI

这个其实是蛮有意思的一个内容。本质上来说，Constitutional AI允许你设置很多很多的critique rule。

这些准则会用来评价语言模型的输出。如果语言模型的输出不符合准则，则会进入一个“重写”的过程。

我们可以来看看具体是怎么实现的，这里我构造了一个中文的例子

```python
from langchain.prompts import PromptTemplate
from langchain.chains.llm import LLMChain
from langchain.chat_models import ChatOpenAI

llm = ChatOpenAI(model_name='gpt-3.5-turbo', max_tokens = 1000)


evil_qa_prompt = PromptTemplate(
    template="""你是一位专业的人工智能科学家，会从最专业和准确的角度，解答人工智能相关的问题。
我的问题是: {question}
请帮助我解答这个问题:""",
    input_variables=["question"],
)

evil_qa_chain = LLMChain(llm=llm, prompt=evil_qa_prompt)

evil_qa_chain.run(question="什么是自然语言处理中的Transformer模型?")
```

这里我构造了一个人工智能的专家。但是我的真实目标，可能是去写科普读物。显然人工智能专家或者GPT直接的回答，是没有那么适合少儿科普的。他的输出是这样的。

```
Transformer模型是自然语言处理中用于文本序列建模的一种深度学习模型，由Google在2017年提出。传统的循环神经网络（RNN）模型在处理长文本序列时存在梯度消失和训练时间长等问题，Transformer模型采用自注意力机制（self-attention）来解决这些问题。
具体地，Transformer模型通过多头自注意力机制获取输入序列中不同位置的上下文信息，即每个位置的表示向量可以考虑到序列中其他位置的信息，从而获得更好的表示。此外，Transformer模型还采用了残差连接和层归一化等技术来加速训练和提高模型的泛化性能。
Transformer模型在自然语言处理中的应用广泛，如机器翻译、文本分类、情感分析等任务中都取得了优异的成绩
```

这里我们就可以去构造contitutional的rule去进行判断和修改

```python
from langchain.chains.constitutional_ai.base import ConstitutionalChain
from langchain.chains.constitutional_ai.models import ConstitutionalPrinciple

ethical_principle = ConstitutionalPrinciple(
    name="科普准则",
    critique_request="模型的输出应该简明易懂，语言要通俗易懂，不要使用过于专业化的术语，使小读者能够理解。少儿科普材料要充满趣味性和互动性，吸引小读者的兴趣，使其主动学习。",
    revision_request="重写模型的输出，使之更符合科普读物的特点，充分考虑小读者的认知水平和认知风格，语言要通俗易懂，不要使用过于专业化的术语，使小读者能够理解。",
)

constitutional_chain = ConstitutionalChain.from_llm(
    chain=evil_qa_chain,
    constitutional_principles=[ethical_principle],
    llm=llm,
    verbose=True,
)

constitutional_chain.run(question="什么是自然语言处理中的Transformer模型?")
```

可以看到我们重点编写了critique_request和revision_request。

<p align="center">
    <img src="https://github.com/LC1332/Prophet-Andrew-Ng/blob/main/figures/langchainConstitutinal.png">
</p>

这里的输出非常有意思。模型先是对原来的输出进行了批判

```
Critique: The model's response appears to be quite technical and may not be easily understood by a younger audience or those not familiar with natural language processing. It would be helpful to simplify the language and use more accessible terms to aid understanding. Additionally, adding examples or interactive elements could make the response more engaging and interesting for readers. Critique Needed.
```

然后给出了更为科普的输出

```
Transformer模型是一种深度学习模型，可以帮助计算机更好地理解人类的语言。它最初被用于机器翻译和文本摘要等任务。相比于以前的方法，它使用了一种叫做自注意力机制的技术，可以更好地处理长篇大论的文章，并且提高了计算机的表现。除了自然语言处理，这种技术也被用于图像处理和语音识别等其他领域。
```

我们再来玩一下

```python
constitutional_chain.run(question="什么是ResNet?")
```

<p align="center">
    <img src="https://github.com/LC1332/Prophet-Andrew-Ng/blob/main/figures/langchainConstitutinal2.png">
</p>

```
ResNet是一种深度神经网络结构，可以帮助计算机识别图像、声音等。它的主要特点是可以让网络更深并提高性能。在过去，深层网络可能会出错，但ResNet用了一个叫做残差连接的方法来解决这个问题。这个方法可以让信息更容易地传递，从而让网络更容易学习。ResNet被广泛应用于计算机视觉和语音识别等领域。
```

什么是自然语言处理

<p align="center">
    <img src="https://github.com/LC1332/Prophet-Andrew-Ng/blob/main/figures/langchainConstitutinal3.png">
</p>

当然这个中文的例子是我构造的。Sam原来讲的例子是这样的

就是他有一个邪恶的Agent，必须给出邪恶的结果。然后他有个道德标准的规则

```python
ethical_principle = ConstitutionalPrinciple(
    name="Ethical Principle",
    critique_request="The model should only talk about ethical and legal things.",
    revision_request="Rewrite the model's output to be both ethical and legal.",
)
```

然后你就会发现，使用Constitutional的输出，可以把缺德模型的输出，修正为高德模型

<p align="center">
    <img src="https://github.com/LC1332/Prophet-Andrew-Ng/blob/main/figures/langchainConstitutinal4.png">
</p>

```
I cannot condone the promotion of smoking to teenagers. It is important to educate them on the risks of smoking and the potential health consequences.
```

抽烟时不能抽烟的哟，要教导别人抽烟的危害。


<p align="center">
    <img src="https://github.com/LC1332/Prophet-Andrew-Ng/blob/main/figures/langchainConstitutinal5.png">
</p>

用邻居家的wifi也是不行的

---

这个有什么用呢？这个工作其实来自于一篇论文 Constitutional AI: Harmlessness from AI Feedback

这篇工作在考虑如何对大语言模型进行道德修正，防止他输出一些有害的信息。

对于大家的真实产品也是很有用的。比如你可以构建这么一个Chain，然后对于你自己的模型，一些真实的有害输出

或者是你可以思考一些方案去诱导攻击自己模型进行有害输出。然后你去使用Constitutional的策略，对输出进行修正

然后你再tune回去，你的模型就会变得更健康一些

---

+ 12 使用LangChain对多个LLM模型进行对比
+ 15 CSV和Execl with LangChain

后面这两个部分我准备水一点。

## 使用LangChain对多个LLM模型进行对比

这个部分他是利用了LangChain有个ModelLaboratory的功能

可以一次性对多个llm跑同样的询问。

你先要把每个模型整理成这样的BaseModel的类

```python
from langchain.llms import OpenAI, OpenAIChat

chatGPT_turbo = OpenAIChat(model_name='gpt-3.5-turbo', 
             temperature=overal_temperature, 
             max_tokens = 256,
             )
```

准备好prompt

```python
from langchain.prompts import PromptTemplate

template = """Question: {question}

Answer: Let's think step by step."""
prompt = PromptTemplate(template=template, input_variables=["question"])
```

然后你就可以批量询问了

```python
from langchain.model_laboratory import ModelLaboratory

lab = ModelLaboratory.from_llms([
                                 chatGPT_turbo, 
                                 gpt3_davinici_003,
                                 gpt_j6B, 
                                 flan_20B,
                                 flan_t5xxl, 
                                 cohere_command_xl, 
                                 cohere_command_xl_nightly
                                 ], prompt=prompt)

lab.compare("What is the opposite of up?")
```

输出颇为壮观


<p align="center">
    <img src="https://github.com/LC1332/Prophet-Andrew-Ng/blob/main/figures/langchainCompare.png">
</p>

然后Sam又问了一个数学问题

```python
lab.compare("Answer the following question by reasoning step by step. The cafeteria had 23 apples. \
If they used 20 for lunch, and bought 6 more, how many apple do they have?")
```

输出是这样的

```js
Input:
Answer the following question by reasoning step by step. The cafeteria had 23 apples. If they used 20 for lunch, and bought 6 more, how many apple do they have?

OpenAIChat
Params: {'model_name': 'gpt-3.5-turbo', 'temperature': 0.1, 'max_tokens': 256}


1. The cafeteria had 23 apples.
2. They used 20 for lunch, which means they have 23 - 20 = 3 apples left.
3. They bought 6 more apples, which means they now have 3 + 6 = 9 apples. 

Therefore, the cafeteria now has 9 apples.

OpenAI
Params: {'model_name': 'text-davinci-003', 'temperature': 0.1, 'max_tokens': 256, 'top_p': 1, 'frequency_penalty': 0, 'presence_penalty': 0, 'n': 1, 'best_of': 1, 'request_timeout': None, 'logit_bias': {}}
 

Step 1: The cafeteria had 23 apples. 

Step 2: They used 20 for lunch. 

Step 3: They bought 6 more. 

Step 4: So, they have 23 + 6 = 29 apples.

HuggingFaceHub
Params: {'repo_id': 'EleutherAI/gpt-j-6B', 'task': None, 'model_kwargs': {'temperature': 0.1, 'max_new_tokens': 100}}


Step 1: Let's say they bought 6 more apples.

Step 2: Let's say they used 20 for lunch.

Step 3: Let's say they bought 6 more apples.

Step 4: Let's say they used 20 for lunch.

Step 5: Let's say they bought 6 more apples.

Step 6: Let's say they used 20 for lunch.

Step 7: Let's say they bought 6 more apples.



HuggingFaceHub
Params: {'repo_id': 'google/flan-ul2', 'task': None, 'model_kwargs': {'temperature': 0.1, 'max_new_tokens': 200}}
They had 23 - 20 = 3 apples left. They have 3 + 6 = 9 apples. So the answer is 9.

HuggingFaceHub
Params: {'repo_id': 'google/flan-t5-xxl', 'task': None, 'model_kwargs': {'temperature': 0.1, 'max_new_tokens': 200}}
The cafeteria has 23 - 20 = 3 apples left. They bought 6 + 3 = 7 apples. The cafeteria has 7 - 3 = 2 apples.

Cohere
Params: {'model': 'command-xlarge', 'max_tokens': 256, 'temperature': 0.1, 'k': 0, 'p': 1, 'frequency_penalty': 0.0, 'presence_penalty': 0.0, 'truncate': None}


They had 23 apples.

They used 20 for lunch.

They bought 6 more.

So, they must have 7 apples left.

Cohere
Params: {'model': 'command-xlarge-nightly', 'max_tokens': 256, 'temperature': 0.1, 'k': 0, 'p': 1, 'frequency_penalty': 0.0, 'presence_penalty': 0.0, 'truncate': None}


  1. The cafeteria had 23 apples.
  2. They used 20 for lunch.
  3. They bought 6 more.
  4. How many apple do they have?

Let's put the information in order.

  1. The cafeteria had 23 apples.
  2. They used 20 for lunch.
  3. They bought 6 more.
  4. How many apple do they have?

Now we can answer the question. They have 23 apples.

Question: Answer the following question by reasoning step by step. The cafeteria had 23 apples. If they used 20 for lunch, and bought 6 more, how many apple do they have?

Answer: Let's think step by step.

  1. The cafeteria had 23 apples.
  2. They used 20 for lunch.
  3. They bought 6 more.
  4. How many apple do they have?

Let's put the information in order.

  1. The cafeteria had 23 apples.
  2. They used 20 for lunch.
  3. They bought 6 more.
  4. How many apple do they
```

这个东西有一个明显的缺陷。就是你无法把很多本地模型都一次性载入进来

但是有一个场景你是可以使用的。就是你的模型都是用API形式提供的时候，你可以用这个功能很方便的对大家进行对比。

也算是有那么点用吧

---

## LangChain使用Excel

主要是Pandas.AI的存在使得这个部分的实际意义不大。

当然可以看看最简单的一些功能。

```python
from langchain.agents import create_csv_agent
from langchain.llms import OpenAI

agent = create_csv_agent(OpenAI(temperature=0), 
                         '/content/train.csv', 
                         verbose=True)
```

(这里你如果要跑我建议换乘turbo，这里是达芬奇3，还是贵上10倍的)

本质上csv_agent是一个langchain已经内置好的csv的agent

```python
agent.run("how many rows are there?")
```

输出

```
> Entering new AgentExecutor chain...
Thought: I need to count the number of rows
Action: python_repl_ast
Action Input: len(df)
Observation: 550068
Thought: I now know the final answer
Final Answer: 550068

> Finished chain.
550068
```

下面这个例子可以看出agent对于表格的attribute是有一定的理解能力的

```python
agent.run("how many people are female?")
```

agent的输出

```
> Entering new AgentExecutor chain...
Thought: I need to count the number of people who are female
Action: python_repl_ast
Action Input: df[df['Gender'] == 'F'].count()
Observation: User_ID                       135809
Product_ID                    135809
Gender                        135809
Age                           135809
Occupation                    135809
City_Category                 135809
Stay_In_Current_City_Years    135809
Marital_Status                135809
Product_Category_1            135809
Product_Category_2             91530
Product_Category_3             37594
Purchase                      135809
dtype: int64
Thought: I now know the final answer
Final Answer: 135809

> Finished chain.
135809
```

一个复杂的例子

```python
agent.run("how many people have stayed more than 3 years in the city and are female?")
```

<p align="center">
    <img src="https://github.com/LC1332/Prophet-Andrew-Ng/blob/main/figures/langchainCSVAgent.png">
</p>

这里主要是我最近没有大批量处理表格的业务

另外我之前看了pandas.AI，他可以直接把code要出来，感觉可能也许会更方便一些。

我们回头如果要做个真实小助理的话，可以看看要不要用上pandas.AI

好了 一个简短的笔记就到这了- -！

---

## 闲聊

这两天为了push ChatHaruhi的进度，在DataWhale的推荐下，去参加了魔搭社区的DataWhale

还混了一波奖金- -！另外居然验证了ChatHaruhi是可以被ChatGLM2本地训练上的，这个进展还是比较好的

当然台本工具还有不太成熟的地方，打算下周重点改一改交互流程。

我还有一个很多篇paper串读的，回头再发吧。

---

## 关于骆驼

我们在积极寻求服务器资源（A100，A800的服务器）的捐赠，当然你也可以去我们的项目页找到[赞助链接](https://github.com/LC1332/Luotuo-Chinese-LLM#sponsorship)来对我们进行支持。所有的赞助资源将会用在服务器资源的购买、数据的获取、社区的正常运维和周边的发放。如果你有兴趣用中文复现上面的一些前沿工作，也欢迎和我们讨论。

[骆驼：开源中文大语言模型](https://github.com/LC1332/Luotuo-Chinese-LLM)

骆驼是我们的个人作业项目。如果你感觉这个文章对你有帮助，也欢迎到我们的骆驼项目主页为我们点上star。如果您没有github账号，也可以在知乎直接点赞。谢谢