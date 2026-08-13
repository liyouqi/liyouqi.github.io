---

title: "学习DORA 03｜系统都外包给云厂商了，出了事到底算谁的？"
date: 2026-04-29
categories:

* Cybersecurity
  tags:
* Cybersecurity
* Fintech
* DORA
* ICT Risk
* Third Party Risk
* Cloud Security
* CTPP
* Digital Operational Resilience Act

layout: single
author_profile: true
read_time: true
comments: false
share: true
-----------

今天AI先问了我一个问题。

假设一家银行把整个网上银行系统都托管到了AWS。某一天AWS出了故障，网上银行跟着挂了几个小时。

这件事情到底算谁的责任？

我感觉是：**银行的**。因为，甲方采购的服务必须自己承担，不可能把责任推到乙方身上

银行自己选择了AWS，客户也是银行的客户，总不能系统一挂就跟监管说一句：

“这是AWS的问题，不关我的事”。

但继续往下学才发现，这个很直觉的答案，其实正好踩中了DORA里面一个很重要的逻辑：

服务可以外包，责任不能跟着一起外包，**但但但**也分情况

## DORA其实不只是管银行

上一篇把DORA那十几份文件搞明白之后，今天开始看它到底管哪些机构。

我本来以为这个问题也很简单。

DORA是金融法规，那不就是银行、券商、基金、保险这些。

结果Article 2拉出来一长串，真没招了（最近一个清华实习生的口头禅）

里面包括银行、支付机构、电子货币机构、投资公司、证券市场基础设施、基金管理公司、保险、养老金、信用评级机构、加密资产服务商、众筹平台等等。

甚至连ICT third-party service providers，也进入了整个DORA体系。

所以DORA其实不是一部“银行法”。

它更像是在金融行业外面画了一个很大的圈。

```text
银行
支付机构
证券和投资机构
基金
保险
养老金
Crypto
众筹
金融市场基础设施
        ↓
都越来越依赖ICT
        ↓
Cloud / SaaS / Data / Network / Outsourcing Provider
```

现在的金融机构已经很难脱离外部技术公司独立运行了。

银行可以买云，可以采购SaaS，可以把某些系统外包出去，甚至很多关键业务背后都可能依赖第三方。

于是问题就来了：

**如果真正提供技术服务的已经不是银行自己，出了事以后谁负责？**

## 系统可以放在AWS，责任还是银行自己的

这一点DORA其实写得很直接。

金融机构使用ICT第三方服务，并不会因此免除自己履行DORA义务的责任。

这个逻辑我觉得很好理解。

假设银行网上银行放在AWS：

```text
客户
 ↓
银行网上银行
 ↓
银行自己的应用
 ↓
AWS Cloud
 ↓
AWS下面可能还有其他技术依赖
```

AWS出了问题以后，AWS当然有自己的责任。

比如：

* 为什么服务中断；
* SLA有没有达到；
* 多久恢复；
* 有没有及时通知银行；
* 有没有提供足够的信息；
* 根因是什么；
* 后续怎么整改。

但银行要回答的问题完全不一样：

* 这次中断影响了多少客户？
* 哪些交易失败了？
* 有没有影响critical or important function？
* 是否达到重大ICT事件的标准？
* 要不要报告CSSF？
* 当初为什么选择这个供应商？
* 有没有做风险评估？
* 有没有替代方案？
* 如果AWS长期恢复不了，银行自己怎么办？

所以站在监管角度，AWS挂了并不能自动变成银行的免责理由。

反过来讲，既然这个供应商对你这么重要，之前为什么没有识别这个风险？

这也是我今天觉得特别重要的一句话：

> **Outsourcing the service does not outsource the responsibility.**

服务外包了，监管责任没有外包。

当然，这不是一个二极管的的答案，一定要么是你要么是我。AWS这种公司，不只是普通供应商。继续往下就碰到了一个之前经常看到、但一直没仔细搞懂的缩写：

**CTPP — Critical ICT Third-Party Provider**

关键ICT第三方服务商。

不是说一个供应商规模大，就自动变成CTPP。

它需要由EBA、ESMA和EIOPA这三个ESAs按照DORA规定的标准进行评估和正式指定。

为什么还要搞这么一套东西？

我想了一下，其实还是挺合理的。

假设只有一家银行使用某个小供应商：

```text
Bank A
  ↓
Provider X
```

Provider X挂了，主要影响Bank A。

这首先还是Bank A自己的third-party risk management问题。

但如果情况变成：

```text
Bank A ─┐
Bank B ─┤
Bank C ─┤
Bank D ─┤
Insurer ├────→ 某个大型Cloud Provider
Fund    ─┤
Broker  ─┘
```

那问题就完全不一样了。

如果欧洲几十家甚至更多金融机构的重要系统，都集中依赖少数几家大型技术公司，那么其中一家发生严重故障，影响的可能就不再是一家银行。

它可能直接影响整个金融体系的operational resilience。

这时候，单靠每家银行自己去管供应商显然不太够。

所以DORA又往上加了一层：

```text
EBA / ESMA / EIOPA
        ↓
   Lead Overseer
        ↓
      CTPP
        ↓
提供ICT服务
        ↓
大量金融机构
```

也就是说，一旦某个ICT供应商被正式认定为CTPP，就会进入DORA建立的欧盟级直接监督体系。

Lead Overseer可以对它的ICT风险管理、治理、事件处理、业务连续性等开展监督，还可以要求信息、调查和检查。

2025年11月，ESAs已经正式公布了第一批指定CTPP名单。

看了一眼名单，里面不少名字都非常熟。

### List of designated CTPPs


![List of designated Critical ICT Third-Party Providers under DORA](/assets/images/dora/critical-ICT-thirdparty-service.png)


这个名单对我来说还有一个挺有意思的地方。

以前看AWS、Google Cloud、Microsoft这些公司，首先想到的是云厂商、技术平台。

但从DORA的角度看，当大量金融机构的关键业务都建立在这些基础设施上以后，它们已经不只是普通IT供应商了。

它们本身也开始成为金融体系operational resilience的一部分。

## CTPP被欧盟监管了，银行是不是就轻松了？

也不是。

既然AWS这种CTPP已经有人直接监管，那银行是不是可以少管一点？

实际上这两套责任是并行的。

```text
欧盟监管机构
        ↓
监督CTPP本身是不是足够稳健

同时

银行
        ↓
继续管理自己使用这个CTPP产生的风险
```

银行仍然需要知道：

* 自己到底用了这个供应商哪些服务；
* 哪些业务依赖这些服务；
* 有没有critical or important function；
* 数据存在哪里；
* 有没有subcontractors；
* 集中度有多高；
* 服务挂了以后能撑多久；
* 有没有替代方案；
* 最后到底能不能退得出去。

所以CTPP oversight并不是欧盟帮银行把third-party risk management做了。

这是两件事。

一个是在看：

> 这家大型供应商会不会对整个欧洲金融体系造成系统性风险？

另一个是在看：

> 你这家银行自己为什么这么依赖它，又准备怎么管理这个风险？

这才是要理解的关健点
 另外一个我觉得比较合理的地方：DORA不是所有机构一刀切

Article 2把很多金融机构都装进了DORA。

那马上又有一个问题。

一家在几十个国家都有业务、几百上千个系统的大型银行，跟一家业务非常简单的小金融机构，总不能要求完全一样吧？比如我现在所在跨国中资行，在国内是巨无霸，但是在海外只在各国首都设置一个点，只提供一些对公业务而已。

DORA在Article 4里面专门写了一个 **Proportionality principle，比例原则**。

大意就是，在落实ICT风险管理这些要求时，要考虑机构自己的：

* Size
* Overall risk profile
* Nature of services
* Scale
* Complexity of services, activities and operations

也就是规模、整体风险情况，以及业务和运营本身的性质、规模和复杂程度。

我理解这不是说：

> 小机构可以少合规一点。

而是说：

> 要达到的监管目标是一致的，但达到这个目标的方法和控制复杂程度不一定完全一样。

为了让我自己好理解，我做了个对照。

**注意：下面这张表不是DORA官方分类，也不是监管机构规定的标准模板，是，我按照Article 4的比例原则做的理解示意。**

| 场景      | 大型跨境银行                          | 中型金融机构           | 规模较小、业务较简单的金融机构          |
| ------- | ------------------------------- | ---------------- | ------------------------ |
| 业务与系统   | 多国家、多实体、大量系统和复杂依赖               | 系统和业务数量相对有限      | 业务线较少、技术架构相对简单           |
| ICT治理   | 可能需要专门委员会、多个风险和控制职能             | 可以采用较精简的治理结构     | 组织可以更紧凑，但责任仍必须明确         |
| 资产和依赖关系 | 通常需要较成熟的CMDB、Service Mapping等能力 | 可以通过统一资产库和业务映射管理 | 数据量可能不大，但仍需要知道关键资产和依赖    |
| 监控      | 需要覆盖大量系统、供应商、区域和业务链             | 根据风险集中监控关键系统     | 可以采用较简单方式，但不能因为规模小就没有监控  |
| 测试      | 测试计划通常更复杂，覆盖更多场景和系统             | 根据关键业务和风险设计测试范围  | 范围可以较小，但仍要证明关键服务具备韧性     |
| 第三方管理   | 供应商数量多、集中度和分包链更复杂               | 重点管理关键供应商        | 供应商少并不等于风险低，尤其可能更依赖单一供应商 |
| 证据和审计   | 通常需要高度制度化、自动化的证据体系              | 可以通过标准化流程保存证据    | 流程可以简单，但仍要证明控制实际执行过      |


监管不是在要求所有机构买一样的工具、建一样大的团队。是要主体负责部门考虑 ，以你自己的规模、风险和复杂度，这套控制够不够？

例如，一家小机构只有十几个关键系统，用一套维护得很好的资产登记也许完全够用。

一家跨国银行如果还靠十几张Excel人工对资产，那就很难用“我们也有资产清单”来解释了。

同样一个控制，放在不同机构里，合理的实现方式可能完全不同。

## 所以DORA到底管谁

我反而觉得Article 2那一长串机构名单没必要看 

它背后的逻辑更重要。

金融行业现在已经不是：

```text
银行
自己建设系统
自己运行系统
自己承担风险
```

而越来越像：

```text
金融机构
   ↓
大量应用和数据
   ↓
Cloud / SaaS / Network / Data / Outsourcing
   ↓
Subcontractors
   ↓
更多基础设施
```

金融机构和科技供应商已经连得非常深了。

所以DORA一方面要求金融机构自己建立数字运营韧性，另一方面又开始把ICT供应链纳入监管。

普通第三方，银行自己要管。

特别关键的第三方，欧盟再加一层直接监督。

但不管供应商有没有被认定成CTPP，银行自己的责任都没有消失。

我给可以出三个最简单的结论：

一是DORA管的是整个金融生态，不只是银行。

二是服务可以外包，责任不能跟着一起外包。

三是DORA讲比例原则。不同机构需要达到同样的监管目标，但实现方式不一定完全一样。

因为法规如果完全不考虑机构规模和业务复杂度，最后很容易变成一套大家都在应付的形式主义。

至少DORA从设计上承认，风险管理这件事，本来就不应该所有人做成一个样子。

so...

## 记录一下今天涉及的文件

* Regulation (EU) 2022/2554 — Article 2: Scope
* Regulation (EU) 2022/2554 — Article 4: Proportionality principle
* Regulation (EU) 2022/2554 — Chapter V: Managing of ICT third-party risk
* Regulation (EU) 2022/2554 — Article 31: Designation of critical ICT third-party service providers
* Commission Delegated Regulation (EU) 2024/1502 — criteria for the designation of CTPPs
* ESAs — List of designated Critical ICT Third-Party Providers
