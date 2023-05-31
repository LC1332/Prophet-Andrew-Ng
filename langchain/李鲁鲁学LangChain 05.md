# 【骆驼LangChain笔记5】在抱脸Hub上运行Flan20B，LangChain本地运行模型

话说日更有点压力大，我打算后面佛系一点更新，但是如果想在6月份把不算这篇笔记在内的10篇都学完，

一周差不多还是要写个三篇。而且在今天这个笔记你就会看到其实langchain接入自己的模型没有那么那么舒适

所以有一点令我沮丧

---

## 无聊的作业加一

先说个知识图谱的，自从上次[那个笔记](https://zhuanlan.zhihu.com/p/633163462)之后，youtube给我推了一个知识图谱的demo视频。

https://www.youtube.com/watch?v=kL-IYn-p3qQ

大致就是你说话之后，这个东西会顺便抽取知识图谱，进行可视化。

这边建议每家做大模型的2B2G公司都可以做一个这样的产品。很可能就会出现这样的对话

```
旁白:领导试用了大模型，屏幕上稍过等待就显示出了领导刚才提到的“经济建设”，“揭榜挂帅”等关键词构成的知识图谱

旁白:领导微笑着点头表示满意，并和企业负责人亲切攀谈起来，稍后，领导转头小声说

领导:小张，你看，我就说还是会有个知识图谱吧，你看他们这个知识图谱就做的挺不错的

小张:对的，他们这个确实不错，有了这个就可以和我们之前的信息化xxx系统连接起来了

领导:你之后和他们的人问一下这个能不能对接，能对接的话就在xx新楼项目里用上去
```

我想到贾治峰的博士读的理论上也是知识图谱相关的，稍后找他一起再训个新时代的知识图谱triple抓取吧

---

## 在HuggingFaceHub上运行模型

https://www.youtube.com/watch?v=VW5LBavIfY4

这节课一方面有点水，约等于第四节课的llm全部改成Flan-ul2 20B

另一方面我没有付费的hugging face账号，所以只能看着sam自己跑代码

```py
from langchain.llms import HuggingFaceHub

flan_ul2 = HuggingFaceHub(
    repo_id="google/flan-ul2", 
    model_kwargs={"temperature":0.1,
                  "max_new_tokens":256})
```

其实就这么载入就可以了，HuggingFaceHub支持在线调用一些类似text generation管线的模型

当然只有一部分模型可以被api调用。

然后再使用上节课提到的ConversationChain就可以进行聊天了

```py
conversation = ConversationChain(
    llm=flan_ul2, 
    verbose=True, 
    memory=memory
)
```

后面所有的聊天内容和第4节是一样的。

我没有HF的付费账号，如果有人了解这部分的资费水平的话可以和我介绍一下

如果价格合适的话，能跑几十B的模型api还是挺方便的

---

## 快速进入第六节课吧

https://www.youtube.com/watch?v=Kn7SX2Mx_Jk&list=PL8motc6AQftk1Bs42EW45kwYbyJ4jOdiZ&index=6

其实Hugging Face Hub的在线模型只能支持text2text或者text-generation

而且只支持一部分模型，Sam举了一个"facebook/blenderbot-1B-distill"的模型并不能被API调用


所以我们肯定是希望本地去部署模型的

本地你可以使用你自己微调后的模型，可以用上自己的GPU，或者运行一些因为特殊原因只能放在本地的模型。


这里Sam本地跑了一个flan-t5-large玩一下

```python
from langchain.llms import HuggingFacePipeline
import torch
from transformers import AutoTokenizer, AutoModelForCausalLM, pipeline, AutoModelForSeq2SeqLM

model_id = 'google/flan-t5-large'# go for a smaller model if you dont have the VRAM
tokenizer = AutoTokenizer.from_pretrained(model_id)
model = AutoModelForSeq2SeqLM.from_pretrained(model_id, load_in_8bit=True)

pipe = pipeline(
    "text2text-generation",
    model=model, 
    tokenizer=tokenizer, 
    max_length=100
)

local_llm = HuggingFacePipeline(pipeline=pipe)

print(local_llm('What is the capital of France? '))
```

输出为

```
paris
```

然后是接入到LLMChain

```python
llm_chain = LLMChain(prompt=prompt, 
                     llm=local_llm
                     )

question = "What is the capital of England?"

print(llm_chain.run(question))
```

输出为

```
The capital of England is London. London is the capital of England. So the answer is London.
```

不过这里因为使用HuggingFacePipeline的原因，其实对于模型有一定的限制

这里我试图接入黄钟健博士训练的[迷你骆驼-3.5B](https://github.com/LC1332/Mini-Luotuo)就给我整失败了，让我觉得有点郁闷。

## LangChain载入Embedding模型

这里又给了一个例子

```py
from langchain.embeddings import HuggingFaceEmbeddings

model_name = "sentence-transformers/all-mpnet-base-v2"

hf = HuggingFaceEmbeddings(model_name=model_name)
```

然后输入

```py
hf.embed_query('this is an embedding')
```

就会发现模型输出了向量。

这里我觉得难受的是这里好像只找到了使用CPU的接口

另外这里我简单把模型转化为silk-road/luotuo-bert跑出来失败了

看来只能继续去后面的课程看怎么替换模型

---

总之这两节课的体验不是很好，后面两节课的内容是

+ 7 PAL： 就是那个把数学应用题转化为程序然后求解的PALChain，

+ 8: 用Langchain搭建一个摘要系统。

等我如果真的把我们的模型包括luotuo-bert, LuotuoQA-B和MiniLuotuo放上去再出对应的笔记吧

---

## Chat凉宫春日

对了，我最近在拿Chat凉宫春日交DataWhale举办的5月学习活动的作业

我们有一个针对Chat凉宫春日这个项目 招人的计划在这里 [招人计划](https://github.com/LC1332/Prophet-Andrew-Ng/blob/main/Hiring.md)

需要一些后端、前端和算法工程师，整个项目的ddl在6月6号和6月13号

如果你也正在参加DataWhale的课程或者这两天非常有空，可以联系我一起玩一下，做多做少都是缘，欢迎加入

另外昨天Chat凉宫春日在群里开放测了一下，大家反馈还都不错。不过有点费钱，如果你想测可以私信我 我发您链接。


## 关于骆驼

我们在积极寻求服务器资源（A100，A800的服务器）的捐赠，当然你也可以去我们的项目页找到[赞助链接](https://github.com/LC1332/Luotuo-Chinese-LLM#sponsorship)来对我们进行支持。所有的赞助资源将会用在服务器资源的购买、数据的获取、社区的正常运维和周边的发放。如果你有兴趣用中文复现上面的一些前沿工作，也欢迎和我们讨论。

[骆驼：开源中文大语言模型](https://github.com/LC1332/Luotuo-Chinese-LLM)

骆驼是我们的个人作业项目。如果你感觉这个文章对你有帮助，也欢迎到我们的骆驼项目主页为我们点上star。如果您没有github账号，也可以在知乎直接点赞。谢谢