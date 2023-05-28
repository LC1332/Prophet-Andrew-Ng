# 李鲁鲁学LangChain 01 LLMs + Prompting

这是李鲁鲁的学习笔记，原来是Sam Witteveen的笔记

原视频地址

https://www.youtube.com/watch?v=J_0qvRt4LNk

增加了李鲁鲁学习时候的吐槽

上这个课程的原因是Andrew的课程没有去照顾langchain的使用。

这里我先让GPT老师对着这个视频进行总结，给我总结出了一些标题

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


