---

title: "Regulatory Knowledge Platform 02 | After Drawing DORA as a Graph, I Finally Started to Understand How These Regulations Are Related"

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

I started to break the regulations from EUR-Lex into structured data. Of course, there are things like Parser and Canonical in the middle, but honestly these are not the most interesting parts of this project for me. The Parser basically parses the XML, HTML and metadata from EUR-Lex, and Canonical normalises data from different sources into my own format. The point where I really started to feel that this Platform was becoming interesting was when I also brought the relationships from CELLAR and EUR-Lex into it.

At this point, DORA suddenly stopped being just a pile of files.

Before this, the Level 1, Level 2 and Level 3 tree was only something I drew for myself to understand DORA:

```text
Level 1
   ↓
Level 2
   ↓
Level 3
```

It was very useful when I first started learning, and I still use this explanation now. But after building the Graph, I found that this is actually only a very rough abstraction. The real relationships between regulations are definitely not such a clean tree.

For example, Regulation (EU) 2022/2554 is the main DORA Regulation, but the Commission Delegated Regulations below it are not simply "child documents". Some supplement certain Articles of the main Regulation, some explain further how a requirement should be implemented, and some legal resources also have amendment, reference and other relationships between them. Directive (EU) 2022/2556 is even more obvious. It is not Level 2 under 2554 at all. It amends a group of existing financial-sector Directives so that the old regulatory framework can connect with DORA.

So the real data is more like:

```text
2022/2554 ──────→ 2024/1774
    │
    ├──────────→ other Level 2 acts
    │
    ├──────────→ related Directives
    │
    └──────────→ referenced legal resources
```

And every line does not even mean the same thing.

This was the first time I clearly felt that regulations naturally fit a Graph. I had studied Knowledge Graphs quite seriously when I was studying in France, but at that time most things I did at school were demo projects. Now I finally had a very clear and meaningful real use case.

Legal resources are not naturally organised like folders. One resource can have different types of relationships with many other resources at the same time. This is a very standard many-to-many network.

My first Graph was actually quite ugly, and basically useless. I used all the regulations as Nodes, imported the relationships from CELLAR, ran everything in GraphDB, and suddenly there were a lot of nodes and edges everywhere. It looked really impressive.

But... so what~~

Then I clicked one Edge:

> Legal resource amends legal resource

Then another one:

> Legal resource cites legal resource

Then I clicked a few more, still more or less the same.

Suddenly I realised that this Graph looked good, but maybe it was not really useful.

Because the real question was never:

> Is there a line between A and B?

The real question was:

**Why is this line here? What does this line actually mean for understanding DORA?**

For example, if someone working in ICT Risk opens 2022/2554, he is not there to admire the fact that this Node has 18 Edges. What he really wants to know is: which Level 2 regulations continue to give details on ICT risk management? What is the legal basis of this Delegated Regulation? After one Article tells me to do something, where are the detailed requirements explained later? If one regulation is amended by another regulation, what exactly was amended? If the Graph cannot answer these questions, then it is just a nice-looking spider web.

This problem influenced me quite a lot. When I studied Knowledge Graphs before, I always felt that once the Nodes and Edges were created, half the work was already done. Now I think the difficult part is not really connecting things. The difficult part is whether the Edge actually has meaning.

Another question that took me quite a long time was how detailed a Node should be. The simplest approach is of course one regulation as one Node.

For example:

```text
Regulation (EU) 2022/2554
Commission Delegated Regulation (EU) 2024/1774
Directive (EU) 2022/2556
```

This is easy to understand, and EUR-Lex already has CELEX, which is very suitable as a unique identifier. So at the first level of the Graph, I still use Legal Resource as the most stable Node.

Then the next question immediately appears: should an Article also be a Node?

If Article is not a Node, many useful relationships can only stay at the whole-regulation level. For example, I can only know that 2024/1774 is related to 2022/2554, but I cannot know which specific part of the main Regulation it is actually giving more detail about.

But if every Article, Paragraph and even Recital becomes a Node, the Graph becomes huge very quickly, and many of those Nodes may not really need to exist independently. If every Paragraph is separated, one regulation can already become hundreds of Nodes. After importing dozens of regulations, it can easily become a very large Graph. In theory it looks complete, but in real use it may not be better.

So later I stopped trying to make "everything into a Node".

Now I prefer to handle it in layers. Legal Resource is definitely the core Node. Article is also a very useful structural Node, because legal references and RAG evidence usually need to finally point to an Article. Whether Paragraph and Recital should also become independent Nodes depends on the real requirement later. I do not want to split everything just to make the Graph look complete.

So as someone with a bit of OCD, I have to stop myself from doing "complete modelling". Now, more practically, the main thing I need to think about is whether this Node will actually be queried.

If one Node does nothing except increase the number of Graph nodes from 200 to 20,000, I can probably leave it out for now.

## Edge is more important than Node

After I really started organising the Edges, the Graph slowly changed from "visualisation" into "knowledge".

The relationships I cared about first included things like `amends`, `supplements`, `implements` and `references`. The names themselves are not the most important part. The important thing is to separate different legal relationships instead of turning everything into one universal `RELATED_TO`.

If the final Graph is only:

```text
A ── RELATED_TO ──→ B
B ── RELATED_TO ──→ C
C ── RELATED_TO ──→ D
```

then basically it says nothing.

This is quite different from the normal network analysis I did before. In a social network, sometimes an Edge only needs to represent "knows" or "has made a transaction with", and that can already be useful. But regulations are different. Different Edges can have very different legal meanings.

`amends` means changing an existing legal resource.

`supplements` means adding more detailed requirements under an existing framework.

`references` may simply mean that another legal resource is mentioned in the text.

If these relationships are mixed together, any analysis I do later will have problems.

For example, in the future a user may ask:

> What more detailed requirements are there under DORA Article 6 for the ICT risk management framework?

If the Graph only has `RELATED_TO`, the system can only tell me "these things are related".

Well, obviously.

What I really want is for the system to follow a more specific legal relationship, find the relevant Level 2 resources, and then pass the actually related Articles to RAG. At that point, Graph is no longer just for drawing pictures. It becomes part of retrieval.

I also separated relationships into "official" and "derived by myself", and this became very important later. EUR-Lex / CELLAR can provide official metadata and legal relationships. I am happy to directly treat this kind of information as facts in the Platform. Where it comes from, what the CELEX is, which resource amends which resource, all of these have a source.

But there are also many relationships that I really want, but the official Graph will not directly build for me.

For example:

> Article X is mainly related to Access Management.

> Article Y is related to ICT Third-Party Risk.

> This obligation may finally map to a certain Control Objective.

These relationships are also very useful, and may even be more useful for the real Risk Copilot. But they are no longer the same kind of thing as `amends`. Some may come from manual mapping, some from rules, and some may be extracted by an LLM later.

So I do not want to mix all of them into the same "fact layer". Otherwise the Graph looks richer and richer, but slowly a very dangerous problem appears: I do not know whether one Edge came from the EU official source, or whether a model guessed it three months ago.

For a normal recommendation system, maybe this is not such a big problem. But what I am building is Regulatory Knowledge. If a user clicks one relationship and asks:

> Why?

I must be able to explain where it came from.

So now I prefer to think of the Graph as having more than one relationship layer. Official legal relationships are the foundation.

Curated / derived relationships can be added on top, but I need to know who created them and based on what evidence. LLM-generated relationships can also enter the Graph in the future, but they must carry provenance. They cannot be generated and then directly pretend to be facts. This is actually the same logic as why I keep emphasising citation in RAG.

## Article-Level Graph started to actually solve the real pain points

A Graph only between Legal Resources already makes the overall DORA legal framework much clearer. But the real problems I want to solve still need to go down to the Article level.

Because users will not always ask:

> What is the relationship between 2024/1774 and 2022/2554?

More common questions are:

> What exactly does DORA require for the ICT Risk Management Framework?

> Which Articles clearly require the management body to participate?

> In which Articles are the Access Management requirements spread?

These questions are not really document-level questions.

So later I increasingly wanted the Graph to keep the high-level relationships between regulations, but also go down to Articles.

Something like:

```text
Legal Resource
    ↓ contains
Article
    ↓ references / related legal basis
Article / Legal Resource
```

Starting from one Article in the main DORA Regulation, I can first know which regulation it belongs to, then find other legal resources related to it, and continue into the related Articles.

At this point, the combination of Graph and RAG becomes interesting. Normal Vector Search starts from text similarity. Graph provides a completely different kind of information:

**Which other parts of the legal system have a clear relationship with this text.**

The two are not replacements for each other. RAG alone cannot do this.

For example, if the user asks a very specific ICT risk question, Vector Search may first find the most similar Article. The Graph can then tell the system which official Level 2 resources supplement this Article, or what clear reference relationships exist nearby. Then RAG reads the actual legal text.

This is much more reliable than putting all regulations into one Vector DB and hoping that embeddings somehow understand the whole legal system by themselves.

At least this is the direction I believe in now.

At the beginning, I really liked the visualisation of the Graph, because honestly it looks cool and makes the project look more impressive. You click one Node, all the legal relationships around it open, then click another Node and continue expanding. It really feels quite nice. I still think making it into a regulation map similar to Google Maps is an interesting idea. I also showed it to a lawyer in the company and he agreed that the idea was useful.

For example, I can follow one Regulation and find all the Level 2 regulations that officially supplement it, see which existing laws were amended by a legal resource, enter related regulations from an Article, or go backwards from a Delegated Regulation and see which DORA requirement it was based on. In the future, after adding business-level relationships, it may even be possible to go from Regulation to Obligation, Risk, Control Objective and Metric.

## At this point, the Platform finally started to show its real meaning

The final business flow of the whole thing is becoming clearer: official regulations first enter the Platform. The Platform must keep confirmed legal facts, structures and relationships. RAG then retrieves and answers based on this layer. Later, Agent handles Risk, Control and specific business actions.

Never let the LLM reinvent facts.

My goal is also not simply to build another regulation website. It is more like preparing a regulatory map for the later RAG and Risk Copilot.

I also do not want to build one "super Knowledge Graph" and put everything into the same Graph. It may look very powerful, but in the end nobody knows which part belongs to the regulation itself and which part belongs to the company's own control framework.

So for now, I still want the Platform Graph to mainly stay at the regulatory fact layer. First understand the relationships inside the legal world itself, and keep the boundaries strict. This is basically putting a harness on the AI.

Then the AI can work on top of it.
