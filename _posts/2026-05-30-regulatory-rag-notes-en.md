---
title: "RAG Is Still an Engineering Problem"
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

Let’s start from the beginning, when I first started building RAG.

A regulatory knowledge platform has now taken shape. I started mainly with DORA, then added some official GDPR and AML materials. The original idea was very simple: ChatGPT is already this powerful, so why not throw the regulations into a RAG system and let users ask questions directly? For example:

What are the minimum requirements for an ICT risk management framework under DORA? Which provisions explicitly require the management body to be involved and conduct an annual review?

At first, I didn’t think this would be difficult. Parse the regulatory PDFs, split them into chunks by provision, create embeddings, and put them into a vector index. When a user asks a question, use vector search to retrieve the Top K, feed the relevant content to a large model, and tell it to “answer only from the context and provide citations.”

That is basically the pattern used by 99% of RAG tutorials online. The first version was up and running very quickly, and seeing AI actually find things in regulations and answer questions for the first time was genuinely satisfying. Ask what DORA requires for ICT risk management and it could find Article 5 and Article 6. Ask about ICT third-party risk and it could find content around Article 28. It looked usable already.

Then I quickly realised something was wrong. The biggest problem was not that it hallucinated everything. It was that it was often “approximately right.”

That might be acceptable in a normal knowledge base. It is not acceptable in regulation. A regulation may list ten requirements, and the AI summarises them into six. The original text says “shall consider,” and the answer turns it into “must implement.” A requirement only applies to a specific financial entity, but the scope disappears in the generated answer. There is an even subtler version: the answer itself looks fine and has an Article citation after it, but when you actually open the Article, it supports only the first half of the sentence. The second half is something the model inferred by following the logic. These answers are the most troublesome because the mistake is not obvious at a glance. If anything, they look extremely professional.

So I kept adding things. By now, the pipeline has become Query Planner, claim decomposition, BM25 and vector hybrid search, reranking with structural and field-level signals, metadata filtering, claim-bound evidence, and finally answer and citation generation. It sounds fairly complete, but what actually worked turned out to be quite different from the original design.

## Optimisation 1: Regulations cannot be sliced into arbitrary 500-token chunks like ordinary documents. Regulations are naturally highly chunked, structured texts.

The kind of tutorials people sell as online courses are basically built around this stupid approach.

Of course I also tried fixed-length chunks at the beginning. There is nothing technically difficult about it: an 800-token chunk with a 100-token overlap, for example. That can work perfectly well for ordinary enterprise documents. Regulations absolutely cannot be handled this way.

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

If the token boundary happens to fall in the wrong place, paragraph 3 may stay in one chunk while points (a), (b), and (c) end up in another. The user’s query retrieves “Financial entities shall...”, but the actual requirements listed underneath do not come back with it. At that point, the AI is not even hallucinating. The context you gave it was incomplete to begin with.

I was very clear about this part. Legal material is insanely complicated and difficult to read, but from the beginning of the design I understood that regulations already have a very strong structure. Why would I destroy that structure first, then ask embeddings to guess it back? Now the basic structure is:

```text
Document → Chapter → Section → Article → Paragraph → Point → Sub-point
```

A chunk does not just store a piece of text. It also stores which regulation, Article and paragraph it belongs to, together with its parent structure. Roughly like this:

```json
{"document":"Regulation (EU) 2022/2554","celex":"32022R2554","article":"5","paragraph":"2","title":"Governance and organisation","text":"..."}
```

Later, I separated the official source files, the outputs from different parsers, and the system’s unified provision structure. At the time I wondered whether I was making things too complicated. Once I started adding new regulations, I realised this was unavoidable. Even different regulatory documents have different layouts. What a parser extracts should not automatically become the system’s final structure. In the end, Articles, Paragraphs, Annexes and the rest still have to be normalised.

This still has not reached the point where you can “throw in any regulation and be done with it.” DORA, GDPR and AML materials go through the same build pipeline, but every new type of source still brings parsing, versioning and provision-mapping problems.

So my understanding of chunking is now completely different from where it started. Ordinary RAG asks, “How large should this chunk be?” Regulatory RAG first has to ask, “What exactly is this chunk within the legal structure?”

## Optimisation 2: Pure vector search is absolutely not enough. This is just common sense.

Take pure vector retrieval. Suppose the user asks:

 What are the responsibilities of the management body under DORA?

Vector search can obviously find plenty of similar content, because words like management, governance, ICT risk and responsibility appear all over DORA. The problem is that what the user may actually need is Article 5, while the vector results often contain a pile of semantically related paragraphs that do not directly answer the question.

Regulations also contain large numbers of specialist terms, regulatory identifiers and fixed expressions.

```text
Article 31
Regulation (EU) 2024/1774
JC/GL/2024/36
critical or important function
major ICT-related incident
ICT third-party service provider
```

BM25 is particularly useful in this situation. So I eventually went straight to Hybrid Search: retrieve through BM25 and Vector in parallel, then fuse and rerank the results.

```text
Question → BM25 + Vector → Merge → Rerank → Evidence
```

The current reranker does not call another expensive model. It uses auditable signals such as provision roles, heading matches and exception provisions to reorder the candidates. If the user is clearly asking for a definition, definition provisions get priority. If the user is not asking about an exception, an exception provision should not somehow jump to the top.

Some articles online will say, “Accuracy jumped directly from 68% to 79% after adding reranking.” I do not have that kind of data, and I do not want to invent it. What I can say from actual use is that the top results used to contain one or two items that looked similar but did not answer the question. That happens noticeably less often now, especially when several Articles discuss the same topic. That is enough.

## Optimisation 3: The real problem is not retrieval. It is that one question may not actually be one question.

This was where I finally ran into trouble. No matter how nicely I tuned Hybrid, the results were still bad.

```text
1. What are the minimum requirements for an ICT risk management framework?
2. What is the legal basis for management body involvement?
3. Where is the annual review requirement?
```

Previously, one retrieval would return a Top 10. All ten results looked “relevant,” but perhaps six pieces of evidence covered the first question, three covered the second, and none covered the third. Then the LLM would start writing from those ten results and would usually still manage to produce an answer to the third question. That is the most absurd part.

So v0.4 introduced a Query Planner. The model first understands the question, then decides what actually needs to be searched. Complex questions are decomposed into claims, and each claim is retrieved separately. Every required claim now has its own coverage status. If the evidence is insufficient, the system says so explicitly instead of dressing “we did not find it” up as “the regulation does not require it.”

The cost is also obvious. Most of the latency comes from the LLM, not BM25, the vector index or the reranker. A full planned answer first has to understand the question, split it into claims, retrieve them separately, process the evidence, and then generate the answer. Current local end-to-end measurements are roughly 15 to 28 seconds. That is much better than the minute-long waits from earlier versions, but it still feels slow for a simple question.

If a user asks, “What is CTPP?”, and the system sits there carefully planning, decomposing claims, retrieving evidence and reranking before eventually saying “Critical ICT Third-Party Provider,” the user has already left.

So at this stage I actually started removing things. Complex questions need a Planner. Simple questions do not necessarily need the same heavy strategy. RAG systems easily develop this problem: to solve the hardest 10% of cases, they make the other 90% unnecessarily complicated. I like this kind of design less and less.

## Optimisation 4: Then I realised a Query Planner was still not enough. First, the system has to know what “type” of question the user is asking.

Recently I ran into another problem involving the anti-money laundering law most commonly used in the company:

Which customer due diligence measures are required under the Fourth Anti-Money Laundering Directive?

The system could already find the correct provisions. The evidence was fine. But the final answer still felt wrong. The reason was simple: this is an enumeration question.

The legal text may explicitly list requirements under (a), (b), (c) and (d). What the user is really asking is, “What exactly are they?” What does an LLM like doing more than anything? Summarising. Give it ten requirements and it compresses them into six. All six may be correct, so you cannot even say the answer is wrong. But the qualifications, conditions and itemised structure in the regulation have disappeared. That is fine for an ordinary summary. It is completely different when the user asks, “What are the legal requirements?”

So now I am not merely rewriting the user’s question into a better search query. I also handle it according to retrieval intent. The system already has directions such as definition, minimum requirements, obligation, scope, exception, threshold, procedure and compound. For “what are the requirements?” questions, the retrieval layer tries to bring back the parent provision together with its complete, ordered child items, instead of pretending that a few topically similar fragments form a complete answer. Definitions should prioritise definition provisions, not drag in a pile of commentary. Scope has to consider exceptions, not just search for scope. Numbers, dates and conditions in threshold questions have to be checked separately. Compound questions get decomposed into claims.

This is still not completely finished, but I find it much more interesting than switching embedding models again. At this stage, the question is no longer just, “Can you retrieve accurately?” It is: do you know what kind of legal question the user is actually asking?

## Optimisation 5: A citation is not a few Article numbers attached to the end of an answer.

I added citations almost from day one because I have always believed that regulatory RAG without sources is basically meaningless. The first version was simple:

```text
[1] Article 5
[2] Article 6
[3] Article 28
```

Later, citations included the trusted document, provision coordinates, an excerpt from the original text and an official source link. I thought that was already pretty good. Then the problems appeared again. For example, the AI answers:

> The management body must approve the ICT risk framework, review X annually, and ensure Y.

It puts Article 5 after the sentence, and everything looks properly supported. But if you actually open it, Article 5 may support only the first part. The annual review may be in another provision, and Y may simply be something the model inferred from context. So “the answer has citations” and “every conclusion has evidence” are two different things. Now I prefer claim-bound answers:

```text
Claim 1 → Evidence A
Claim 2 → Evidence B
Claim 3 → Evidence C
```

Every claim has to know which evidence is keeping it alive. If there is no evidence, do not sneak it in. This is one of the major differences between ordinary RAG and regulatory RAG. LLMs are extremely good at taking three correct facts and inferring a fourth fact that also sounds perfectly reasonable. That is fine in casual conversation. In Compliance, it is a problem.

## Optimisation 6: Metadata eventually became extremely important in the regulatory system.

At first, I added metadata mainly to know where a chunk came from. Then I gradually realised that it could also constrain retrieval. Suppose the user explicitly asks:

> According to Regulation (EU) 2024/1774...

Why would I search the entire corpus? Just do this:

```text
document = 2024/1774
```

Done. If the question is about DORA, do not somehow put a semantically similar GDPR or AML provision into the answer. The metadata now includes not only basics such as Article, document and CELEX, but also framework, jurisdiction, document type, legal effect, current version, application date and official source. Scope filtering is shared by every Retriever. BM25 does not apply one set of filters while Vector applies another.

At this point, metadata filtering is no longer the enterprise knowledge-base version of language = java or type = guide. It starts to resemble the applicability problem in regulation: which law? Which version? What was in force at what time? Does it apply to this entity?

There is still plenty more to do here. Some corpus boundaries and version states are under control now, but moving from that to applicability decisions for a specific organisation and each individual obligation is still a long way from “fully solved.”

## Optimisation 7: When generating the final answer, the model cannot be too clever.

This is counterintuitive, right? When choosing an LLM, everyone initially wants the smarter one. The stronger the model and the stronger its reasoning, the better the answer. It turns out that this is not necessarily true in regulatory work.

The smarter the model is, the more it sometimes wants to “help.” The source lists ten items, but that feels too long, so it merges a few for you. The source says shall consider, but that does not sound direct enough, so it changes it to must implement. The source applies only in a specific situation, but logically other situations probably work the same way, so it casually expands the scope.

The model is not making these things up. It is even trying to “help.” But I do not want its help. So I increasingly constrain answer generation: tell it which evidence corresponds to each claim, how far that evidence supports the claim, that numbers cannot change, exceptions cannot disappear, scope cannot expand, and that it must explicitly say when there is no evidence. The generator cannot invent citations either. It can only select from the evidence retrieved for that request.

I used to think the LLM’s role was: understand the retrieved material and give me an intelligent answer.

The reality is: I have already done my best to find the evidence. Do not get creative. Just organise it into something a human can read. This detail matters. I figured it out one step at a time.

## The final optimisation: Evaluation. I now think this is the most important part, and also the part I have done least well.

Once you reach this stage, the biggest problem is that you have no idea whether you have actually improved anything. Change a prompt today. Test a few questions. Feels good. Adjust the Hybrid weighting tomorrow. Test a few more. Also feels good. Replace the Planner a few days later. Complex questions seem better again. It is all “feeling.”

So I have started building Golden Tests and evaluation. But I do not want only correct, partial and wrong. That is far too crude for regulatory questions.

Suppose the source contains ten obligations and the model answers with eight, all of them correct. An ordinary LLM Judge will probably mark it correct. I would call it incomplete. Or suppose the source says at least annually and the model says regularly. Is the general idea right? Yes. Is it legally the same? Obviously not.

There is now a frozen question set and a set of automated checks. At minimum, they check the expected status and whether required evidence was retrieved and cited. But faithfulness, completeness, and whether a citation really supports an answer still require human review. I am not going to let another LLM Judge produce a score and pretend that makes it true. This work is still ongoing. On the fixed retrieval question set:

Pure Vector has a Recall@5 of 80% and an MRR@5 of 59.8%.
After adding BM25 through RRF Hybrid, Recall@5 reaches 86.7% and MRR@5 reaches 75.9%.
In other words, compared with pure Vector, Hybrid improves Top 5 recall by 6.7 percentage points and ranking quality by 16.1 percentage points.
Heading- and legal-structure-aware chunking, metadata filtering, intent analysis, reranking and the Planner are all in use now, but I have not switched them off one by one and rerun the same frozen test set. So I do not want to invent attractive-looking numbers such as “68%, 79%, 91%.”
Before the retrieval enhancements, required-evidence recall was 58.9% for both the extractive and Private AI paths. After adding scope-local BM25 and the retrieval enhancements, the extractive path reached 100%, while the historical Private AI run reached 96.2%.

Citations do not directly improve retrieval accuracy. They solve a different problem: letting the user see exactly which provision a statement came from, whether the AI expanded the meaning of the original text, and whether a claim has evidence at all.

Is 95% good enough? For regulatory RAG, I do not think that is how the question should be framed.

The remainder does not necessarily mean “the model answered incorrectly.” Sometimes the answer genuinely is not in the corpus. Sometimes the question is missing scope. Sometimes the regulation requires a judgement based on entity type, timing and specific facts. When the system correctly says “insufficient evidence” or “clarification required,” that is itself correct behaviour.

One thing I have learned.

After a month of working on RAG, my biggest takeaway is this: the bottleneck in RAG is not the model. It is engineering, engineering, engineering, engineering, engineering, engineering, engineering, engineering, engineering, engineering, engineering!

Whether you use GPT or DeepSeek, or which embedding model you choose, obviously makes a difference. But what really separates systems is whether the chunking preserves the legal structure, whether retrieval finds the right provisions, whether scope is filtered correctly, and whether every conclusion is actually bound to evidence.
Many people start by obsessing over prompt engineering, tuning temperature and rewriting the system prompt.
But if the context you feed the model is garbage, it does not matter how fancy the prompt is.
Trash in, trash out.
C’est RAG.
