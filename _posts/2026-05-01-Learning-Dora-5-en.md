---

title: "Regulatory Knowledge Platform 01 | I Just Wanted AI to Read DORA, but Later I Found the PDF Itself Was the Problem"

date: 2026-05-01

categories:

* Cybersecurity

tags:

* Cybersecurity

* Network Security

* Fintech

* DORA

* Vulnerability Management

* Digital Operational Resilience Act

* GRC

* Regulatory Compliance

* EU Regulation

* Governance, Risk, and Compliance

* ICT Risk Management

---

When I was writing DORA 02, I was still struggling with how many documents DORA actually has. At first I thought finding Regulation (EU) 2022/2554 was the end. Then I found 2556, and after that came different Delegated Regulations, Implementing Regulations, RTS, ITS, Guidelines, Final Reports, plus materials from EBA, ESMA, EIOPA and CSSF.

More importantly, I could not understand them!!!

Think about it, I have a CS background and basically had no idea about legal things. At the beginning I thought I could just throw all these PDFs to AI and it would help me understand them. Later I found that the problem was not really AI. The PDF itself was also a problem.

At that time, I spent quite a lot of time downloading these documents one by one, and finally created a folder on my computer. There were maybe ten or twenty PDFs. I renamed them, roughly knew which one was which, and honestly it felt quite good. My idea was also very direct. AI is already so powerful, so why not just put all of them into AI? Actually, my earliest Regulatory RAG also started more or less like this.

But after working on it for some time, I found that I was being led by AI all the time, while I still did not really understand these things myself.

## What is a PDF actually?

For example, from a computer's point of view, Regulation (EU) 2022/2554 is just a file called `2022_2554_EN.pdf`. A slightly more advanced version is maybe just adding some metadata:

```text
{
  "filename": "2022_2554_EN.pdf",
  "title": "Digital Operational Resilience Act"
}
```

But this information was far from enough for what I actually wanted to do.

I have studied English for many years, but I did not understand the meaning behind many professional legal words. For example, is it a Regulation or a Directive? What is CELEX? What is the official ELI link? Is this the current valid version? Where is Article 5? How many paragraphs are under Article 5? Which Chapter does it belong to? Where else is the management body mentioned and further explained? Which part of 2554 is a Level 2 Regulation supplementing? Is the document I have now a formally adopted Commission Delegated Regulation? At the beginning, I did not even know there was something called a Final Report.

Basically, you cannot see these things from the filename.

There is of course nothing wrong with PDF. PDF is good for reading, but being good for reading does not mean it is suitable to become the knowledge structure inside a system. Take Article 5 for example. When I first studied DORA, my main understanding was that ICT Risk cannot simply be pushed completely to IT, and the management body has the final governance responsibility. But when I continued reading, I found that this idea does not stop at Article 5. It continues to appear in the ICT risk management framework, review, approval, reporting and other requirements, and there are also more detailed Level 2 rules.

So what I really wanted to know was not: on which page of the PDF is Article 5? What I wanted to know was: where is Article 5 located in the whole DORA framework, and how is it connected with the later requirements?

At this point, the PDF page number started to become less important to me. Chapter, Article and Paragraph are the real structure of the regulation. The structure was the first layer I needed to understand.

Gradually, I realised that what I lacked was not more PDFs. At the beginning I kept adding more. If one document was missing, I downloaded it. If I found a new RTS, I downloaded another one. If CSSF published some related material, I added that too. It is very easy to have an illusion: my files are becoming more complete, so the system should also become smarter.

But imagine I have 100 regulations, and inside the system they are still only `file_001...file_100`. When I actually started building RAG, this problem became more and more obvious. Every time the model receives a chunk, it has to decide again: which regulation is this? Which Article? Which level? Is it an official legal text? What is its relationship with the main regulation? Sometimes if the chunk is not cut very well and the context is lost, the model may even need to guess which Article this paragraph belongs to.

I cannot ask the LLM to guess everything again every time. What I want is grounded evidence.

What CELEX is does not need AI to decide. The title of Article 5 does not need AI reasoning. If the official data already tells me the legal relationship between a Commission Delegated Regulation and the main DORA Regulation, there is also no reason to ask the model to guess it again every time.

Slowly, I formed the direction I am following now:

> **Build a knowledge graph based underlying system that can work together with LLMs**

The LLM should do what it is good at, for example understanding what a user is really asking, whether several regulations contain similar requirements, or how a certain obligation should be understood in a specific situation. But the identity, structure, source and clear relationships of the regulations should already exist in the data layer.

## Regulations are also not a clean folder structure

In DORA 02, I drew a very simple structure to help myself understand it:

```text
Level 1
   ↓
Level 2
   ↓
Level 3
```

I still think this diagram is not wrong. It was very useful when I first started learning DORA. But once I really started working with the data, I found that the legal world is not this clean.

Some regulations supplement 2554, and some implement a certain requirement. 2556 is also not simply the "next level" under 2554. It mainly amends existing financial sector Directives. One Regulation may reference another Regulation, and one Article may also reference another Article. During the drafting stage, one RTS may first appear as a Consultation Paper, draft RTS and Final Report, and finally become a formal Commission Delegated Regulation. Their names are also often very similar.

Regulations themselves can also change. There can be amendments, corrigenda, or consolidated versions. So if I really want to make this into a system that can be used for a long time, "I have one PDF saved on my computer" is definitely not enough. At least I need to know what legal resource this is, which version it is, what its status was at a certain point in time, and where the official source is.

At this point, I realised that maybe I had been thinking about the problem in the wrong direction.

My first idea was: how can I make AI read these PDFs better?

Later it slowly became: should I first organise the regulations themselves into a structure that machines can really use, and then think about AI?

These two directions are completely different.

## This is how the Regulatory Knowledge Platform started

At that time, I actually did not know what I should call this thing. Regulatory Database sounded too much like a pure database. Knowledge Graph seemed to focus only on relationships. DORA Explorer made the scope too small. Later I slowly started using the current name: **Regulatory Knowledge Platform**.

The name sounds a little big, but what I wanted was really no longer just a regulation reader. Ideally, after one regulation enters the system, it should at least become something like this:

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

In the future, when I open DORA, I can directly see the Articles. From an Article I can jump back to the official legal source. When I see a Level 2 Regulation, I can know why it exists and what its legal basis is. I can also jump from one regulation to related regulations. Later, when Search, RAG or an Agent is connected, they do not need to start from a pile of unfamiliar PDFs and understand everything again every time.

The two approaches are actually very different.

The old way:

```text
A pile of PDFs
   ↓
Cut into Chunks
   ↓
Embedding
   ↓
AI decides everything else
```

And the current way:

```text
Official Regulatory Sources
   ↓
Structured Regulatory Knowledge
   ↓
Search / Graph / RAG / Agent
```

The first one is of course much faster to develop. Actually, it is basically the first version of RAG, and sometimes it is not even as useful as directly asking GPT. The second one is much more troublesome, but at least the system knows what it is actually processing.

When I started structuring regulations, a new problem immediately appeared.

EU-Laws has HTML, XML and different kinds of metadata, while CELLAR also has RDF. Different sources look completely different. If every downstream module needs to understand these raw formats by itself, then Search will have one implementation, Graph another one, and RAG another one. The project will probably become very difficult to maintain very quickly.

So later I added another layer in the middle:

**Parser → Canonical → DocumentView**

The first time I saw the word Canonical, it was really, really difficult for me to understand. Again, it was one of those architecture words that sounded very complicated. My simple understanding is that it is a kind of normalisation of the text and document structure. Of course this is not a very accurate definition, but it was easier for me to understand it this way. Using EU-Laws as an example makes it much easier.

## What EU-Laws gives me is not the same as what I should directly store in my system

At the beginning, my idea was very simple. EU-Laws already has the regulations, so why not just fetch them directly? The webpage already shows Articles, Chapters and the legal text clearly. HTML is available, XML is also available, and CELLAR also has metadata and RDF. The problem is that although all of them describe the same regulation, they look completely different. Some information is easier to find in HTML, some is more stable in XML, and some legal relationships are clearer in RDF.

If I directly let Search, Graph, RAG and the frontend understand these raw formats separately, the project will quickly become messy. Search needs to recognise Article once, Graph recognises Legal Resource again, RAG uses another cleaned JSON structure, and the frontend needs to understand the structure again for display. The biggest problem is not the amount of code. The real problem is that the same thing may be understood differently in different modules. An Article is an Article here, but in another module it may become only a piece of text. If the upstream structure changes one day, I may not even know how many places I need to modify.

So later I forced myself to separate this into three layers. Parser reads things from outside. Canonical decides what these things are after they enter my system. DocumentView decides how they should finally be shown to people.

Parser is the easiest one to understand. If EU-Laws gives me XML, the Parser extracts things that can be clearly identified, such as CELEX, Title, Chapter, Article, Paragraph, Recital and Annex. If the source changes to HTML, then I use another Parser. Here I gradually developed one rule: **the Parser should not be too smart.** At the beginning, when I saw a regulation paragraph, I always wanted to also decide whether it was an obligation, whether it was an exception, which risk topic it belonged to, and sometimes even cut the chunks at the same time. It feels very good when writing it, because one function can do everything. Two months later, I basically do not dare to touch it. Now the Parser should mainly deal with things that are certain. Article 5 is Article 5. CELEX is CELEX. If this paragraph belongs to Chapter II, then keep that structure. Whether it is an Access Management obligation is already not something the Parser should decide.

## Canonical was the part that confused me for a long time

After the Parser extracts the data, the most direct approach is of course to store it directly. For example, an Article can simply have `number`, `title` and `text`, and this seems enough. The problem is that today this field may have one name in EU-Laws XML, tomorrow it may have another name in HTML, and if I change to another source, it may not even have the same field structure. I cannot let the whole system know how every external website is designed.

So the meaning of Canonical is basically this: I cannot control the external data, but after it enters my system, it needs to follow my rules.

The most important thing here is actually not how the fields are named. It is that the system starts to have its own stable regulation structure. Before, I just used whatever EU-Laws gave me. Now EU-Laws is only a source. After the data enters the system, it first becomes my own Canonical Model. Even if I change the data source later, in theory only the Parser in front should change. Search, Graph, RAG and API should not all need to change together.

I really like decoupling systems. Maybe this comes from the phrase I had to memorise countless times at university: high cohesion and low coupling.

Of course, Canonical can also easily become bigger and bigger. If I put everything into it, it will finally become another garbage box. So now I am actually quite conservative. A confirmed fact like CELEX can go inside. An Article belonging to a certain Chapter can go inside. If official data confirms that one Regulation supplements another Regulation, that can also go inside. But "this Article mainly relates to Access Management" is different. It may come from manual classification, AI extraction, or simply my own mapping for business use. These things should not be mixed together with the original regulatory facts.

One of the most important things for my Platform is to keep this boundary clear: what the regulation itself is, and how I understand it later, are two different things.

## Then what is DocumentView for?

After finishing Canonical, I also needed to think about visualisation. After all, my users are people who work with legal and regulatory requirements.

The system may care about things like `id`, `parent_id`, `source_ref` and `text_blocks`, but when people read regulations, they still want to see the familiar structure: Chapter II, Article 5, Governance and organisation, and then the paragraphs below it. Users do not care how my Canonical Model is designed. They just want to click Article 5, know which Chapter it belongs to, what the previous and next Articles are, where the official EU-Laws link is, and which regulations are related to it.

So DocumentView is basically reorganising the underlying structure into something humans can read. It sounds a little funny. First I split the regulation into Chapter, Article and Paragraph, and then for the user I put them together again. But this "split and rebuild" is actually useful, because the underlying facts can stay unified while the outside View can be different.

A normal user sees a normal regulation reading page. Graph sees relationships between Legal Resources. RAG may receive the Article text together with its parent Chapter, source and related metadata. The same regulation can look different for different modules. I used to think that one piece of data should simply be passed from beginning to end with as few transformations as possible. Now I am quite sure that having a unified underlying layer and changing the presentation according to different use cases is easier to maintain in the long term.

This design also had a big impact on my RAG later. My earliest RAG was very traditional: PDF to text, cut chunks, embedding, vector search. The problem is that once the text is cut into chunks, the original legal structure can easily be lost. If the model only receives a sentence such as "The management body shall define, approve, oversee...", of course it can still answer something. But if it also knows that this is Regulation (EU) 2022/2554, Article 5, which Chapter it belongs to, and what the official source is, then the meaning is completely different.

So I definitely cannot understand a chunk as simply "a piece of text with 500 tokens". A Chunk is only the retrieval unit. Behind it, it should still know which Article, which Chapter, which regulation and which official source it comes from. Then when the final answer is generated, I can at least try to make sure that the citation is not just one random sentence cut from a PDF, but evidence with a clear legal position.

This goes back to the problem I kept talking about in the previous part: what I want is grounded evidence, not asking the LLM to guess everything again every time.

Of course, things were still not finished here. Should Paragraph be a separate Node inside Canonical? Should Recital and Article use the same structure? How should multilingual versions be handled? How should a consolidated version be stored? What happens to old versions after an amendment? Any one of these questions can take a long time.

If I tried to solve everything from the beginning, this Platform would probably still be at the architecture diagram stage now.

So later I removed a lot of things. First make sure the regulatory source can be found, the identity is clear, the Article structure is not lost, the context can be restored, and the relationships can be traced. As long as Search and RAG can use it normally, that is enough for now. Other complicated problems can be added later when they are really needed.

Finally, after I really extracted the relationships from EU-Laws and CELLAR, it became very clear that the tree structure was only the simplest layer. Regulations also have relationships such as amends, supplements, implements and references, and one regulation can connect to many other legal resources at the same time.

The Platform was starting to look like a Knowledge Graph.

Happy!
