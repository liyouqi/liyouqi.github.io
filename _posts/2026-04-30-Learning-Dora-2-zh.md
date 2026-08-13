---
title: "学习DORA 02｜我只是想下载一部DORA，为什么最后找到了十几份文件"
date: 2026-04-28
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

layout: single
author_profile: true
read_time: true
comments: false
share: true
---


我一开始以为DORA就是一部法规，对应一个PDF。

领导让我学DORA，我想得也很简单：先把法案下载下来，丢给AI，然后从第一页开始慢慢啃。

结果光是下载文件这一步，就把我绕晕了。

先是在EUR-Lex找到了 **Regulation (EU) 2022/2554**，我以为任务已经完成了。后来又冒出来一个 **Directive (EU) 2022/2556**。

继续往下找，又看到了：

- Commission Delegated Regulation
- Commission Implementing Regulation
- RTS
- ITS
- Guidelines
- Final Report
- Q&A
- Reporting tools
- CSSF Circular

更离谱的是，这些东西还不在同一个网站。

有的在EUR-Lex，有的在European Commission，有的在EBA，有的在ESMA和EIOPA。到了卢森堡本地执行，又要去CSSF。

感觉每个网站上都有DORA，但谁都不像那个唯一的“DORA官网”。

我一度怀疑是不是自己下载错了，或者漏掉了一个别人已经整理好的完整版压缩包。

后来才搞明白：DORA本来就不是一个PDF能够装完的东西。

它更像一棵从上往下不断展开的树。

最上面是主法规，先把整体框架定下来；下面再通过一系列法规和监管文件，把风险管理、事件报告、测试、供应商和监管报送这些具体问题分别展开。

为了方便理解，大致可以先分成三层：

```text
Level 1
总体法律框架
      ↓
Level 2
把具体要求、标准、阈值和模板展开
      ↓
Level 3
监管指引，帮助各国主管机关和金融机构统一理解
```

再往下，还有各国监管机构的本地执行要求。
对我来说，就是CSSF告诉卢森堡的银行：具体向谁报、通过什么渠道报、采用什么流程。

## 2554才是通常所说的DORA主法规

**Regulation (EU) 2022/2554**，就是通常所说的DORA主体。

它先把整个框架搭了起来，包括：
- ICT风险管理
- ICT事件管理和报告
- 数字运营韧性测试
- ICT第三方风险管理
- 关键ICT第三方服务商监督
- 管理层责任
- 监管与执法机制

所以我现在把2554理解成一个总纲。
但“总纲”不代表它只是讲一些正确但没什么用的原则。里面已经有很多金融机构必须履行的直接义务，只是大量操作细节还需要后面的文件继续补充。
例如，2554会告诉金融机构：

> 重大ICT事件需要向主管机关报告。

但它不可能在同一个条款里，把所有事件阈值、报告时限、表格字段和提交程序全部写完。

这些东西需要后面的法规继续展开。

## 那2556又是干什么的

**Directive (EU) 2022/2556**并不是另一部DORA，也不是2554的下半本。

欧盟原来已经有很多银行、支付、证券、基金和保险方面的法律。
DORA出来以后，这些旧法律中涉及ICT风险和运营韧性的部分也要跟着调整，不然很容易出现两套规则互相打架。
所以2556的作用，更像是修改和衔接原来的金融行业指令。
它修改了UCITS、Solvency II、AIFMD、CRD、BRRD、MiFID II、PSD2和IORP II等一系列既有指令，让它们能够和DORA的新框架接轨。

我现在觉得应该可以这么理解：

```text
2554：建立DORA的新框架

2556：修改原来的金融行业指令，让它们和DORA接轨
```

银行日常做DORA控制、风险评估、事件管理和第三方管理时，主要还是围绕2554和后面的具体法规。

2556更多是在处理：DORA出现之后，原有的金融监管体系应该怎么跟着调整。

## 为什么下面还要有十几份Level 2

因为2554不可能把所有操作细节都写完。

还是拿重大ICT事件报告举例。

主法规可以规定：

> 金融机构需要报告重大ICT事件。

但真正轮到银行执行，马上会出现一堆问题：

- 什么程度才算重大？
- 中断两个小时算不算？
- 应该在多长时间内报告？
- 是一次报完，还是分阶段报告？
- 初次报告的时候不知道根因怎么办？
- 需要填写哪些字段？
- 使用什么模板？
- 向哪个主管机关提交？

这些问题不能让每家银行各自发挥，也不能让每个国家自己搞一套完全不同的标准。

所以DORA授权欧盟委员会继续制定一系列 **delegated acts** 和 **implementing acts**，把具体标准和执行方式补充完整。

这就是Level 2。

简单理解：

```text
2554提出义务
      ↓
Level 2把义务变成更具体的标准、方法、阈值和模板
```

这里又会碰到两个经常出现的缩写：

- RTS：Regulatory Technical Standards**
- ITS：Implementing Technical Standards**

我目前比较粗糙的理解是：

RTS更偏向规定：

> 应该满足什么标准、考虑哪些因素、采用什么方法进行判断。

ITS更偏向规定：

> 应该怎样统一执行、使用什么模板、填写哪些字段、按照什么程序提交。

这个区别并不是在每份文件里都划得特别绝对，但暂时这样理解已经够用了。

例如，重大ICT事件这一块就被拆成了几份不同的法规：

- 一份负责定义事件分类和重大性标准
- 一份负责规定什么时候报告、分几次报告
- 一份负责规定具体使用什么表格和字段

所以这些文件并不是重复，而是在回答不同的问题。

## Final Report为什么不能直接当成正式法规

这也是我差点踩的一个坑。

在EBA、ESMA或者EIOPA网站上，经常能搜到类似这样的文件：

> Final Report on draft RTS on ICT Risk Management

标题又长又正式，PDF可能还有一两百页，看起来非常像最终法规。

但它其实还不是最终生效的Commission Regulation。

大概的过程是这样的：

```text
DORA授权制定某项技术标准
      ↓
EBA、ESMA和EIOPA联合起草
      ↓
发布Consultation Paper、draft RTS或Final Report
      ↓
欧盟委员会审查和正式采纳
      ↓
成为Commission Delegated Regulation
或者Commission Implementing Regulation
      ↓
在欧盟官方公报发布，并进入EUR-Lex
```

EBA、ESMA和EIOPA这三个机构合称 **ESAs**，也就是European Supervisory Authorities。

它们分别主要负责银行、证券和保险领域。

DORA覆盖的金融机构类型比较多，所以很多技术标准不是EBA一家制定，而是三家联合起草。

例如，ICT风险管理框架在草案阶段可能叫：

> Final Report on draft RTS on ICT Risk Management Framework

等欧盟委员会正式采纳以后，就会变成：

> Commission Delegated Regulation (EU) 2024/1774

所以以后如果同时看到：

```text
Final Report on draft RTS...
```

和：

```text
Commission Delegated Regulation (EU)...
```

我会把后者作为主要的正式法规依据。

前面的Final Report也不是没用。

事实上，它有时候比最终法规更容易读，因为里面会解释为什么要这样设计、公开咨询时收到了什么意见，以及最后为什么选择某一种方案。

但它不能代替最终法规。

不然很可能出现一个尴尬情况：研究了半天，研究的是草案，最终发布的版本已经发生了变化。

## Level 3又是什么

Level 3不会再像Level 2那样，最后变成一份Commission Delegated Regulation或者Commission Implementing Regulation。

它主要是ESAs发布的监管指引，例如 **Guidelines**。

它的目的不是重新制定一部法律，而是尽量让不同国家的监管机构和金融机构，对同一条DORA要求采用比较一致的理解和做法。

这次我下载的核心Level 3文件有两份：

- 关于重大ICT事件年度成本和损失计算的Guidelines
- 关于ESAs与各国主管机关开展关键ICT第三方监督合作的Guidelines

至于Q&A、Decisions、Final Reports和Reporting tools，它们也很重要，但不能全部简单地塞进Level 3。

例如Register of Information除了正式法规以外，还会有：

- Data dictionary
- Validation rules
- Reporting templates
- Filing rules
- Technical package
- Frequently asked questions

这些是为了让银行真的能把数据整理出来并提交上去，并不是又多出了一部新法规。

## 这几个网站到底分别是干什么的
绕了一大圈之后，我现在给自己定了一套最简单的规则。

### EUR-Lex：找正式法律原文
2554、2556以及正式发布的Level 2法规，都从这里下载。
如果某份文件到底是不是最终法律，我先看它有没有进入欧盟官方公报。
所以正式法规引用、内部控制矩阵和审计依据，最后还是要回到EUR-Lex。

### European Commission：检查Level 2目录

European Commission的网站会列出DORA下面已经制定的delegated acts和implementing acts。
这里更像一个目录。
它告诉我有哪些Level 2文件已经正式采纳，但具体法规正文通常还是会链接回EUR-Lex。

### ESMA或者EIOPA：看DORA整体地图

ESMA和EIOPA的DORA页面，会把Level 1、Level 2和Level 3文件按主题放在一起。
对人类相对友好一点。
我们之前就是顺着ESMA的DORA页面，把十几份核心文件下载完整的。

### EBA：看银行和报送相关材料

DORA当然不只管银行，但因为我现在就在银行工作，所以EBA的很多材料和我的日常工作更接近。

特别是：
- Register of Information
- Reporting tools
- Q&A
- 数据字典
- 校验规则
- 关键ICT供应商监督

但EBA并不是唯一的“DORA官网”。

### CSSF：看卢森堡具体怎么执行

欧盟法规告诉银行：

> 必须做什么。

CSSF则会进一步告诉卢森堡的金融机构：
- 向谁提交
- 通过eDesk还是其他渠道
- 使用什么程序
- 本地有哪些Circular
- 事件报告怎么操作
- Register of Information怎么提交

例如DORA要求重大ICT事件需要报告，但真正从卢森堡提交时，还要按照CSSF规定的本地流程进行。

所以这几个网站并不是在争夺“谁才是DORA官网”。

它们只是在不同层面做不同的事情。

我现在给自己记的版本是：

```text
正式法律：EUR-Lex

Level 2目录：European Commission

DORA整体地图：ESMA / EIOPA

银行和报送材料：EBA

卢森堡本地执行：CSSF
```

## 怎么快速判断自己正在读什么文件

以后再看到一份DORA文件，我准备先看标题。

如果标题是：

> Regulation (EU) 2022/2554

或者：

> Commission Delegated Regulation (EU) 2024/1774

而且能在EUR-Lex和欧盟官方公报里找到，那么它是正式法律文本。

如果标题是：

> Final Report on draft RTS...

那么它通常是ESAs提交的技术标准草案和解释材料，还不能代替最终的Commission Regulation。

如果标题里带有：

> Joint Guidelines  
> JC/GL/2024/34

那么它属于监管指引。

如果标题是：

> Circular CSSF...

那么它是卢森堡主管机关发布的本地监管文件。

如果标题是：

> Data Dictionary  
> Validation Rules  
> Reporting Package  
> Filing Rules

那么它通常是报送技术材料，并不是又多出了一部法律。

## 为什么DORA没办法只做成一个PDF

到这里，我终于明白为什么自己只是想下载一部DORA，最后却得到了十几份文件。

因为2554只负责把整栋楼的主体结构搭起来。

事件怎么分类、什么时候报告、使用什么模板、怎样管理供应商、怎样做TLPT、怎样维护Register of Information，这些具体房间还得由后面的法规分别建出来。

Guidelines、Q&A、技术工具和CSSF文件，则是在告诉银行这栋楼到底应该怎么使用。

一开始真的被绕晕了
主要是先把法规文件本身搞清楚。
不然以后很可能学了半天，连自己读的是正式法律、监管指引、草案还是报送说明都不知道。
目前我给自己的原则很简单：
法律结论回到EUR-Lex，具体实施问题再去找ESAs和CSSF。

## 记录一些比较重要的文件

- [Regulation (EU) 2022/2554 — EUR-Lex](https://eur-lex.europa.eu/eli/reg/2022/2554/oj/eng)
- [Directive (EU) 2022/2556 — EUR-Lex](https://eur-lex.europa.eu/eli/dir/2022/2556/oj/eng)
- [European Commission — Implementing and delegated acts under DORA](https://finance.ec.europa.eu/regulation-and-supervision/financial-services-legislation/implementing-and-delegated-acts/digital-operational-resilience-regulation_en)
- [ESMA — Digital Operational Resilience Act](https://www.esma.europa.eu/esmas-activities/digital-finance-and-innovation/digital-operational-resilience-act-dora)
- [EIOPA — Digital Operational Resilience Act](https://www.eiopa.europa.eu/digital-operational-resilience-act-dora_en)
- [EBA — Digital Operational Resilience Act](https://www.eba.europa.eu/activities/direct-supervision-and-oversight/digital-operational-resilience-act)
- [CSSF — ICT and cyber risk for DORA entities](https://www.cssf.lu/en/ict-and-cyber-risk-for-dora-entities/)