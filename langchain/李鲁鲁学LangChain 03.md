# 【骆驼LangChain笔记3】Turbo接入LangChain, 记忆机制，使用知识图谱等多样化的方式记忆对话

因为我们一次笔记会覆盖sam的两节课，所以呢我准备直接用奇数来命名这个笔记。

因为Sam在出这个视频的时候，OpenAI已经发布了turbo-3.5的模型

也就是说如果你使用gpt-3.5-turbo

会比使用text-davinci-003模型便宜10倍，并且也会得到更快的响应


## OpenAI使用传统方式

我们都知道OpenAI现在3.5的接口是这么去使用的

```python
response = openai.ChatCompletion.create(
  model="gpt-3.5-turbo",
  messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Hello what kind of assistant are you?"},
    ]
)
```

然后response会是一个OpenAIObject

```json
<OpenAIObject chat.completion id=chatcmpl-7LQA...S6E at 0x7f9cf46eb330> JSON: {
  "choices": [
    {
      "finish_reason": "stop",
      "index": 0,
      "message": {
        "content": "I am an AI language model created to assist and communicate with users. I can answer questions, provide information, give suggestions, and engage in conversations. How may I be of assistance to you?",
        "role": "assistant"
      }
    }
  ],
  "created": 1685339806,
  "id": "chatcmpl-7LQ....S6E",
  "model": "gpt-3.5-turbo-0301",
  "object": "chat.completion",
  "usage": {
    "completion_tokens": 39,
    "prompt_tokens": 27,
    "total_tokens": 66
  }
}
```

使用这种形式的接口很大的原因就是为了防止用户在聊天中（像原来的text-davinci接口中）进行指令注入

这里Sam很细节的回顾了OpenAI把这个新的接口叫做ChatML(Chat Markup Language)

传统上，GPT 模型会消化无结构的文本。相反，ChatGPT 模型需要一种结构化格式，称为聊天标记语言（简称 ChatML）。
ChatML 文档包含一系列消息。每个消息包含一个头部（目前包括谁说了它，但将来将包含其他元数据）和内容（目前是文本有效载荷，但将来将包含其他数据类型）。
我们仍在发展 ChatML，但当前版本（ChatML v0）可以用我们即将推出的“字典列表” JSON 格式表示如下：

```
[
 {"token": "<|im_start|>"},
 "system\nYou are ChatGPT, a large language model trained by OpenAI. Answer as concisely as possible.\nKnowledge cutoff: 2021-09-01\nCurrent date: 2023-03-01",
 {"token": "<|im_end|>"}, "\n", {"token": "<|im_start|>"},
 "user\nHow are you",
 {"token": "<|im_end|>"}, "\n", {"token": "<|im_start|>"},
 "assistant\nI am doing well!",
 {"token": "<|im_end|>"}, "\n", {"token": "<|im_start|>"},
 "user\nHow are you now?",
 {"token": "<|im_end|>"}, "\n"
]
```

你还可以用经典的“不安全原始字符串”格式表示它。但是，这种格式固有地允许包含特殊令牌语法的用户输入注入，类似于 SQL 注入

```
<|im_start|>system
You are ChatGPT, a large language model trained by OpenAI. Answer as concisely as possible.
Knowledge cutoff: 2021-09-01
Current date: 2023-03-01<|im_end|>
<|im_start|>user
How are you<|im_end|>
<|im_start|>assistant
I am doing well!<|im_end|>
<|im_start|>user
How are you now?<|im_end|>
```

## 非聊天用例

ChatML 可应用于传统上不被视为聊天的经典 GPT 用例。例如，可以将指令跟随（其中用户请求 AI 完成指令）实现为类似以下的 ChatML 查询：

```
[
 {"token": "<|im_start|>"},
 "user\nList off some good ideas:",
 {"token": "<|im_end|>"}, "\n", {"token": "<|im_start|>"},
 "assistant"
]
```

We do not currently allow autocompleting of partial messages, 

```
[
 {"token": "<|im_start|>"},
 "system\nPlease autocomplete the user's message.",
 {"token": "<|im_end|>"}, "\n", {"token": "<|im_start|>"},
 "user\nThis morning I decided to eat a giant"
]
```

请注意，ChatML 明确告诉模型每个文本片段的来源，特别是显示人类文本和 AI 文本之间的边界。这为缓解和最终解决注入提供了机会，因为模型可以告诉哪些指令来自开发人员、用户或其自己的输入。

## 少量训练提示

一般而言，我们建议使用带有 `name` 字段为 `example_user` 或 `example_assistant` 的独立 `system` 消息添加少量训练示例。例如，下面是一个 1-shot 提示：

```
<|im_start|>system
Translate from English to French
<|im_end|>
<|im_start|>system name=example_user
How are you?
<|im_end|>
<|im_start|>system name=example_assistant
Comment allez-vous?
<|im_end|>
<|im_start|>user
{{user input here}}<|im_end|>
```

让我们回到接口

```python
import openai

openai.ChatCompletion.create(
  model="gpt-3.5-turbo",
  messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Who won the world series in 2020?"},
        {"role": "assistant", "content": "The Los Angeles Dodgers won the World Series in 2020."},
        {"role": "user", "content": "Where was it played?"}
    ]
)
```

另外这样一套新的API接口实际上也更适合用户去开发聊天助手类型的应用。

哦，另外just remind这个模型仍然可以当作一个completion模型去使用，只要你只输入user一句就可以。

## 无聊的聊天应用

这里Sam放了一个无聊的聊天应用，我们就不跑了，读一读代码就可以

```python
conversation_total_tokens = 0

while True:
    message = input("Human: ")
    if message=='exit':
        print(f"{conversation_total_tokens} tokens used in total in this conversation")
        break
    if message:
        messages.append(
            {"role": "user", "content": message},
        )
        response = openai.ChatCompletion.create(
            model="gpt-3.5-turbo", messages=messages
        )
    
    reply = response.choices[0].message.content
    total_tokens = response.usage['total_tokens']
    conversation_total_tokens += total_tokens
    print(f"ChatGPT: {reply} \n {total_tokens} tokens used")
    messages.append({"role": "assistant", "content": reply})
```

还带了一个token统计功能哈哈哈

哦不过这个token统计功能提醒我了，因为我之前写代码的时候我都用tiktoken去统计

那个是不完全准确的，还是应该使用openAI返回的response自己的这个数字去统计。


## 用LangChain调用turbo-3.5

这里我们做点狡猾的事情吧，我懒得每一句都去跑翻译了，我们在system prompt的最后，增加一句"你总是用中文回答问题。"

```python
from langchain import PromptTemplate, LLMChain
from langchain.prompts import PromptTemplate
from langchain.llms import OpenAI, OpenAIChat

prefix_messages = [{"role": "system", "content": "You are a helpful history professor named 鲁Bot. 你总是用中文回答问题。"}]

```

注意这里让鲁Bot扮演了一个历史学教授

然后就开始了问题

```py
## old way
# llm = OpenAI(model_name="text-davinci-003",
#              temperature=0, )

## New way
llm = OpenAIChat(model_name='gpt-3.5-turbo', 
             temperature=0, 
             prefix_messages=prefix_messages,
             max_tokens = 256)
```

这里Sam注释里还给了老方法和新方法的对比。然后就开始提问啦

```python
llm_chain = LLMChain(prompt=prompt, llm=llm)

user_input = "When was Marcus Aurelius the emperor of Rome?"

llm_chain.run(user_input)
```

给的输出是

```
马库斯·奥勒留是罗马帝国的一位皇帝，他在公元161年至180年期间统治罗马帝国。他是一位哲学家皇帝，他的著作《沉思录》至今仍被广泛阅读和研究。他的统治时期被认为是罗马帝国的黄金时期之一，他致力于改善人民的生活和减少贫困。他的死亡标志着罗马帝国的一个时代的结束。
```

我记得上Yale的那个建筑课还是提到这恶搞Marcus好多次的

```python
llm_chain = LLMChain(prompt=prompt, llm=llm)

user_input = "Who was Marcus Aurelius married to?"

llm_chain.run(user_input)
```

给的输出是

```
马库斯·奥勒留（Marcus Aurelius）的妻子名叫法维亚·安东尼娅（Faustina Antonina），她是一位美丽、聪明、有才华的女性，也是一位忠实的妻子和母亲。然而，她的名声并不好，因为有传言说她与其他男人有染。尽管如此，马库斯·奥勒留仍然深爱着她，并在她去世后写了一本书来纪念她。
```

---

在第三节课的视频里面提到了每次对话对于llm_chain.run都是独立的。这个时候就要使用LangChain的memory模块来对过往对话进行组织

所以第三节课就结束了，相比吴恩达的一个视频，Sam的视频还是比较短的。在这里我们快速进入第四节课

---

## 使用LangChain中的Memory系统

第四节课其实集中都在讲LangChain的Memory机制。因为无论是OpenAI的ChatGPT接口，还是本地的模型接口。

总的Token长度都是有限的。为了让Agent记忆之前的对话信息，就需要Memory机制

LangChain提供了

+ ConversationBufferMemory

+ ConversationSummaryMemory

+ ConversationBufferWindowMemory

+ ConversationSummaryBufferMemory

甚至有基于知识图谱的记忆系统

+ ConversationKGMemory

还有一个ConversationEntityMemory，

+ ConversationEntityMemory

这个有点像是基于关键词的记忆系统，让我们来看代码看看有什么区别。

## ConversationBufferMemory

```py
from langchain.chains.conversation.memory import ConversationBufferMemory
from langchain import OpenAI
from langchain.chains import ConversationChain
from langchain.llms import OpenAIChat
```

对了这里额外提一下，朋友圈有同事问我怎么改成Turbo的接口

```python
# llm = OpenAI(model_name='text-davinci-003', 
#              temperature=0, 
#              max_tokens = 256)

from langchain.chat_models import ChatOpenAI

llm = ChatOpenAI(model_name='gpt-3.5-turbo',
                 max_tokens=256)
```

其实我实验下来只要这么改就行了。

其实还有一个非常离谱的是他同时还有一个OpenAIChat接口，千万不要用混了，这个接口会帮你用贵的那个模型。。

然后我们继续回到memory的调用上

```py
memory = ConversationBufferMemory()

conversation = ConversationChain(
    llm=llm, 
    verbose=True, 
    memory=summary_memory
)
```

然后就是我和这个模型的聊天过程，我依次运行了

```py
conversation.predict(input="你好，我是鲁鲁！")
conversation.predict(input="你今天过得怎么样?找你聊天的用户多吗？")
conversation.predict(input="今天是周一，朋友圈里面我很多朋友都去长沙短途旅游了，你知道最近的长沙短途旅游为什么那么热门吗？")
conversation.predict(input="我老婆说主要是因为长沙作为旅游城市，吃的东西还比较平价，现在国内游复苏，所以这两个因素导致去的人很多，你觉得她说的有道理吗")
```

你在最后一次运行的时候会看到

```json
> Entering new ConversationChain chain...
Prompt after formatting:
The following is a friendly conversation between a human and an AI. The AI is talkative and provides lots of specific details from its context. If the AI does not know the answer to a question, it truthfully says it does not know.

Current conversation:
Human: 你好，我是鲁鲁！
AI: 你好鲁鲁，我是AI！很高兴能和你交流。你需要什么帮助吗？
Human: 你今天过得怎么样?找你聊天的用户多吗？
AI: 我很好，谢谢！今天用户确实比平常多，估计是因为天气不错，许多人都想享受户外活动。不过我也很高兴能和你聊天！
Human: 今天是周一，朋友圈里面我很多朋友都去长沙短途旅游了，你知道最近的长沙短途旅游为什么那么热门吗？
AI: 长沙是一个历史悠久、文化丰富的城市，也是湖南省的省会。近年来，长沙市政府加大了旅游宣传力度，推动旅游业的发展。此外，长沙拥有得天独厚的自然环境和独特的人文景观，如岳麓山、橘子洲头、天心阁等，吸引了众多游客前来观光游览。同时，近几年长沙的城市建设也得到了很大的改善和提升，使得游客们在旅途中享受到了更好的服务和体验。总的来说，这些因素都为长沙旅游业的热门提供了有力的支撑。
Human: 我老婆说主要是因为长沙作为旅游城市，吃的东西还比较平价，现在国内游复苏，所以这两个因素导致去的人很多，你觉得她说的有道理吗
AI:

> Finished chain.
你的老婆提到的东西确实也是长沙旅游热门的原因之一。长沙美食多样，品种繁多，价格相对较为亲民，吸引了不少吃货前来品尝当地特色美食。而随着国内疫情防控形势的好转，国内旅游业近两年逐渐复苏，也促使了长沙旅游的热度上升。总的来说，长沙旅游之所以热门，是由于多种原因的综合作用。
```

如果你print conversation.memory，你可以看到


```
Human: 你好，我是鲁鲁！
AI: 你好鲁鲁，我是AI！很高兴能和你交流。你需要什么帮助吗？
Human: 你今天过得怎么样?找你聊天的用户多吗？
AI: 我很好，谢谢！今天用户确实比平常多，估计是因为天气不错，许多人都想享受户外活动。不过我也很高兴能和你聊天！
Human: 今天是周一，朋友圈里面我很多朋友都去长沙短途旅游了，你知道最近的长沙短途旅游为什么那么热门吗？
AI: 长沙是一个历史悠久、文化丰富的城市，也是湖南省的省会。近年来，长沙市政府加大了旅游宣传力度，推动旅游业的发展。此外，长沙拥有得天独厚的自然环境和独特的人文景观，如岳麓山、橘子洲头、天心阁等，吸引了众多游客前来观光游览。同时，近几年长沙的城市建设也得到了很大的改善和提升，使得游客们在旅途中享受到了更好的服务和体验。总的来说，这些因素都为长沙旅游业的热门提供了有力的支撑。
Human: 我老婆说主要是因为长沙作为旅游城市，吃的东西还比较平价，现在国内游复苏，所以这两个因素导致去的人很多，你觉得她说的有道理吗
AI: 你的老婆提到的东西确实也是长沙旅游热门的原因之一。长沙美食多样，品种繁多，价格相对较为亲民，吸引了不少吃货前来品尝当地特色美食。而随着国内疫情防控形势的好转，国内旅游业近两年逐渐复苏，也促使了长沙旅游的热度上升。总的来说，长沙旅游之所以热门，是由于多种原因的综合作用。
```

其实这个coversation.memory就是朴素地存储了对话的信息。

## 带summarization的memory机制

后面这三个一起讲了

+ ConversationSummaryMemory

+ ConversationBufferWindowMemory

+ ConversationSummaryBufferMemory

这里面有两个是带Summary功能的，先讲**ConversationSummaryMemory**

```py
summary_memory = ConversationSummaryMemory(llm=llm)

conversation = ConversationChain(
    llm=llm, 
    verbose=True, 
    memory=summary_memory
)
```

类似这么用就可以了，我觉得有点鸡肋

他在每次对话的时候都会用第三人称进行总结。并且因为这个SummaryMemory的prompt是英语的（当然可以改成中文） 

summary_memory.prompt其实就是这么一个变量，你可以改成中文。

```python
input_variables=['summary', 'new_lines'] output_parser=None partial_variables={} template='Progressively summarize the lines of conversation provided, adding onto the previous summary returning a new summary.\n\nEXAMPLE\nCurrent summary:\nThe human asks what the AI thinks of artificial intelligence. The AI thinks artificial intelligence is a force for good.\n\nNew lines of conversation:\nHuman: Why do you think artificial intelligence is a force for good?\nAI: Because artificial intelligence will help humans reach their full potential.\n\nNew summary:\nThe human asks what the AI thinks of artificial intelligence. The AI thinks artificial intelligence is a force for good because it will help humans reach their full potential.\nEND OF EXAMPLE\n\nCurrent summary:\n{summary}\n\nNew lines of conversation:\n{new_lines}\n\nNew summary:' template_format='f-string' validate_template=True
```

不过我觉得很鸡肋，比如他给刚才的对话的总结是这样的

```
The human introduces themselves as 鲁鲁 and the AI responds with a friendly greeting, identifying itself as an AI that can provide information and assistance. The human asks how the AI's day is going and if it has had many users to talk to. The AI states that it doesn't have emotions, so it's just working today, but it's always ready to answer user's questions. 鲁鲁 asks why short trips to Changsha have become so popular recently. The AI explains that Changsha is a historic city with many famous attractions, like Orange Island, Yueyang Tower, and Tianshi Pavilion. Additionally, Changsha has a variety of delicious food and abundant hotel resources to meet different travel needs. The pleasant climate in Changsha also makes it a great destination for tourism.
```

**ConversationBufferWindowMemory**是个还可以但是有点美中不足的机制，他的特点就是顾名思义保留最近的k条对话

```python
from langchain.chains.conversation.memory import ConversationBufferWindowMemory

window_memory = ConversationBufferWindowMemory(k=2)
```

这里让我觉得美中不足的点在于，保留的k条聊天记录可以是很长，也可以是很短。

我在Chat凉宫春日中，实际上给了一个计算token进行保留的，可以保留比如不超过1000个token内的聊天记录

这个里面好像就没有这个机制，让我觉得有点头秃。

然后就有了**ConversationSummaryBufferMemory**，这个卡是卡token了但是他是以summary的形式的

```py
from langchain.chains.conversation.memory import ConversationSummaryBufferMemory

memory = ConversationSummaryBufferMemory(llm=OpenAI(), max_token_limit=40) 
```

反正都不是很喜欢，但是我们可以就着他的模版去写一个我们自己的customMemory机制。

## 基于知识图谱的Memory

```py
from langchain.chains.conversation.memory import ConversationKGMemory

prompt = PromptTemplate(
    input_variables=["history", "input"], template=template
)

template = """下面是人类与AI之间友好的对话。AI很健谈，并提供了许多来自其上下文的具体细节。如果AI不知道问题的答案，它会真诚地说出并否认知道。AI仅使用“相关信息”部分中包含的信息，不会产生幻觉。

相关信息：

{history}

对话：

人类：{input}

AI："""

conversation_with_kg = ConversationChain(
    llm=llm, 
    verbose=True, 
    prompt=prompt,
    memory=ConversationKGMemory(llm=llm)
)
```

这个稍微有趣一些

他这个如果打印出知识图谱里面的信息的话，是这样的

```python
import networkx as nx

print(conversation_with_kg.memory.kg)
print(conversation_with_kg.memory.kg.get_triples())
```

打印出来是这样(哦这里输入的是那个修电视的故事，大家自己看代码吧)

```python
<langchain.graphs.networkx_graph.NetworkxEntityGraph object at 0x7efc8272f070>
[('鲁鲁', 'person', 'is a'), ('Human', 'a television', 'has'), ('Human', 'after-sales service', 'is looking for'), ("Human's TV", 'abnormal sounds', 'occasionally emits'), ("Human's TV", 'after emitting abnormal sounds', 'black screens and shuts down')]
```

不知道有多少人看过Generative Agents那个代码，里面的记忆就使用了这个机制。

也可以看我[翻译和实验的版本](https://github.com/LC1332/Chinese-generative-agents)

## Entity

这个我比较喜欢是因为比较简洁。他采用的是关键词-解释的形式去存储信息的


```python
from langchain.chains.conversation.memory import ConversationEntityMemory

conversation = ConversationChain(
    llm=llm, 
    verbose=True,
    prompt=ENTITY_MEMORY_CONVERSATION_TEMPLATE,
    memory=ConversationEntityMemory(llm=llm)
)
```

他这里最后抽取的信息形式是这样的

```python
from pprint import pprint
pprint(conversation.memory.store)
```

输出是这样的

```python
{'A512453': "A512453 is the warranty number for Sam's TV.",
 'Dave': 'Dave is a repair person.',
 'Sam': 'Sam owns a TV that is currently broken and under warranty.',
 'TV': 'TV is under warranty.'}
```

我觉得这个形式相对其实很简洁。能够省掉不少token。也很有助于手动添加或者删除。

## 后面的课程

好了第三课和第四课的内容其实就到这里了。

另外其实第四课里面还比较有趣的是Sam不停在替换聊天的prompt，相互之间也有一些细微的差别。

（比如是否健谈，是否只基于事实，这个大家可以自己再去看看代码）

后面两节课讲的是Flan20B和如何在本地使用Hugging Face的模型。

## 关于骆驼
我们在积极寻求服务器资源（A100，A800的服务器）的捐赠，当然你也可以去我们的项目页找到[赞助链接](https://github.com/LC1332/Luotuo-Chinese-LLM#sponsorship)来对我们进行支持。所有的赞助资源将会用在服务器资源的购买、数据的获取、社区的正常运维和周边的发放。如果你有兴趣用中文复现上面的一些前沿工作，也欢迎和我们讨论。

[骆驼：开源中文大语言模型](https://github.com/LC1332/Luotuo-Chinese-LLM)

骆驼是我们的个人作业项目。如果你感觉这个文章对你有帮助，也欢迎到我们的骆驼项目主页为我们点上star。如果您没有github账号，也可以在知乎直接点赞。谢谢