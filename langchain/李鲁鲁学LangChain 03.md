# 【骆驼LangChain笔记3】Turbo接入LangChain

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


Traditionally, GPT models consumed unstructured text. ChatGPT models
instead expect a structured format, called Chat Markup Language
(ChatML for short).
ChatML documents consist of a sequence of messages. Each message
contains a header (which today consists of who said it, but in the
future will contain other metadata) and contents (which today is a
text payload, but in the future will contain other datatypes).
We are still evolving ChatML, but the current version (ChatML v0) can
be represented with our upcoming "list of dicts" JSON format as
follows:

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
You could also represent it in the classic "unsafe raw string"
format. However, this format inherently allows injections from user
input containing special-token syntax, similar to SQL injections:
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
## Non-chat use-cases
ChatML can be applied to classic GPT use-cases that are not
traditionally thought of as chat. For example, instruction following
(where a user requests for the AI to complete an instruction) can be
implemented as a ChatML query like the following:
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
Note that ChatML makes explicit to the model the source of each piece
of text, and particularly shows the boundary between human and AI
text. This gives an opportunity to mitigate and eventually solve
injections, as the model can tell which instructions come from the
developer, the user, or its own input.

## Few-shot prompting

In general, we recommend adding few-shot examples using separate
`system` messages with a `name` field of `example_user` or
`example_assistant`. For example, here is a 1-shot prompt:
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
If adding instructions in the `system` message doesn't work, you can
also try putting them into a `user` message. (In the near future, we
will train our models to be much more steerable via the system
message. But to date, we have trained only on a few system messages,
so the models pay much more attention to user examples.)

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



