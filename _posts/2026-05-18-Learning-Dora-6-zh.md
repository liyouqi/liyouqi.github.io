---
title: "Regulatory Knowledge Platform 02｜把DORA画成一张Graph以后，才能看懂这些法规之间到底什么关系"

date: 2026-05-18
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
  - RegTech
---

我开始把EUR-Lex里面的法规拆成结构化数据。中间当然有Parser、Canonical这些东西，这些不是我做这个项目最有意思的部分。Parser无非就是把EUR-Lex的XML、HTML、metadata解析出来，Canonical再把不同来源统一成我自己的格式。真正让我开始觉得这个Platform有意思，是把CELLAR和EUR-Lex里面的relationship也拉进来以后。

这时候DORA突然不再是一堆文件了。

之前Level 1、Level 2、Level 3这个树，只是我为了学习自己画出来的，我用这个结构理解DORA：

```text
Level 1
   ↓
Level 2
   ↓
Level 3
```
刚开始学习特别有用，我现在也还会这么解释。但做Graph以后发现，它其实只是一个非常粗的抽象，真实的法规关系完全不是这么整齐的一棵树。

比如Regulation (EU) 2022/2554是DORA主法规，但下面的Commission Delegated Regulations不是简单的“child document”。有的在supplement主法规某些Article，有的进一步规定某个要求怎么执行，有些法律资源之间还有amendment、reference以及其他关系。Directive (EU) 2022/2556就更明显，它根本不是2554下面的Level 2，而是在修改原有金融行业的一批Directives，让原来的监管体系和DORA衔接起来。

所以真正的数据更接近：

```text
2022/2554 ──────→ 2024/1774
    │
    ├──────────→ other Level 2 acts
    │
    ├──────────→ related Directives
    │
    └──────────→ referenced legal resources
```

而且每条线还不是同一种意思。

做到这里我第一次比较明显地感觉到：法规天然适合Graph。也是我在法国读书时特别去认真做过的知识图谱，只是当时在学校里玩的都是demo相关，现在有了一个十分明确且有意义的应用场景。

法律资源本来就不是按照文件夹组织的。一个resource可以同时和很多其他resource发生不同类型的关系，这就是很标准的many-to-many network。

第一版Graph其实很丑，而且基本没什么用。把法规全部作为Node，把CELLAR里面的关系导进去，GraphDB一跑，确实一下就出来一大片节点和边。看着特别牛。

然并卵~~

然后点开一条Edge：

> Legal resource amends legal resource

再点一条：

> Legal resource cites legal resource

再点几个，还是差不多。

突然发现这个Graph除了好看，好像也没什么用。

因为真正的问题从来不是：

> A和B之间有没有一条线？

而是：

 **为什么有这条线？这条线对我理解DORA有什么意义？**

比如一个做ICT Risk的人打开2022/2554，他不是为了欣赏这个Node有18条Edge。他真正想知道的是：哪些Level 2法规在继续细化ICT risk management？这份Delegated Regulation的legal basis是什么？某个Article要求我做一件事情以后，具体细节在哪里继续展开？一份法规被另一份法规修改以后，到底修改的是什么？如果Graph回答不了这些问题，那它就是一个漂亮的蜘蛛网。

这个坑对我影响挺大的。以前学习Knowledge Graph总觉得Node和Edge搞出来就差不多完成一半了。现在觉得真正麻烦的根本不是“连起来”，而是Edge到底有没有语义。

另外，Node到底应该做到多细，也折腾了很久，最简单的做法当然是一份法规一个Node。

比如：

```text
Regulation (EU) 2022/2554
Commission Delegated Regulation (EU) 2024/1774
Directive (EU) 2022/2556
```

这样做很好理解，而且EUR-Lex本身就有CELEX这种非常适合做唯一标识的东西。所以第一层Graph我现在还是以Legal Resource作为最稳定的Node。

问题马上就来了：Article要不要也是Node？

如果Article不是Node，那很多真正有价值的关系只能停留在整部法规层面。比如我只能知道2024/1774和2022/2554有关，却不能进一步知道，它到底是在细化主法规里面哪一部分要求。

但如果每一个Article、Paragraph甚至Recital都变成Node，Graph规模马上会膨胀，而且很多节点其实没有必要单独存在。Paragraph全部拆出来以后，一部法规就可以变成几百个Node，再导入几十份法规，很快就会变成一个巨大Graph。理论上当然很完整，实际用起来未必更好。

所以我后来没有追求“所有东西全部Node化”。

现在更倾向于分层处理：Legal Resource是肯定存在的核心Node，Article是很有价值的结构Node，因为法律引用和RAG evidence经常最后需要落到Article。至于Paragraph、Recital这些要不要继续做成独立Node，则看后面的实际需求，不为了Graph看起来完整就全部拆。

所以强迫症的我必须避免“完整建模”，现在务实的来讲，最需要考虑的是这个Node到底会不会被查询。

如果一个Node除了让Graph节点数量从200变成20,000以外没有实际作用，我其实可以先不加。

## Edge比Node更重要

真正开始整理Edge以后，这个Graph才慢慢从“可视化”变成“知识”。

最开始我比较关心的关系包括amends、supplements、implements、references这一类。名字本身不是最重要，重要的是把不同法律关系分开，而不是全部变成一个万能的`RELATED_TO`。

假如最后Graph里面全是：

```text
A ── RELATED_TO ──→ B
B ── RELATED_TO ──→ C
C ── RELATED_TO ──→ D
```

那其实等于什么都没说，这点跟我以前做普通network analysis还挺不一样。社交网络里面，一条edge有时候只需要代表“认识”或者“发生过交易”就已经够用了。但法规不一样，不同Edge的法律含义差别很大。

`amends`意味着修改已有法律资源。

`supplements`意味着在已有框架下面继续补充要求。

`references`可能只是正文里面提到了另外一份法律资源。

这些关系如果混在一起，我后面做任何分析都会有问题。

例如用户以后问：

> DORA Article 6下面关于ICT risk management framework还有哪些更详细要求？

如果Graph只有`RELATED_TO`，系统只能告诉我“这些东西有关系”，废话，我真正希望它能够沿着更明确的legal relationship继续往下找到对应的Level 2资源，再把真正相关的Article交给RAG，Graph就不是用来画图，而是参与检索。

我还给关系分了“官方的”和“我自己推出来的”，这是后来特别重要的一点。EUR-Lex / CELLAR能够给我一些官方metadata和法律关系，这种东西我愿意直接作为Platform的fact。它来自哪里，CELEX是什么，什么resource修改了什么resource，这些都有来源。但还有大量关系是我自己很想要、官方Graph却不会直接替我做好的。

比如：

> Article X主要涉及Access Management。

> Article Y和ICT Third-Party Risk有关。

> 这个obligation最终可能对应某个Control Objective。

这些关系当然也非常有价值，甚至对真正的Risk Copilot更有用。但它们已经不是和`amends`同一类东西了。有的是人工mapping，有的是规则产生。有的以后可能是LLM提取。

所以我不想把它们全部混进同一个“事实层”，不然Graph看起来越来越丰富，但慢慢就会出现一个特别危险的问题，我自己不知道某条Edge到底是欧盟官方告诉我的，还是三个月前某个模型猜出来的。

对普通推荐系统可能问题没这么大，但我做的是Regulatory Knowledge。如果以后用户点开一条关系问：

> Why?

必须能回答它从哪里来的。所以现在更想把Graph理解成不止一种关系层：Official legal relationships是底座。
Curated / derived relationships可以在上面增加，但要知道是谁产生的、依据是什么。以后LLM生成的关系也不是不能进Graph，但必须带provenance，不能生成以后直接冒充事实。这其实和我做RAG时一直强调citation是同一个逻辑。

## Article-Level Graph开始让这个东西具备了痛点解决能力

只做到Legal Resource之间的Graph，其实已经能把DORA整个法律体系看清不少。但是真正需要解决的问题，最后还是需要往Article里面走。
因为用户不会总问：

> 2024/1774和2022/2554是什么关系？

更常见的问题是：

> DORA对ICT Risk Management Framework到底要求什么？

> 哪些条款明确要求management body参与？

> Access Management相关要求散落在哪些Article里面？

这种问题最终都不是document-level。

所以我后来越来越希望Graph既能保留法规之间的宏观关系，又能往下落到Article。

大概就是：

```text
Legal Resource
    ↓ contains
Article
    ↓ references / related legal basis
Article / Legal Resource
```

这样我从DORA主法规某一个Article出发，可以先知道它属于哪部法规，再找到跟它相关的其他法律资源，然后继续进入对应Article。这时候Graph和RAG的结合，普通Vector Search是从文字相似度出发，raph提供的是另外一种完全不同的信息：**这段文字在法律体系里面和谁有明确关系。**

两个东西不是互相替代，rag做不到。比如用户问一个很具体的ICT risk问题，Vector Search可能首先找到最相似的Article；Graph则可以继续告诉系统，这个Article还有哪些正式Level 2资源在supplement它，或者它附近还有哪些明确引用关系，然后再让RAG去读真正的文本。这比把所有法规全部塞进一个Vector DB，然后希望embedding自己理解完整法律体系，要靠谱得多。


最开始我特别喜欢Graph的可视化，因为装逼啊。节点一点开，周围法律关系全部展开，再点一个节点继续展开，确实挺爽。做成类似Google Maps那种法规地图，我到现在都觉得很有意思。给公司里一个律师看了后，获得了他的认同。

比如从一份Regulation往下追所有正式补充它的Level 2法规；查看某个legal resource修改了哪些既有法律；从Article进入相关法规；反过来看一份Delegated Regulation是基于哪一个DORA要求出现的；以后再加上业务层关系，还可以从Regulation一路走到Obligation、Risk、Control Objective，甚至Metric。



## 到这里算是实现了这个Platform的意义
整个东西业务流程最后形成的是：官方法规先进入Platform，Platform必须保存能够确定的法律事实、结构和关系；RAG在这个基础上检索和回答；Agent以后再去处理Risk、Control和具体业务动作。

绝不让大模型重新发明事实。

我的目标也不是仅仅做一个法规网站，更像是在给后面的RAG和Risk Copilot先铺一张法规地图。我不想为了做一张“超级知识图谱”，把所有东西全部塞进一个Graph里面。那样看起来特别宏大，最后没人知道哪个东西属于法规，哪个属于公司自己的control framework。所以现在Platform的Graph我还是希望主要守住法规事实这一层，先把法律世界本身的关系搞清楚，严格的套住缰绳，也就是harness，后面的AI再基于它工作。