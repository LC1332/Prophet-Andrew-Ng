# 【LangChain笔记25】Kor文本格式化抽取，韦小宝和哪个老婆互动多

周一终于坐到办公室，打上咖啡，可以趁着人都没来写份笔记

上周去深圳出差，回来发现已经是周五了就不太想笔记。周末的时候研究了一下Kor这个文本抽取。

Kor是个支持LangChain的文本抽取库，可以把文本抽取成json格式。

<p align="center">
    <img src="https://github.com/LC1332/Prophet-Andrew-Ng/blob/main/figures/langchainKorExample.png">
</p>

这东西的作者居然是MIT物理系毕业的，好吧所以物理系出来都转码是吧。。

今天的笔记打算只讲Kor一个内容。因为我们在Chat凉宫春日中，其实有大量的台词抽取任务。我其实一个月以前就在思考，是不是可以用这个东西批量的金庸小说抽取，但是当时我是在思考骆驼大乱斗的，但是凉宫春日这个中，也一样有类似的任务，而且更简单，我只要抽取dialogue对话，并不需要抽取很精确的技能和招式。

<p align="center">
    <img src="https://github.com/LC1332/Prophet-Andrew-Ng/blob/main/figures/langchainLudingji.png">
</p>

（知乎版本的笔记这个会换成视频，github不支持视频超过10M）

看起来抽取的效果还是非常不错的。顺便一提，其实这个Kor似乎知道的人很少，因为我在github反查代码，只有40个左右的repo使用了kor

<p align="center">
    <img src="https://github.com/LC1332/Prophet-Andrew-Ng/blob/main/figures/langchainCodeUseKor.png">
</p>

相比于Output Parser，因为后者在吴恩达的langchain笔记中被提到了，所以用的人很多。

---

## 简单调用Kor

好了我们开始笔记的代码部分

你可以点击 [这个笔记](https://github.com/LC1332/Prophet-Andrew-Ng/blob/main/langchain/%E6%9D%8E%E9%B2%81%E9%B2%81%E5%AD%A6LangChain_25_Kor%E4%BF%A1%E6%81%AF%E6%8A%BD%E5%8F%96.ipynb) 查看notebook的版本，如果你有openAI的key应该可以直接运行。

```shell
!pip -q install langchain openai tiktoken
!pip -q install kor markdownify
```

这里我不知道markdownify这个为什么要安装，可能是kor库的依赖

然后我们先给出模型的部分

```python
from kor.extraction import create_extraction_chain
from kor.nodes import Object, Text, Number
from langchain.chat_models import ChatOpenAI

from langchain.llms import OpenAI
llm = ChatOpenAI(
    model_name="gpt-3.5-turbo",
    temperature=0
)
```

然后我们可以定义kor中一个称为抽取schema的数据结构

```python
schema = Object(
    id="script",
    description="Adapted from the novel into script",
    attributes=[
        Text(
            id="role",
            description="The character who is speaking or performing an action",
        ),
        Text(
            id="dialogue",
            description="The dialogue spoken by the characters in the sentence",
        ),
        Text(
            id="action",
            description='''The actions performed by the characters in the text, A high-level summary of a character's behavior. action equals "对话" or "独白" if it's dialogue in sentence, equals other action if it's no dialogue''',
        )
    ],
    examples=[
        (
            '''韦小宝心中闪过一个念头：“我如得了这一千两，我和妈娘儿俩可有得花了” 见茅十八仍是侧头瞧着自己，韦小宝怒道：“你肯定猜我会去通风报信，领这赏银”茅十八道：“是啊，银子谁不爱”''',
            [
                {"role": "韦小宝", "dialogue": "我如得了这一千两赏银，我和妈娘儿俩可有得花了","action":"独白"},
                {"role": "韦小宝", "dialogue": "你肯定猜我会去通风报信，领这赏银","action":"对话"},
                {"role": "茅十八", "dialogue": "是啊，银子谁不爱","action":"对话"},
            ],
        ),
        (
            '''澄观一直站在禅房门口等候。韦小宝和那女郎的对答，虽微声细语，亦无不入耳，只觉这位师叔“劝说”女施主的言语，委实高深莫测，正自感佩赞叹，听得他问起解穴之法，忙道：“这位女施主被封的是大包穴，师叔替她解开即可”''',
            [
                {"role": "澄观", "dialogue": "","action":"站在禅房门口听韦小宝和女郎对话，听到韦小宝询问解穴方法"},
                {"role": "澄观", "dialogue": "这位女施主被封的是大包穴，师叔替她解开即可","action":"对话"},
            ],
        )
    ],
    many=True,
)
```

这里比较重要的是attributes和examples，这两者会以显著的形式出现在最终运行的pormpt中

我们先看看运行结果吧。我们随便找一段鹿鼎记中间的段落

```python
response = chain.run(
        """黄龙使殷锦忙道：“夫人高见。取经之事，想来和福份大小，干系极大。黑龙使也不是不努力。不肯替教主立功，可是始终阻难重重，多半是福气不够，因此宝经难以到手。”洪夫人微笑道：“依你之见，谁的福份够呢”殷锦道：“本教福气最大的，自然是教主他老人家，其次是夫人。不过总不能劳动两位大驾亲自出马。更其次福份最大的，首推白龙使。他识得碣文，又立下大功，印堂隐隐透出红光，福份之大，教主属下无人能出其右。”

    教主捻须微笑，道：“但他小小孩童，能担当这件大任么”

    白龙使一职，在神龙教虽然甚尊，在韦小宝心里，却半点份量也没有，他既陷身岛上，只好随遇而安，瞧着闭月羞花的洪夫人。自是过瘾之极，但瞧得多了，如给教主发觉自己色迷迷的神色，难免有杀身之祸，还是尽速回北京为妙，听教主这么说，正是脱身的良机，便道：“教主，夫人，承蒙提拔，属下十分感激，我本事是没有的，但托了两位大福气，混进皇宫中去偷这四部宝经，倒也有成功的指望。”洪教主点了点头。洪夫人喜道：“你肯自告奋勇，足见对教主忠心。我知你聪明伶俐，福份又大，恐怕正是上天派来给教主办成这件大事的。”"""
)["data"]
```

这个抽取完因为`many=True`的原因，所以会抽取成一个list，我们把list中的元素依次打印出来看看

```python
for chat in (response['script']):
    print(chat)
```

输出结果

```js
{'role': '黄龙使殷锦', 'dialogue': '夫人高见。取经之事，想来和福份大小，干系极大。黑龙使也不是不努力。不肯替教主立功，可是始终阻难重重，多半是福气不够，因此宝经难以到手。', 'action': '对话'}
{'role': '洪夫人', 'dialogue': '依你之见，谁的福份够呢', 'action': '对话'}
{'role': '殷锦', 'dialogue': '本教福气最大的，自然是教主他老人家，其次是夫人。不过总不能劳动两位大驾亲自出马。更其次福份最大的，首推白龙使。他识得碣文，又立下大功，印堂隐隐透出红光，福份之大，教主属下无人能出其右。', 'action': '对话'}
{'role': '教主', 'dialogue': '但他小小孩童，能担当这件大任么', 'action': '对话'}
{'role': '白龙使', 'dialogue': '教主，夫人，承蒙提拔，属下十分感激，我本事是没有的，但托了两位大福气，混进皇宫中去偷这四部宝经，倒也有成功的指望。', 'action': '对话'}
{'role': '洪教主', 'dialogue': '点了点头', 'action': '对话'}
{'role': '洪夫人', 'dialogue': '你肯自告奋勇，足见对教主忠心。我知你聪明伶俐，福份又大，恐怕正是上天派来给教主办成这件大事的。', 'action': '对话'}
```

看起来非常不错的。这里我区分了对话、独白，以及希望在没有对话的时候，kor能够把角色的行为总结到action。

---

## Kor的原理

Kor的原理其实还是挺简单的，其实Kor的整个库都是围绕一个prompt来展开的

我们可以通过以下方式打印出Kor的prompt

```python
chain = create_extraction_chain(llm, schema)
print(chain.prompt.format_prompt(text="[user input]").to_string())
```

打印出来是这样的哈

```js
Your goal is to extract structured information from the user's input that matches the form described below. When extracting information please make sure it matches the type information exactly. Do not add any attributes that do not appear in the schema shown below.

```TypeScript

script: Array<{ // Adapted from the novel into script
 role: string // The character who is speaking or performing an action
 dialogue: string // The dialogue spoken by the characters in the sentence
 action: string // The actions performed by the characters in the text, A high-level summary of a character's behavior. action equals "对话" or "独白" if it's dialogue in sentence, equals other action if it's no dialogue
}>
```

prompt其实是这样的

```
Please output the extracted information in CSV format in Excel dialect. Please use a | as the delimiter. 
 Do NOT add any clarifying information. Output MUST follow the schema above. Do NOT add any additional columns that do not appear in the schema.


Input: 韦小宝心中闪过一个念头：“我如得了这一千两，我和妈娘儿俩可有得花了” 见茅十八仍是侧头瞧着自己，韦小宝怒道：“你肯定猜我会去通风报信，领这赏银”茅十八道：“是啊，银子谁不爱”
Output: role|dialogue|action
韦小宝|我如得了这一千两赏银，我和妈娘儿俩可有得花了|独白
韦小宝|你肯定猜我会去通风报信，领这赏银|对话
茅十八|是啊，银子谁不爱|对话

Input: 澄观一直站在禅房门口等候。韦小宝和那女郎的对答，虽微声细语，亦无不入耳，只觉这位师叔“劝说”女施主的言语，委实高深莫测，正自感佩赞叹，听得他问起解穴之法，忙道：“这位女施主被封的是大包穴，师叔替她解开即可”
Output: role|dialogue|action
澄观||站在禅房门口听韦小宝和女郎对话，听到韦小宝询问解穴方法
澄观|这位女施主被封的是大包穴，师叔替她解开即可|对话

Input: [user input]
Output:
```

+ 可以看到Kor的抽取实际上是让语言文字根据input输出了csv格式

+ 这里我怀疑example和input用三个反引号括起来会更好一些。。

+ 直觉感觉Kor这种先抽取csv格式，再整理成json的形式，对语言模型更友善，对于Output Parser直接输出json其实对语言模型的压力是比较大的。毕竟json是一个带迭代层次的结构

+ Kor的官方建议在整个attribute后面去加example，而不是每个attribute各自加

---

## 使用LangChain对整个小说进行抽取

有了基础的代码后，我们很快就能拓展到整个小说的抽取

<p align="center">
    <img src="https://github.com/LC1332/Prophet-Andrew-Ng/blob/main/figures/langchainKorRuning.png">
</p>

这里我就花一个晚上，对鹿鼎记整个小说进行了抽取。可以发现中间有好几次没有抽取到任何对话，我暂时还没时间去检查里面对应的文本是什么，我觉得应该有可以优化的空间

<p align="center">
    <img src="https://github.com/LC1332/Prophet-Andrew-Ng/blob/main/figures/langchainKorLudingji.png">
</p>

好吧，结果就是这样的。对于这个结果我的个人感受是这样的

+ 相比于从TV字幕里面去提取，这个方法的效率太高了，一晚上就可以得到主角近万条对话。即使有一些噪音，效率也快乐很多

+ 有一些人物相互是重复的，比如沐剑屏和小郡主是一个人（当然模型从文本片段上肯定看不出来，加起来有224次了，仅次于阿珂）同样重复的还有

```python
replace_map['公主'] = '建宁公主'
replace_map['小玄子'] = '康熙'
replace_map['苏荃'] = '洪夫人'
replace_map['九难'] = '白衣尼'
replace_map['沐剑屏'] = '小郡主'
```

+ 当然主动行为的出现次数肯定是 康熙 > 双儿 > 海公公，所以除了大老婆双儿之外，康熙和海公哦那个才是真爱（误）

曾柔从统计上直接消失了，我去看原文本，发现曾柔整个小说文本中出现了70几次，大概都是侧写，有点惨。

+ action不一定是独白和对话，这个还需要人工check，不过工作量不大

```python
action_for_dialogue = ['对话', '对句', '骂道', '惨呼', '惊叫', '大叫', '对骂', '叫道',  '叫声', '插口', '怒骂','对方怡说']
```

这个代码包括那个炫酷的plotly可视化，我放在 [这个脚本](https://github.com/LC1332/Chat-Haruhi-Suzumiya/blob/main/notebook/novel_dialogue_analysis.ipynb)


## 从抽取后的数据中得到段落型对话

因为我们是把文档切片之后逐段抽取的。我们希望把这个很长的json转化为最终我们chat凉宫春日架构能够使用的记忆库文本，

所以我大致实现了一个这样的状态机：

+ 先找到要抽的角色的对话的一句话

+ 向前寻找一个主要角色的话，用来导引这句话

+ 在token数量允许的前提下增加对话

+ 如果角色数过多，或者token过多，则退出，切成一个chunk

结果差不多是这样的，看起来还是够用的

<p align="center">
    <img src="https://github.com/LC1332/Prophet-Andrew-Ng/blob/main/figures/langchainKorResult.png">
</p>


---

## Kor的更多特征

你可以到[这个网站](https://eyurtsev.github.io/kor/tutorial.html)

去看更多的Kor的特性，

+ 比如他还支持Nest，我觉得不是很好用。我更愿意把所有attribute写开。

+ 还有Pydantic的支持，可以让你的json格式更严格，比如引入一个validator

## 后续

这个Kor我还会持续使用下去，今天可能抽取一下天龙八部，因为之前天龙八部的几个主角我们也有小伙伴希望做一下ChatBot

（虽然我自己不是很想和乔峰聊天）

哈利波特我抽取了Dumbledore, Harry, Hermione, Luna, Malfoy, McGonagall和Snape这些角色

回头打算试一下把这些语料放到ChatBot里面看看效果。

另外在Chat凉宫春日这个项目之外，我还打算去详细抽取金庸的武功和技能描述，因为那些打斗场面的描述还是挺有趣的，放到游戏里应该会不错。

今天的笔记就到这里。


笔记实际可以在[骆驼先知](https://github.com/LC1332/Prophet-Andrew-Ng/blob/main/langchain/%E6%9D%8E%E9%B2%81%E9%B2%81%E5%AD%A6LangChain25.md)的页面直接看到

知乎的版本是从md顺便自动转化的。langchain的笔记看起来还有4-5篇，写完之后可以再出个目录总结篇。

发现带娃一周出差一周回来已经要8月了。。。

今天先写Kor是因为正好 

[Chat凉宫春日](https://github.com/LC1332/Chat-Haruhi-Suzumiya)

这个项目要用掉。

## 关于骆驼

我们在积极寻求服务器资源（A100，A800的服务器）的捐赠，当然你也可以去我们的项目页找到[赞助链接](https://github.com/LC1332/Luotuo-Chinese-LLM#sponsorship)来对我们进行支持。所有的赞助资源将会用在服务器资源的购买、数据的获取、社区的正常运维和周边的发放。如果你有兴趣用中文复现上面的一些前沿工作，也欢迎和我们讨论。

[骆驼：开源中文大语言模型](https://github.com/LC1332/Luotuo-Chinese-LLM)

骆驼是我们的个人作业项目。如果你感觉这个文章对你有帮助，也欢迎到我们的骆驼项目主页为我们点上star。如果您没有github账号，也可以在知乎直接点赞。谢谢



