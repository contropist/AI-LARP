借助世界上最大的单体中文NLP大模型(浪潮 源1.0 https://air.inspur.com/home ），我们做出了一个可以跟人类玩“剧本杀”的AI……

# 无图无真相

我们为本项目特别改编了一个微型线上剧本杀剧本，本子有五个角色，分别由五名玩家扮演，但我们每场只会召集四个玩家，并在他们不知情的情况下，派出AI扮演剩下的那个角色。

为了尽量避免引起玩家怀疑，我们不想让用户额外下载APP或者登录一个网页。对于绝大部分中国用户而言，微信就是线上最自然的交流方式，也是“最真实”的交流方式，为此我们需要打造一个“微信虚拟人”，
很幸运的，我们发现了wechaty这个神器（https://github.com/wechaty ）。

本着细节拉满的原则，编导组精心策划了这个账号的昵称、头像，甚至每场游戏前我们还会紧扣时事的为它准备近三天的朋友圈内容，而游戏后还会继续连发三天朋友圈内容提供延展剧情（非常类似"规则怪谈"）。

<img alt="img" height="600" src="https://github.com/bigbrother666sh/shezhangbujianle/blob/main/assets/af65z-209za.jpeg" width="300"/> <img alt="img" height="600" src="https://github.com/bigbrother666sh/shezhangbujianle/blob/main/assets/af65z-209za.jpeg" width="300"/>

整体剧情并不复杂，讲的是某高校社团中五个骨干成员因为一件事情牵涉到各自利益而产生的种种勾心斗角。玩家要做的也非常简单，就是想方设法、拉帮结派的说服其他人接受自己的主张……不过我们这次对原作做了比较大的改动，剧作中AI所扮演的角色（蔡晓）受控于某邪恶的科技巨头（“北极鹅”公司），
她要帮助“北极鹅”实行一个庞大的阴谋，而这个阴谋其实笼罩了所有人……坦率的说，从游戏角度，这个角色的难度还挺高，承担着推动剧情的作用，并且游戏机制设定最后所有的疑点矛头都会指向她，如果在现实的剧本杀游戏中，这个角色也应该是由DM扮演，而非普通玩家，当然这也就大大增加了对AI的考验。

下面四幅图展示了AI的实际表现效果（游戏中会要求玩家更改群昵称，而这里为了保护玩家隐私，也为了方便大家理解，我们直接把玩家的微信昵称备注为了角色名）。

### 谭明VS蔡晓（AI）

剧情中谭明为了实现自己的目的，不择手段的策划了一个诡计，并计划私下与蔡晓达成联盟，然而他不知道的是蔡晓其实在下一盘更大的棋，正想借他的诡计实现自己的阴谋……所以AI对谭明的策略就是可劲儿的忽悠他并想方设法利用他。
实际表现中，AI很好的贯彻了这个思路，甚至发挥想象力的使用了色诱绝技……坦率的讲，这招也极大的超出了我们的预料……

![img](https://github.com/WukongZeming/shezhangbujianle/blob/main/assets/%E5%B1%8F%E5%B9%95%E5%BD%95%E5%88%B62022-03-13%2014.48.44.gif)

### 孔墨VS蔡晓（AI）

### 李超VS蔡晓（AI）

### 孙若VS蔡晓（AI）

### 蔡晓（AI）在公聊（房间）

# 核心功能——“目的性对话”端到端生成方案

“剧本杀”是一种用户与用户之间博弈的游戏，玩家之间的对话可能更是无穷无尽，这种情况下使得传统的规则式或“词槽式目的域对话”方案均行不通，当然无目标导向的“开放域对话“就更不适合了。

本项目所使用的NLP大模型——**浪潮源1.0**是一种生成式预训练模型（GPT），其使用的模型结构是Language
Model（LM），类似于openAI的GPT-3，但是与GPT-3不同，源1.0更加擅长的是零样本（Zero-Shot）和小样本（Few-Shot）学习，而非目前更多模型所擅长的微调试学习（finetune）。从实际应用效果来看也确实如此，在2~
3个，甚至1个合适example的示范下，模型可以很好的理解我们希望实现的“对话策略”，仿佛具有“举一反三”的能力，但是如果没有example的话，那么模型的生成则非常不靠谱，甚至会出现答非所问的情况。因此，本项目的关键就在于如何针对用户的提问选择适当的example供给模型。

我们最终采取的方案是：建立example语料库，然后针对每次提问从语料库中选择最贴近的三个example作为模型生成的few-shot输入。

实际实现中，因为AI需要根据剧情对不同角色采用不同而回答策略，所以语料库被分装成4个TXT文件，程序会根据提问者去对应选择语料来源。这个机制的思路很简单，但是执行起来马上遇到的一个问题就是，如何从对应语料中抽取与当前提问最为相似的example？因为在实际游戏中， 玩家可能的提问措辞是无穷
无尽的。在这里我们用到了百度飞桨@PaddlePaddle 发布的预训练模型——**simnet_bow**
（https://www.paddlepaddle.org.cn/hubdetail?name=simnet_bow&en_category=SemanticModel
），它能够自动计算短语相似度，基于百度海量搜索数据预训练，实际应用下来效果非常不错，且运算速度快，显存占用低。这里不得不特别表扬下百度飞桨，尤其是其中的预训练模型开放库paddlehub，里面有不少非常实用的“小模型”，均可“开箱即用”。

解决了抽取合适example的问题之后，接下来就是合并example和用户当前提问文本生成prompt。玩过GPT类大模型的都知道，这类模型生成的本质是续写，Prompt兼有任务类型提示和提供续写开头的作用，机器不像人，同样的意思不同的Prompt写法可能导致差距十万八千里的生成结果，
盘古大模型群里有网友甚至说，他感觉GPT-3的prompt书写本身就相当于一门编程语言了……不过这次浪潮团队的技术支持可谓“暖男级”贴心，针对prompt生成、request提交以及reply查询，团队都给出了详细的、质量极高的范本代码（可以说也是我见过的大模型开源项目中给到的质量最高的示例代码），
好到什么程度呢？这么说吧，好到了我们直接拿来用的程度 :joy:……事实上，本项目代码库中的__init__.py、inspurai.py、url_config.py这三个文件都直接来自浪潮的开源代码（https://github.com/Shawn-Inspur/Yuan-1.0/tree/main/yuan_api ）。

至此所有的工程问题已经基本都解决了，剩下的就是语料来源问题，但这其实也是最核心的问题之一。GPT类大模型生成本质是根据词和词的语言学关联关系进行续写，它是不具有人类一样的逻辑能力的，即我们无法明确告知它在何种情况下应该采用何种对话策略，或者该往哪个方向去引导，
在本项目中这一切都得靠example进行“提醒”。打个不恰当的比方，AI相当于天资聪慧的张无忌，但是如果他碰到的不是世外高人，而都是你我这样的凡夫俗子，每天给他演示的就是如何上班摸鱼、上课溜号这些，它是绝无可能练出九阳神功的……
源1.0模型也是这样，虽然它背了5.02TB的中文数据，差不多相当于500多万本书了，但是它完全不懂城市的套路啊，也没玩过剧本杀，它能做的就是模拟和有样学样……所以这个AI在游戏中的表现就直接取决于我们给它的example如何。

对于这个问题，团队最终采取了一个非常简单粗暴的方案：编导组除主编外每人负责一个角色（刚好四人），自己没事儿就假装在玩这个游戏，想象看会跟AI提什么问题，然后再切换到AI的角度，思考合适的回答……初始语料文件好了之后，大家交换角色进行体验，每次体验后更新各自负责的语料库文件；
之后公测也是一样，每轮之后编导组都会根据当场AI回答得比较差的问题进行语料库的完善和补充……为此我们在程序中增加了一个功能：程序会把本场用户的每次提问，以及对应抽取出的三个example问题的simnet_bow相似度得分，并源1.0最终生成的回答文本，按语料库对应另存为4个文本文件，
以便于编导们针对性更新语料库（本项目目前开源提供的语料库是截止3轮公测后的版本）。

# 记忆机制

本来这个项目一开始是不打算引入记忆机制的，因为我们看源1.0在合适example的few-shot下生成效果已经很不错了，就琢磨着偷点懒。但在之后的测试中我们发现，会有用户习惯先提问再@，或者私聊中先发一句问题，然后再另发一句"你对这个问题的看法？"；另外我们也发现AI如果不记忆自己之前答案的话，
后续生成的结果会比较缺乏连续性，甚至给出前后矛盾的回答！这些问题迫使我们决定增加"多轮对话记忆机制"。

原理很简单，就是把之前若干轮次用户与AI的对话存在一个列表里面，然后提交生成的时候把这个列表和当前问题文本join一下，当然具体实施的时候，我们需要调整下提交的pre-fix和输出的pre-fix这些……我们一开始比较担心的是，这种记忆机制会不会跟exampl的few-shot机制有冲突，毕竟example都是
一问一答，没有多轮的例子,然而实践下来发现完全没有这个问题，且增加记忆机制后，AI因为生成依据变多，明显弥补了其逻辑能力的短板，如下图，是我们的一段测试对话，AI表现出了一定"逻辑推理能力"：

![img](https://github.com/bigbrother666sh/shezhangbujianle/blob/main/assets/a5xrh-20blg.jpeg)

然而当这个机制实际应用到本项目中时，我们马上就发现了新的问题，AI的回答变得紊乱，实际效果对比没有记忆机制反而是下降的！

经过分析，我们认为造成这种情况的原因可能有二：1、前面若干轮次的用户对话，虽然我们本意是为AI提供更多生成依据，但是这也同时增加了干扰，使得example的few-short效果降低；2、如果AI前面自己回复的内容就不是特别靠谱的话，这个回复文本作为后续轮次的输入，又会放大偏差；
事实上，对于这两个问题根本的解决方案是增加"注意力机制"，人类在日常生活中也不会记住所有事情、所有细节，没有遗忘的记忆其实等同于没有记忆，同理，***没有"注意力机制"的"记忆机制"其实对于对话AI来说是弊大于利的***。

然而，如果要引入"注意力机制"，那就要增加更加复杂的NLU算法，整个项目的复杂度会提高一个数量级（因为还存在一个"需要注意哪些"的问题）。好在本项目的实际应用场景更多的还是关注当前轮次的对话，所以我们可以用一个简化的处理方案——**只记忆当前轮次和上一轮次的对话**。
而对于需要遥远轮次对话内容回答的情况，AI可以托言"忘记了"，这对于真人来说，也是比较正常的现象。 实际测试下来，这个方案的效果还是相当不错的。另外在这个过程中，我们也尝试过只让AI记忆用户对话，而不记忆自己的回复，发现效果非常差，这可能是因为这种不对称的记忆实在跟example差的太多。好在只记忆一轮对话的情况下，不靠谱结果的"放大效应"也并不明显。另外对于群聊对话，我们也尝试过把并非需要AI参与的对话也进行记忆
，比如角色A与角色B之间的撕逼，没有涉及AI，一开始我们的想法是一旦突然要把AI拉进来呢？是不是这个时候AI知道背景会好一些，但实践下来效果也不理想，我们猜想原因可能还是没有注意力机制的记忆机制其实记不了多少，反而增加了干扰项，比如前面A一直说不赞同，这个时候
B拉AI进来，想让AI帮自己说话，而剧情设计此时AI也应该帮B说话，但是模型生成的时候，反而会被A反复的不赞同信息给带偏……

当然，我们承认，我们最终采用的这个"记忆力机制"并非最佳解决方案，仍然会有很多弊端，AI依然可能生成不符合剧情、甚至前后矛盾的回答，对于这个问题的终极解决方案我想可能需要引入一个seq2seq模型，通过这个模型先处理前序轮次对话和当前问题，再输入给NLP大模型进行生成。或者条件允许干脆直接上
seq2seq大模型，然后用目前的example语料进行微调，可能这样会炼出一个终极效果的AI……
另外熟悉NLP大模型的同学可能会说大模型本身不也有"注意力机制"么？其实这是两个层面的问题，一个是单纯的文本生成层面的"注意力"（transformer模型自带），一个是更高层面对于对话内容的"注意力"（也就是生成具体要依据哪些前序对话内容）。

# 导演机制
我们还必须澄清一点，现阶段的任何应用你都不能指望AI单独去完成，它依然需要人类的协助，或者监督、或者提示、或者backup……

在本项目上，我们为AI设定的辅助机制就是**"导演"**。他的作用是在出现AI不适用的情况下对AI进行"手动引导"，这些情况有一些是游戏机制本身，有一些是因为目前我们采用的wechaty puppet-xp协议限制（有关wechaty和puppet协议的关系可以查看wechaty主页（https://github.com/wechaty ）。

具体而言这些情况包括：
### 用户注册问题
首先由于目前puppet-xp拿不到用户的群昵称，所以程序无从得知哪个用户对应哪个角色，我们为此设定了一个游戏规则，即用户需要先添加其他用户好友，然后发一句"这是xxx"（角色名），我们会要求用户对所有其他玩家都这样做，这样就不会暴露AI，并且这也确实有必要，因为真人也不能每次都翻看群昵称
来确定发消息的玩家到底扮演的是谁。
但是我们不能保证每个用户都会遵守，所以就需要导演来手动帮助AI完成这个识别。
### 主动消息发送
项目测试中，我们也发现会出现冷场问题，比如一时间大家都不说话，或者某两个用户聊的火热，而忽略了其他玩家……从玩家角度来说，这是蛮主观的问题，可能我就是不知道该说什么，按程序"一问一答"的设计，这个时候AI也只能等。所以我们设定了主动消息或者说***闲聊***机制，具体来说就是预设一些问题，
导演账号向AI账号发送特定指令就会触发。
### 应急消息发送
目前puppet-xp仅能识别文本消息（当然浪潮源1。0目前也只支持text in text out，然而其实多模态的方案目前也不少），我们为此规定玩家必须发文本，这其实已经很限制了。结果测试中我们发现puppet-xp还不支持引用消息，而这在项目实际场景中出现的频率还不低！为此我们不得不针对一些常见问题
预设了回答文本， 当出现无法识别消息的情况，就由导演发送指令指挥AI发送……
有同学可能会问，你们不能直接用AI登录的微信客户端发送么？答案是——其实也可以，只是这样显得比较"不智能"，当然从玩家体验角度，可能这样也不错。只是这里面有一个小细节，你必须通过AI所登录的Windows客户端发送，而不通过同时登录的移动端发送，因为puppet-xp会错误的把多终端同步的信息
当成是对方发来的信息，这样的话，就会引发AI"回复自己的消息"。

话说，我们在预设消息中把AI设定为"神经比较大条"的性格，会不经意间"说漏嘴"一些剧情，所以实际应用下来，应急消息往往会产生比较戏剧性的效果。从作品本身而言，这个机制可能会是一个很好玩的东西，所以对于应急消息我们没有写在程序中，而是通过外部jason文件进行维护，大家可以随时增加。


另外，针对上一轮不好的生成结果有可能影响后续轮次生成的情况，我们也设计了"记忆遗忘机制"，导演可以随时让程序遗忘针对某个用户或者群聊的记忆，这也是应急机制的一部分。
### 流程引导
在用户视角中导演更像是"主持人"的角色，他负责拉群、分配角色等等，而对于AI也是这样，游戏什么时候开始、什么时候结束，这些也是导演通过指令告知程序的。而我们也设计了AI角色在游戏开始和结束时特殊的剧情。

说到这，其实我还是很希望puppet-xp能够再完善一些，可以支持群操作（拉群、解散群、加群）以及朋友圈操作（发送朋友圈、朋友圈留言等），这些都会更加丰富我们的玩法，比如AI可以在游戏中突然给玩家的最近一条朋友圈留言，而留言包含了一些线索，这一定很带劲！

有关导演机制详情，各位可以查看script文件夹下的《导演手册》。

# 其他一些细节
本项目其他一些细节主要是用户输入文本的预处理。比如，有些用户喜欢输入大段的空格或者频繁换行代替逗号或者句号，这就很容易造成simnet_bow报错，所以我们需要先用re把这些都替换为逗号；另外，上面说到的有些用户习惯先把问题提出再另外发一条只@人的消息，按我们最初的处理方案，
使用正则把@+昵称全部删除，就会产生"空文本"，simnet_bow还会报错，最后我们的方案是只去掉@，当然这可能会造成一句话中人名重复，但好在源1.0模型貌似对此的抗干扰能力还不错……

在测试的后期，我们还发现输入文本中如果包含#和&会导致源1.0API报错(https://github.com/Shawn-Inspur/Yuan-1.0/issues/13) , 浪潮团队建议我们进行转义替换，但我们觉得替换为"号"和"和"字可能会更贴近用户的真实意图……

类似的例子还有很多，比如puppet-xp无法识别除微信自带的第一页以外的emoji等等，考虑到出现频率不高，我们也就忽略了，所以稍微懂点技术的朋友，恳请你们测试的时候 ***千万不要抱着"极限测试"的思想*** ，关注AI生成本身就好:smile:。

话说100个人就有100种微信使用风格，这也算是本项目使用微信作为UI界面带来的额外挑战吧。

# 回到作品本身，先有创意还是先有技术？
本项目的初衷是结合NLP大模型和wechaty做一个好玩的东西，这是一个模糊的定义，但我作为项目发起人从一开始就清楚，这至少是一个创意和技术各占50%的事儿……

然而实践中，到底是先有技术还是先有创意却很纠结，如果我们先去做创意的话，那么很可能设计很多不可实现的东西，后期就得改创意；反过来如果从技术出发来考虑，那么做出来的东西一定不好玩，好的技术必然是"对用户不可见的"。最终对于这个问题，***我的答案是唯有一起考虑***！而这就要求项目主创
必然是懂技术的，所以各位能够在本项目中看到一个有趣的现象——写剧本的人跟写代码的人是同一个！我担心的是，可能随着AI的深化普及，可能python会成为每一个创作者的必备技能，好比现在没有作家不会用word，没有自媒体人不会用视频剪辑软件一样。

而引入AI后，带来的另一个有趣的问题是，本作还是一个"剧本杀"么？

不得不承认，本作最后的呈现与之前我们设想的不一样，或者说很不一样。NLP大模型的生成能力，使得AI可以和用户共同"演绎"出很多新的剧情， 比如下面这段，"谭明"找AI复盘，结果AI告诉他其实他和张家怡（游戏情节人物）是gay！

<img alt="img" height="650" src="https://github.com/bigbrother666sh/shezhangbujianle/blob/main/assets/WechatIMG337.jpeg" width="960"/>

针对这种情况，我们索性也直接在游戏规则中加入了一条："如果其他人提到了你所不知道的剧情，请相信那只是没有出现在你的剧本中，而非不存在"，并且我们也不提供复盘文本，而是鼓励他们互相（自然也包括AI）对信息进行复盘。
在这个复盘过程中，玩家也早晚会发现某个玩家其实是AI（剧情中AI所扮演的角色是一个被植入了AI程序的女大学生，但大部分玩家都会认为这个角色是我们的工作人员假扮的，而他们最后会发现这个账号真的只是一个AI！），
为此，我们准备了相应的延展剧情，在"剧本杀"游戏后1~3天以朋友圈的形式进行发送，这非常类似于 ***"规则怪谈"***，而互相私聊复盘的机制以及"共同恐惧"的剧情效果也让本作兼具 ***陌生人社交*** 的属性……可以说本作最好玩的部分恰恰是在"剧本杀"游戏结束后才刚开始。

这一切都让本作成为一部 ***"活着的故事"***，是一部由玩家和AI在不知不觉中共同创造的故事，我将之称为 ***"交互式叙事"***。（其实这就是我一开始最想实现的形式，但我不知道怎么实现，最后选择了剧本杀模式，但没想到最后我得到的还是我最初所设想的。）

而这种叙事有时也会产生非常感人的随机剧情。

<img alt="img" height="400" src="https://github.com/bigbrother666sh/shezhangbujianle/blob/main/assets/WechatIMG334.jpeg" width="960"/>

本项目中作为每次生成example的语料文件可以说直接决定了AI的表现，因此我们也一并将语料文件进行开源，因为在这种“开发范式”下，这些语料本质上就是代码的一部分。

而本项目中的人类编辑跟AI的关系也更像是"教练员与运动员"的关系，编导组会在每轮测试后根据AI当场表现针对性更新语料，从而提高AI后续的表现。这种人类教练员与AI运动员之间的"迭代合作"模式也是值得探讨的。（相对而言，目前虚拟人普遍的“中之人”做法相当于人类和提线木偶的关系。）

总之，本项目所积累的种种工程方案或可作为一种全新的“目的性对话”解决方案，不仅仅可以应用于娱乐创作，还可以泛化于教育、客服、销售、政务、心理咨询、情感陪护等各领域。

# 致敬

除综合使用了浪潮源1.0NLP大模型和百度飞桨的paddlehub预训练模型外，本项目涉及到的另一个关键组件是wechaty，这是一个可以几乎无缝对接包括微信在内一众主流IM软件的“神器”，
如果没有它，我们将只能让用户通过某个网页或者APP与AI进行交流，这不仅极大的增加了开发工作量，也大大削弱了实际体验，甚至无法实现我们的关键创意：让AI悄无声息的潜入玩家中，
并不知不觉的把所有人笼罩在“细思极恐”中。

wechaty操控微信是通过所谓的“wechaty-puppet”实现的，wechaty社区目前提供多个wechaty-puppet方案，我们这次使用的是完全免费的puppet-xp方案。
这是一个基于Windows微信客户端协议的方案，跟本项目非常适配，我们可以将主程序代码和微信客户端跑在同一台电脑上，部署最省事，然后还完全免费！
好比我写代码时最爱听一首歌在网易云音乐上的热评——既能听林忆莲，又能抖腿，你们还要啥？

不过也不得不说的是，puppet-xp目前的功能实在有限，仅能实现文本信息的收发，这导致了很多限制，甚至留下了一些硬伤。比如由于puppet-xp不支持引用格式的消息，
对于这类消息连正文文本都拿不到，所以如果有用户在群里引用消息再@AI的话，程序是没有任何反应的，这其实是本作一个无法遮蔽的穿帮点。
好在puppet-xp团队也在不停的升级开发，我们期待后续版本可以在免费的同时提供更多功能支持。现在NLP大模型都有往多模态发展的趋势，text2img甚至text2video都已经在路上，而这也是我下一个项目所将要尝试的功能。

然而在这里，还是要向 **@Wechaty** 社区致敬！”好用到哭“——你们对得起这个评价！

也向爱写诗的**laozhang**（puppet-xp作者）致敬！感谢你把这么好的东西无私的奉献了出来！

致敬**浪潮源1.0**团队、**百度飞桨**团队，致敬**github**和**csdn**，本项目实现过程中没少在这两个地方抄代码（其实不止本项目了，我想每个程序员都一样吧），总之致敬**开源社区**，致敬所有**开发者**！

最后向**Linus Torvalds**表达我由衷的敬佩！尤其当我因为本项目而系统的学习了git和github的历史之后，我更加要表达这种敬佩，这确实是一个凭一己之力改变世界的男人！

# 创作名单

### 主创&工程&测试导演

大兄弟666 —— Forgive me for having to remain mysterious

### 编导组成员

李青玲 —— 上海交通大学媒体与传播学院

胡璟葳 —— 华东理工大学艺术设计与传媒学院

赵家宁 —— 上海交通大学媒体与传播学院

翡冷翠的小羊 —— 跟这兄弟网友好多年，从来不知道真名，就更别提其他的了……

编剧指导：韩飞

感谢所有参与公测的同学，希望没有把你吓得太过，虽然这也正是本作的目的之一 :smirk: ……

特别感谢华东理工大学图书馆的**吉久明教授**
授予我们原作的改编权，同时还无偿贡献了配套的“角色选择心理测验“，通过16道是否题，就可以比较精确的匹配到跟玩家性格特征最贴近的角色，从而让整个游戏的沉浸感更加强烈，个人认为，这倒是每一个剧本杀都值得引入的一个机制。

# 写在最后的话

做这个项目是有感于去年大热的各种虚拟人，我丝毫不怀疑这个赛道的潜在价值，虽然我们目前还没有办法统一而清晰的定义作为人类最终归宿的“元宇宙”，但有一点是毋庸置疑的，即未来的元宇宙中，
虚拟人数量将数倍于真人，因为只有这样，才能让我们每个人过得比现实世界中更好。然而目前阶段，各路虚拟人在“好看的皮囊”方面已经愈发进步，然而在“有趣的灵魂”方面好像还都很欠缺，
靠“中之人”驱动毕竟不是长久之策；另一方面，自去年上半年我了解到NLP领域近两年来在生成式预训练大模型方面的长足进展后，也一直想看看基于这种大模型有什么可以实际落地的场景，
就这样，两个不同角度的想法合流成为了本项目的初衷。

而我选择将本项目全部代码开源，一方面在这个项目中我用了太多开源的资源，最后的项目成果不开源实在不地道；另一方面，也欢迎有高手和专业人士不断完善这个项目，
尤其欢迎中文NLP大模型研究机构移植本项目代码，看看自家模型在这种“目的性对话”应用方向的潜力。如果有机会能够看到各路AI机器人互相玩剧本杀我想会是一个有趣的事情，
而这种良性的比拼和竞争也必将对中文NLP大模型的发展有所裨益。

蔡晓和"北极鹅"的故事并未完结，让我们在这里最后上一张我们为蔡晓制作的"北极鹅"公司的工卡吧！
![img](https://github.com/bigbrother666sh/shezhangbujianle/blob/main/assets/a5xrh-20blg.jpeg)
作为一个喜欢将细节拉满的团队，这张工卡里面其实隐藏着两个彩蛋，你能找到么？：）
