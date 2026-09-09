---
title: "RAG，其实依旧是工程问题"
date: 2026-09-09
categories:
- AI
tags:
- RAG
- Regulatory RAG
- DORA
- RegTech
- LLM
- Hybrid Search
- Rerank
- Query Planner
- Legal AI
- Fintech
---

从最开始做RAG说起。

现在有一个法规知识平台成型了。最早主要做DORA，后面又把GDPR和AML的一部分官方材料接进来了。最初非常简单：现在ChatGPT都这么强了，那我把法规丢进去做个RAG，不就可以让用户直接问问题了吗？比如用户问：

DORA对ICT风险管理框架的最低要求是什么？哪些条款明确要求management body参与和年度review？

最开始我觉得这个东西应该不难。Parser把监管PDF解析出来，按条款切chunk，做embedding，放进vector index。用户提问以后vector search找Top K，把相关内容丢给大模型，然后让模型“只能根据context回答，并且给出引用”。

网上99%的RAG教程差不多都是这个套路。第一版确实很快就跑起来了，而且第一次看到AI真的能从法规里找东西回答，还是挺爽的。问“DORA对ICT风险管理有什么要求”，它能找到Article 5、Article 6；问ICT third-party risk，它也能找到Article 28附近的内容。看上去已经能用了。

然后很快就发现，不对。最大的问题不是它完全胡说，而是它经常“差不多对”。

这在普通知识库可能还能接受，在法规里就不可以接受了。比如法规明明列了10项要求，AI最后给你总结成6项；原文写的是shall consider，AI回答的时候变成了must implement；某个要求其实只适用于特定financial entity，生成答案以后scope没了；还有一种更隐蔽，答案本身没什么问题，后面也跟着Article引用，但仔细点进去看，会发现这个Article只支持前半句话，后半句是模型自己顺着逻辑推出来的。这种答案最麻烦，因为不是一眼就能看出错，反而写得特别专业。

接下来我就在这个东西上不断加东西。做到现在，整个流程已经变成了Query Planner、claim拆解、BM25和vector hybrid search、结构和字段信号做的rerank、metadata过滤、claim-bound evidence、最后再生成答案和citation。听起来已经挺完整了，但实际做下来，真正有效的东西和一开始设计的差别不小 

## 优化1：法规不能像普通文档一样500 token一刀切。法规是天然的高度chunk化的结构化文本。

卖网课的那种教程里基本都是这SB玩法。

最开始当然也试过固定长度chunk。这个方法没有什么技术难度，比如800 token一个chunk，再留100 token overlap。对于普通企业文档其实完全可以跑，但法规绝不可以这样。

```text
Article 5
Governance and organisation

1. The management body shall...
2. The management body shall...
3. Financial entities shall...
   (a) ...
   (b) ...
   (c) ...
```

如果刚好在token边界切开，很可能paragraph 3留在一个chunk里，后面的(a)(b)(c)跑到另外一个chunk。用户问的时候检索到了“Financial entities shall...”，但是它真正列出来的具体要求没有一起回来。这时候AI也不算胡说，因为你给它的context本身就缺了。

这一点我特别清醒，虽然法律那玩意太复杂很看难看懂，但我从一开始设计就明白，法规本来就已经有非常强的结构，为什么我要先把这个结构打烂，然后再让embedding帮我猜？现在基本是按法规自己的结构来：

```text
Document → Chapter → Section → Article → Paragraph → Point → Sub-point
```

chunk不是单纯保存一段text，还会把它属于哪个regulation、哪个Article、哪个paragraph以及父级结构一起保存下来。比如大概会有：

```json
{"document":"Regulation (EU) 2022/2554","celex":"32022R2554","article":"5","paragraph":"2","title":"Governance and organisation","text":"..."}
```

后面我又专门把官方原件、不同Parser的输出和系统里的统一条款结构分开了。当时还觉得自己是不是搞复杂了，后来加新法规的时候才发现这东西其实躲不掉。甚至不同监管文件自己的排版都不一样，Parser解析到什么，不代表系统里面就应该直接把它当成最终结构。最后还是要把Article、Paragraph、Annex这些东西统一下来。

当然现在这个地方也没有做到“随便扔一部法规进去就完事”。DORA、GDPR和AML材料走的是同一条构建流水线，但每加一种来源，还是会碰到解析、版本和条款映射的问题。

所以我现在对chunking的理解已经跟最开始完全不一样了。普通RAG在考虑“这一块应该多大”。法规RAG还得先考虑“这一块在法律结构里面到底是什么”。

## 优化2：纯vector search绝对滴不够用，这只是常识

纯向量检索。比如用户问：

 What are the responsibilities of the management body under DORA?

vector search当然能找到很多相似内容，因为DORA里面management、governance、ICT risk、responsibility这些词到处都有。问题是用户真正要找的可能就是Article 5，而vector结果里经常会混进一堆语义相关但是并不直接回答问题的段落。

法规还有一个特点，就是大量专有名词、法规编号和固定术语。

```text
Article 31
Regulation (EU) 2024/1774
JC/GL/2024/36
critical or important function
major ICT-related incident
ICT third-party service provider
```

这种情况下BM25反而特别好用。所以后面就直接做Hybrid Search，BM25和Vector两路召回，然后再做融合和rerank。

```text
Question → BM25 + Vector → Merge → Rerank → Evidence
```

这里的rerank目前也不是再接一个很贵的模型。它先用条款角色、标题词命中、例外条款这些可审计的信号，把候选重新排一下。比如用户明明在问definition，就优先定义条款；没在问exception，也别让例外条款莫名其妙冲到前面。

网上有些文章会写：“加了rerank以后准确率从68%直接到了79%”。我这里没有这种数据，我也不想现在硬编一个。实际感觉就是：以前Top结果经常有一两条很像但没回答问题的内容，加了以后这种情况明显少了。尤其在多个Article都讨论同一个主题的时候，效果比较明显。这就够了。

## 优化3：真正麻烦的不是检索，是一个问题可能根本不是一个问题

到了这里我终于碰到问题了，不管你Hybrid调得多漂亮，结果还是不好。

```text
1. ICT risk management framework最低要求是什么？
2. management body参与的法律依据是什么？
3. annual review要求在哪里？
```

以前一次检索Top 10，看上去十条都“相关”，但可能第一个问题找了六条证据，第二个问题三条，第三个问题一条都没有。然后LLM拿着这十条东西开始写，一般也能写出第三个问题的答案。这才是最离谱的。

所以v0.4开始加Query Planner，让模型先理解问题，再决定到底应该查什么。复杂问题会先拆claim，然后每个claim分别retrieval。现在每个required claim都有自己的coverage状态，证据不够就明确说不够，而不是把没找到包装成“法规没有要求”。

当然代价也很明显：慢的主要还是LLM，不是BM25、vector index或者rerank。一次完整的planned answer，先得理解问题、拆claim、分别检索、处理证据、再生成。当前本地端到端实测大概15到28秒，已经比最早那种一分钟级等待好很多，但简单问题还是会觉得慢。

用户问一句“ What is CTPP? ”，系统如果也在那里认真Planner半天，拆claim，查证据，rerank，然后过很久才告诉你Critical ICT Third-Party Provider，用户早跑了。

所以这个阶段我反而开始做减法。复杂问题需要Planner，简单问题不一定需要同样重的策略。RAG做到后面很容易有一个毛病：为了搞定最难的10% case，把剩下90%的问题也搞得特别复杂。我现在越来越不喜欢这种设计。

## 优化4：后来发现Query Planner还不够，先得知道用户问的是什么“类型”

最近我又碰到一个问题，关于公司中最常用反洗钱法：

Which customer due diligence measures are required under the Fourth Anti-Money Laundering Directive?

系统其实已经能找到正确条文了，证据也没问题。但是最后答案还是让我觉得不对。原因很简单，这是一个enumeration question。

法律原文可能明确写了(a)(b)(c)(d)一堆要求，用户实际上是在问“到底有哪些？”。LLM最喜欢做的一件事是什么？总结。十项要求，它给你归纳成六项。六项内容都对，你甚至不能说它回答错了。但法规里面那些限定词、条件和分项结构已经被吃掉了。如果是普通summary没什么，用户问“有哪些法定要求”，那就完全不是一回事。

所以我现在不只是把用户问题改写成更好的search query，还会把它按检索意图处理。现在已经有definition、minimum requirements、obligation、scope、exception、threshold、procedure、compound这些方向；“有哪些要求”这类问题，检索层会尽量把父条款和完整有序子项一起带回来，不让几段主题相似的碎片冒充完整答案。definition就优先找definitions，不要给它拉一堆评论文章。scope不能只搜scope，还要考虑exception。threshold里面的数字、时间、条件必须单独检查。compound就拆claim。

这个东西现在也还没有全部做完，但我觉得它比继续换embedding模型有意思多了。因为到了这个阶段，问题已经不是“搜得准不准”。 你知不知道用户到底在问哪一种法律问题？

## 优化5：Citation不是在答案后面挂几个Article编号

我这个项目从第一天很就加了citation，因为我一直觉得法规RAG如果没有出处，那基本没什么意义。最早也很简单，答案后面：

```text
[1] Article 5
[2] Article 6
[3] Article 28
```

后来引用里带上可信的document、条款坐标、原文摘录和官方来源链接。本来以为这就已经挺好了，后来还是出问题。 比如AI回答：

> The management body must approve the ICT risk framework, review X annually, and ensure Y.

后面跟一个Article 5，用户看上去觉得有理有据。但是如果真的点进去看，Article 5可能只支持第一句话。annual review可能在另外一条，Y可能干脆就是模型根据上下文推出来的。 所以“答案有引用”和“每个结论有证据”是两回事。现在我更倾向做claim-bound：

```text
Claim 1 → Evidence A
Claim 2 → Evidence B
Claim 3 → Evidence C
```

每个claim得知道自己到底靠什么证据活着。没有证据就不要混进去。这个地方我觉得是普通RAG和法规RAG区别比较大的一点。LLM非常擅长用三个正确事实推导出第四个“听起来也特别合理”的事实。日常聊天里没什么，Compliance里就很麻烦。

## 优化6：Metadata后来变成了法规系统里面非常重要的东西

最开始加metadata主要是为了知道一个chunk来自哪里，后来慢慢发现它还可以拿来限制检索。比如用户明确问：

> According to Regulation (EU) 2024/1774...

那我为什么要把整个corpus全搜一遍？直接：

```text
document = 2024/1774
```

就完了。如果问DORA，也不要莫名其妙把GDPR或者AML里一个意思很像的条款塞进答案。现在metadata里面除了Article、document、CELEX这些基本信息，还有framework、jurisdiction、document type、法律效力、当前版本、适用日期、官方来源这些。scope过滤也是所有Retriever共享的，不是BM25筛一套，vector又筛另一套。

这时候metadata filtering已经不是企业知识库里那种language = java、type = guide，而有点接近法规的applicability问题了：哪一部法？哪个版本？什么时间有效？这个entity到底适不适用？

这些东西后面肯定还要继续搞。目前有些语料范围和版本状态已经管起来了，但真正落到具体机构画像和每条义务的适用判断，离“全部解决”还很远。

## 优化7：最后生成答案的时候，模型反而不能太聪明

这个挺反直觉，对吧。最开始选LLM肯定都想选更聪明的，模型越强，reasoning越强，答案越好。后来发现法规场景不一定。

模型越聪明，它有时候越喜欢“帮你”。原文写10条，它觉得太长，帮你合并一下；原文写shall consider，它觉得这样不够直接，改成must implement；原文只说某种情况下适用，它觉得根据逻辑其他场景应该也差不多，然后顺手扩大scope。

这些事情模型不是在胡编，它甚至是在“帮助你”。但是我不要它帮。所以现在answer generation这块，我越来越倾向于限制它。告诉它这个claim对应哪些evidence，这些evidence支持到什么程度，数字不能改，exception不能丢，scope不能扩大，没有证据就直接说没有证据。生成器也不能自己编citation，只能从这次检索回来的evidence里选。

以前我理解LLM的作用是：把检索出来的材料理解以后，给我一个聪明的答案。

现实情况：证据我已经尽量找好了，你别乱发挥，帮我把它整理成人能读的话。这个细节很重要，都是我一步一步摸索搞出来的。

## 最后一个优化：Evaluation，这个我现在反而觉得最重要，但是也最没做好

做到这个阶段以后，最大的问题就是你根本不知道自己到底有没有变好。今天改了一版prompt，测几个问题，感觉不错。明天把Hybrid权重调一下，再测几个，感觉也不错。过几天换了Planner，又觉得复杂问题好像更好了。全是“感觉”。

所以我现在也开始做Golden Test和evaluation。但是我不太想只搞correct、partial、wrong。法规问题这样评分太粗了。

比如原文10项义务，模型回答8项，8项全部正确。普通LLM Judge很可能给correct，但是我会觉得这是incomplete。又比如原文写at least annually，模型回答regularly，大概意思对不对？对。法律上是不是一样？肯定不是。

现在已经有一套冻结题集和自动检查，至少会检查预期状态、必需evidence有没有被检索和引用。但faithfulness、completeness、citation到底是不是真的支持答案，还是得靠人工复核，不能让另一个LLM Judge给个分就当真。这套东西还在继续做，在固定的题集检索集里

纯Vector的 Recall@5 是80%，MRR@5 是59.8%。
加上BM25做RRF Hybrid以后，Recall@5到了86.7%，MRR@5到了75.9%。
也就是说，Hybrid相对纯Vector，Top 5找回率提升了6.7个百分点，排序质量提升了16.1个百分点。
按标题和法律结构切分、metadata过滤、意图分析、rerank、Planner这些东西现在都已经在用，但我没有拿同一套冻结题集把它们一个一个关掉重跑，所以不想硬写“68%、79%、91%”这种看起来很漂亮的数字。
增强前，extractive和Private AI两条路径的必需证据召回都是58.9%；后来把scope-local BM25和检索增强补上，extractive到了100%，历史Private AI运行是96.2%。

Citation不直接提高检索准确率。它解决的是另一件事：让用户知道这句话到底是从哪一条来的，AI有没有把原文意思扩大，某个claim到底有没有证据。
95%够用吗？对法规RAG来说，我觉得不是这么算。

因为剩下的部分不一定是“模型没答对”。有些是语料里确实没有，有些是问题本身缺scope，有些是法规要求必须结合机构类型、时间点和具体事实判断。系统在这些时候正确地说“证据不足”或者“需要澄清”，本身就是正确行为。

我的一点体会。
搞了一个月RAG，最大的感悟是：RAG的瓶颈不在模型，在工程，在工程，在工程，在工程，工程，工程，工程，工程，工程，工程，工程！

用GPT还是DeepSeek、用哪个embedding模型，这些当然有影响，但真正拉开差距的，往往还是切块有没有保住法律结构、检索能不能把正确条款找回来、scope有没有过滤对、每个结论有没有真的绑到证据上。
很多人一上来就折腾prompt工程，调temperature、改system prompt。
但如果你喂给模型的context本身就是垃圾，prompt写得再花哨也没用。
Trash in, trash out。
C'est rag



