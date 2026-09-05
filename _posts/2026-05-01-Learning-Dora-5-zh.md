---
title: "Regulatory Knowledge Platform 01｜我只是想让AI读DORA，后来发现PDF本身就是问题"

date: 2026-05-01
categories:
  - Cybersecurity
tags:
  - Cybersecurity
  - Network Security
  - Fintech
  - DORA
  - Vulnerability Management
  - Digital Operational Resilience Act
  - GRC
  - Regulatory Compliance
  - EU Regulation
  - Governance, Risk, and Compliance
  - ICT Risk Management
---
前面写DORA 02的时候，我还在纠结DORA到底有多少份文件。一开始以为找到Regulation (EU) 2022/2554就结束了，后来又发现2556，然后是各种Delegated Regulation、Implementing Regulation、RTS、ITS、Guidelines、Final Report，再加上EBA、ESMA、EIOPA和CSSF自己的东西。

最重要的是，我看不懂！！！ 

想想是把，一个CS背景的人，对法律相关的东西完全没有概念。刚开始我以为只要把这些PDF丢给AI，它就能帮我理解。后来发现，问题不在AI，而在PDF本身。

当时我花了不少时间把这些文件一个个下载下来，最后在电脑里搞了一个文件夹。十几二十个PDF，名字也改好了，大概知道谁是谁了，看起来还挺有成就感。我的想法也特别直接，反正现在AI这么厉害，把这些全部丢进去不就完了吗。实际上我最早的Regulatory RAG差不多也是从这里开始的。

做着做着发现被AI吊着走了，自己任然没有理解这些东西。

## 一个PDF到底是什么

比如Regulation (EU) 2022/2554，从电脑角度看就是一个`2022_2554_EN.pdf`。高级一点，无非再加几个metadata：

```text
{
  "filename": "2022_2554_EN.pdf",
  "title": "Digital Operational Resilience Act"
}
```

但这些信息对我真正要做的事情其实远远不够。
学了很多年英文，但是专业词汇背后的含义并不了解。比如，Regulation还是Directive？CELEX是什么？正式ELI链接是什么？是不是当前有效版本？Article 5在哪里？Article 5下面有几个paragraph？属于哪个Chapter？它讲的management body又在哪些地方继续展开？某个Level 2 Regulation到底是在补充2554的哪一部分？我现在拿到的这个文件到底是正式生效的Commission Delegated Regulation，并且我不知道还有Final Report这个说法？

这些东西文件名基本都看不出来的。

PDF当然没有什么问题，PDF适合阅读，不代表它适合成为系统里面的知识结构。比如Article 5。我第一次学DORA的时候，主要理解成ICT Risk最终不能全部甩给IT，management body承担最终治理责任。继续往后看，会发现这个概念并不会停在Article 5，它还会继续出现在ICT risk management framework、review、approval、reporting这些要求里面，再往下还有Level 2的细化。

所以我真正想知道的其实不是：Article 5在PDF第几页？而是：Article 5在整个DORA体系里面处在什么位置，它和后面的要求到底怎么连起来？

这时候PDF的页码对我的意义就开始下降了。Chapter、Article、Paragraph才是法规自己的结构。结构是我要学习的第一层。

逐渐的，意识到我缺的不是更多PDF，刚开始一直在做加法。缺一个文件就下载一个，发现一个新的RTS再下载一个，CSSF又出了相关材料就继续往里面放。很容易产生一种错觉：我的文件越来越全，系统应该也越来越聪明。

但假设我现在有100份法规，如果它们在系统里面仍然只是file_001...file_100。后来实际做RAG就越来越明显。每次模型拿到一个chunk，都得重新判断：这是什么法规？Article几？属于哪一层？是不是正式文本？跟主法规是什么关系？有时候chunk切得不好，上下文一丢，连这个段落属于Article几都可能要猜。

不能每次都要重新让LLM猜一次？我要做的是grounded evidence。

CELEX是什么，不需要AI判断。Article 5叫什么，也不需要AI推理。某个Commission Delegated Regulation和DORA主法规是什么法律关系，如果官方数据里面本来就有，也没必要每次问问题的时候重新推测。

慢慢就形成了一个我现在的方向：

> **做一个跟大模型协同工作的知识图谱底层系统**

LLM应该去做它擅长的东西，比如理解一个用户问题到底在问什么，几条法规之间有没有实质上的共同要求，某段义务在一个具体场景里面应该怎么解释。但法规自己的身份、结构、来源和明确关系，本来就应该先存在data layer里面。

## 法规也不是一个整齐的文件夹

我在DORA 02里面为了方便自己理解，画过一个特别简单的结构：

```text
Level 1
   ↓
Level 2
   ↓
Level 3
```

现在我还是觉得这个图没错，刚开始学DORA的时候特别有用。但真开始做数据以后就会发现，法律世界根本没有这么整齐。

有的是supplement 2554，有的是implement某一项要求；2556也不是2554的“下一层”，它主要是在修改原来的金融行业Directives。一个Regulation可能引用另一个Regulation，Article也会引用Article。某个RTS在起草阶段可能先有Consultation Paper、draft RTS、Final Report，最后才变成正式的Commission Delegated Regulation，而且名字还经常长得特别像。

再加上法规本身还会变化。可能有amendment，有corrigendum，也可能有consolidated version。所以如果真准备把这个做成长期使用的系统，“我电脑里面存了一份PDF”肯定是不够的。至少要知道：这是什么legal resource，哪个版本，什么时间点的状态，正式来源在哪里。

做到这里的时候，我才意识到，我之前可能一直把问题想反了。

最早想的是：怎么让AI更好地读这些PDF？
后来慢慢变成：我是不是应该先把法规本身整理成机器真正能使用的结构，再考虑AI？

这两个方向差别截然不同。

## Regulatory Knowledge Platform就是这么来的

当时其实不知道应该给这个东西叫什么。Regulatory Database感觉太像纯数据库，Knowledge Graph又好像只强调关系，DORA Explorer又把范围说小了。后来才慢慢用了现在这个名字：**Regulatory Knowledge Platform**。

名字听起来有一点大，不过我真正想要的东西确实已经不只是一个法规阅读器了。大概希望一份法规进来以后，至少能够变成这样：

```text
Regulation (EU) 2022/2554
│
├── Metadata
│   ├── CELEX
│   ├── ELI
│   ├── Date
│   ├── Status
│   └── Legal Type
│
├── Recitals
├── Chapters
│   └── Articles
│       └── Paragraphs
├── Annexes
│
└── Relationships
    ├── amends
    ├── supplements
    ├── implements
    └── references
```

以后打开DORA，可以直接看到Article；从Article跳回正式法律来源；看到一份Level 2 Regulation的时候，知道它为什么存在、法律基础是什么；从一份法规跳到相关法规。再往后接Search、RAG或者Agent的时候，它们也不用每次从一堆陌生PDF开始重新认识别和理解。

两种思路其实差很多：
之前的：
```text
一堆PDF
   ↓
切Chunk
   ↓
Embedding
   ↓
AI自己判断剩下的东西
```

和现在：

```text
Official Regulatory Sources
   ↓
Structured Regulatory Knowledge
   ↓
Search / Graph / RAG / Agent
```

第一种当然开发起来快，其实也就是初代Rag，基本上还不如直接去查GPT。第二种麻烦得多，但至少系统知道自己处理的到底是什么。

当我开始做法规结构化以后，新的麻烦马上又来了。

EUR-Lex有HTML、XML和各种metadata，CELLAR里面还有RDF。不同来源长得都不一样。如果每个下游模块都自己理解一次这些原始格式，那以后Search写一套、Graph写一套、RAG又写一套，项目估计很快就没法看了。

所以后来中间又多出来一层东西：

**Parser → Canonical → DocumentView**

我第一次看到Canonical这个词的时候很难很难理解，又是架构里面那种听起来特别唬人的单词。我粗浅的理解，就是文字结构上的归一化，当然，不是那么精确，但这样好理解点，拿EUR-Lex来说就好理解多了。

EUR-Lex给我的东西，不等于我系统里应该直接存的东西

刚开始我想得特别简单，EUR-Lex已经把法规放在那里了，那直接抓下来不就行了吗。网页上Article、Chapter、正文都排得好好的，HTML能看，XML也有，CELLAR里面还有metadata和RDF。问题是，这些东西虽然都在描述同一部法规，但长得完全不一样。有些信息在HTML里面很好找，有些东西XML更稳定，有些法律关系反而在RDF里面更清楚。

如果直接让后面的Search、Graph、RAG和前端各自去理解这些原始数据，项目很快就会变得很乱。Search自己认一遍Article，Graph再认一遍Legal Resource，RAG又用一套我自己清洗过的JSON，前端为了展示还得再写一遍。最麻烦的不是代码多，而是同一个东西可能在不同模块里面被理解成不同的东西。Article在这里是Article，在另外一个模块里面可能就只剩一段text了。以后上游字段一变，我也不知道到底要改几个地方。

所以后来我把这件事硬拆成三层。Parser负责把外面的东西读进来，Canonical负责规定进来以后它到底是什么，DocumentView再决定最后给人怎么看。

Parser其实最好理解。EUR-Lex给我XML，那Parser就把CELEX、Title、Chapter、Article、Paragraph、Recital、Annex这些能明确识别出来的东西拿出来。如果换成HTML，就换另外一个Parser。这里我后来越来越坚持一个原则：Parser不要太聪明。 刚开始我特别容易看到一段法规以后，顺手再判断一下这是不是obligation、是不是exception、属于哪个risk topic，甚至想顺便把chunk都切了。写的时候觉得很爽，一个function什么都干，过两个月基本不敢碰。现在Parser尽量只处理确定的东西。Article 5就是Article 5，CELEX就是CELEX，这段属于Chapter II，就把这个结构留下来。至于它到底是不是Access Management相关义务，已经不是Parser应该决定的事情。

Canonical才是我真正绕了很久的地方

Parser把数据拿出来以后，最直接的办法当然是直接存。比如一个Article存成number、title、text，看起来完全够用了。问题是，今天EUR-Lex XML里面这个字段叫一个名字，明天HTML里面可能叫另外一个名字，再换一个来源，甚至可能根本没有同样的字段结构。总不能让整个系统知道外面所有网站是怎么设计的。

所以Canonical存在的意义就是：外面的数据我管不了，但进了我的系统以后，得按我的规矩来。

这里真正重要的其实不是字段怎么命名，而是系统开始有自己稳定的一套法规结构。以前是EUR-Lex怎么给，我就怎么用；现在EUR-Lex只是source，进来以后先变成我自己的Canonical Model。以后哪怕换一个数据源，理论上也只是前面的Parser变化，后面的Search、Graph、RAG和API不应该全部跟着改。

我特别喜欢对系统解耦，可能跟大学时无数次的背诵：高内聚低耦合有关。

当然Canonical也特别容易越做越大。什么都想往里面放，最后它自己又会变成一个垃圾桶。所以我现在反而比较保守。CELEX这种确定事实可以放，Article属于哪个Chapter可以放，某个Regulation正式supplement另外一份Regulation，如果官方关系能够确认，也可以放。但“这条Article主要对应Access Management”就不一样了，这可能是人工分类，也可能是AI提取，也可能只是我自己为了业务方便做的映射。这种东西和法规原始事实最好不要混成一层。

我哦的Platform最重要的一件事就是把这条线守住：法规本身是什么，和我后来怎么理解它，是两回事。

那DocumentView又是干嘛的

做到Canonical以后，还需要考虑可视化，毕竟我的用户都是有法律需求的工作者。

系统里面可能关心的是id、parent_id、source_ref、text_blocks这些东西，但人看法规的时候还是希望看到熟悉的结构：Chapter II、Article 5、Governance and organisation，然后下面一段一段正文。用户也不会关心我的Canonical Model怎么设计，他只想点开Article 5，知道它属于哪个Chapter，前后是什么条款，正式EUR-Lex链接在哪里，跟哪些法规有关系。

所以DocumentView其实就是把底层结构重新组织成人能看的样子。听起来有点搞笑，我先把法规拆开，拆成Chapter、Article、Paragraph，然后为了给用户看，又重新把它拼回来。但这个“拆了再拼”其实很有意义，因为底层事实可以统一，外面的View可以不一样。

普通用户看到的是正常法规阅读页面，Graph看到的是Legal Resource之间的关系，RAG拿到的可能是Article正文加父章节、来源和相关metadata。同一份法规，不同模块需要的样子本来就不一样。我以前总觉得一份数据应该从头传到尾，越少转换越好。现在可以确定，底层统一，展示层根据用途变化，长期看更省事。

这个设计后来对RAG影响也挺大。最开始的RAG就是很传统的PDF转文本、切chunk、embedding、vector search。问题是chunk一切，法规原来的结构特别容易丢。模型最后拿到一句“The management body shall define, approve, oversee...”当然也能回答问题，但如果同时知道这是Regulation (EU) 2022/2554、Article 5、属于哪个Chapter、正式source是什么，意义完全不一样。

所以决不能把chunk理解成“500个token的一段文字”。Chunk只是检索单位，它背后还应该知道自己来自哪个Article、哪个Chapter、哪一部法规、哪个正式来源。这样最后生成答案的时候，我才能尽量确认：引用的不是某个PDF里面随机切下来的一句话，而是一段有明确法律位置的evidence。

这其实又回到了上一篇我一直强调的那个问题：我要做的是grounded evidence，不是让LLM每次重新猜。

当然做到这里也没有完事。Canonical里面到底要不要把Paragraph单独做成Node，Recital和Article是不是同一种结构，多语言怎么处理，consolidated version怎么存，amendment以后旧版本怎么办，这些问题随便一个都能继续折腾很久。

如果一开始全部想解决，这个Platform估计现在还停留在画架构图。

所以我后来砍掉了很多东西。先保证法规来源找得到，身份确定，Article结构不丢，上下文能恢复，关系能够追溯，够Search和RAG正常用就行。其他复杂问题以后真的需要再加。

最终，把EUR-Lex和CELLAR里面那些relationship真正拿出来以后就能清晰的看到，树只是最简单的一层。法规之间还有amends、supplements、implements、references，一份法规可以同时连到很多其他法律资源。

Platform已经开始长得像一个Knowledge Graph了，开心！