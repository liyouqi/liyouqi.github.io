---
title: "Learning DORA 02 | I Just Wanted to Download DORA—Why Did I End Up with More Than Ten Files?"
date: 2026-04-28
categories:
  - Cybersecurity
tags:
  - Cybersecurity
  - Fintech
  - DORA
  - Regulatory Compliance
  - EU Regulation
  - Digital Operational Resilience Act

layout: single
author_profile: true
read_time: true
comments: false
share: true
---

At first, I thought DORA was simply one regulation, which meant one PDF.

My manager asked me to study DORA, and my plan was also very simple: download the regulation, give it to AI, and then slowly work through it from the first page.

Instead, I got completely lost before I had even finished downloading the files.

I first found **Regulation (EU) 2022/2554** on EUR-Lex and thought the job was done. Then another document appeared: **Directive (EU) 2022/2556**.

As I continued searching, I came across even more terms:

- Commission Delegated Regulation
- Commission Implementing Regulation
- RTS
- ITS
- Guidelines
- Final Report
- Q&A
- Reporting tools
- CSSF Circular

What made it worse was that these documents were not even on the same website.

Some were on EUR-Lex, some on the European Commission website, some on EBA, and others on ESMA or EIOPA. When it came to local implementation in Luxembourg, I also had to look at CSSF.

It felt like every website had its own DORA section, but none of them looked like the one and only “official DORA website”.

For a while, I wondered whether I had downloaded the wrong things, or whether I had somehow missed a complete package that somebody else had already put together.

Eventually, I realised that DORA was never something that could fit into a single PDF.

It is more like a tree that keeps branching out from the top.

At the top is the main regulation, which establishes the overall framework. Below that, a series of regulations and supervisory documents expand on specific topics such as risk management, incident reporting, resilience testing, third-party providers, and regulatory reporting.

For the moment, I understand it as three broad levels:

```text
Level 1
The overall legal framework
      ↓
Level 2
More detailed requirements, standards, thresholds, and templates
      ↓
Level 3
Supervisory guidance intended to support a consistent understanding
among national authorities and financial entities
```

Below those levels, there are also local implementation requirements issued by national supervisory authorities.

For me, that means CSSF tells banks in Luxembourg who to report to, which channel to use, and which procedure to follow.

## 2554 Is What People Normally Mean by the Main DORA Regulation

**Regulation (EU) 2022/2554** is the main body of DORA.

It establishes the overall framework, including:

- ICT risk management
- ICT incident management and reporting
- Digital operational resilience testing
- ICT third-party risk management
- Oversight of critical ICT third-party service providers
- Management responsibilities
- Supervisory and enforcement mechanisms

I currently think of 2554 as the main framework or master document.

But calling it a framework does not mean that it only contains broad principles that sound correct but are not particularly useful.

It already contains many direct obligations that financial entities must follow. It is just that a large amount of operational detail still needs to be provided by the documents below it.

For example, 2554 tells financial entities that:

> Major ICT-related incidents must be reported to the competent authority.

But it cannot include every incident threshold, reporting deadline, form field, and submission procedure in the same provision.

Those details have to be expanded in later regulations.

## So What Is 2556 For?

**Directive (EU) 2022/2556** is not another version of DORA, and it is not the second half of 2554.

The EU already had many existing laws covering banking, payments, securities, investment funds, and insurance.

Once DORA was introduced, the parts of those older laws dealing with ICT risk and operational resilience also had to be adjusted. Otherwise, there could easily have been conflicts or overlaps between the old rules and the new DORA framework.

The role of 2556 is therefore more about amending and connecting the existing financial-sector directives.

It amended several existing directives, including UCITS, Solvency II, AIFMD, CRD, BRRD, MiFID II, PSD2, and IORP II, so that they could align with the new DORA framework.

I think the easiest way to remember it is:

```text
2554: establishes the new DORA framework

2556: amends existing financial-sector directives
so that they can align with DORA
```

For the day-to-day implementation of DORA controls, risk assessments, incident management, and third-party management, banks mainly work with 2554 and the more detailed regulations that follow it.

2556 is more about what needed to happen to the existing financial regulatory framework after DORA was introduced.

## Why Are There More Than Ten Level 2 Documents?

Because 2554 cannot include every operational detail.

Take major ICT incident reporting as an example.

The main regulation can say:

> Financial entities must report major ICT-related incidents.

But as soon as a bank tries to implement that requirement, a long list of questions appears:

- What exactly counts as major?
- Does an outage lasting two hours qualify?
- How quickly must it be reported?
- Is there one report, or are reports submitted in stages?
- What happens if the root cause is still unknown at the time of the first report?
- Which fields need to be completed?
- Which template should be used?
- Which authority receives the report?

Every bank cannot be allowed to invent its own answers, and every Member State cannot create a completely different standard.

DORA therefore authorises the European Commission to adopt a series of **delegated acts** and **implementing acts** that provide the detailed standards and execution methods.

That is Level 2.

A simple way to understand it is:

```text
2554 creates the obligation
      ↓
Level 2 turns the obligation into more detailed
standards, methods, thresholds, and templates
```

This is also where two common abbreviations appear:

- **RTS: Regulatory Technical Standards**
- **ITS: Implementing Technical Standards**

My current, slightly rough understanding is this:

RTS are more about defining:

> Which standards must be met, which factors must be considered, and which methods should be used to make a decision.

ITS are more about defining:

> How the requirement should be implemented consistently, which templates should be used, which fields should be completed, and which procedure should be followed.

The distinction is not perfectly clear-cut in every document, but for now this explanation is good enough for me.

For example, major ICT incident reporting is split across several different regulations:

- One defines incident classification and materiality criteria
- One defines when reporting must happen and how many reporting stages there are
- One defines the specific forms and fields to be used

These documents are not repeating each other. They are answering different questions.

## Why a Final Report Cannot Be Treated as the Final Regulation

This was another trap I almost fell into.

On the EBA, ESMA, or EIOPA websites, it is common to find documents with titles such as:

> Final Report on draft RTS on ICT Risk Management

The title sounds extremely official, and the PDF may be one or two hundred pages long. It looks very much like the final regulation.

But it is not yet the final Commission Regulation in force.

The process is roughly like this:

```text
DORA authorises the development of a technical standard
      ↓
EBA, ESMA, and EIOPA draft it jointly
      ↓
A Consultation Paper, draft RTS, or Final Report is published
      ↓
The European Commission reviews and formally adopts it
      ↓
It becomes a Commission Delegated Regulation
or a Commission Implementing Regulation
      ↓
It is published in the Official Journal of the European Union
and appears on EUR-Lex
```

EBA, ESMA, and EIOPA are collectively known as the **ESAs**, or European Supervisory Authorities.

Broadly speaking, they cover banking, securities, and insurance respectively.

DORA applies to many different types of financial entities, so a large number of technical standards are drafted jointly by all three authorities rather than by EBA alone.

For example, during the drafting stage, the ICT risk management framework may appear as:

> Final Report on draft RTS on ICT Risk Management Framework

After formal adoption by the European Commission, it becomes:

> Commission Delegated Regulation (EU) 2024/1774

So, if I later see both:

```text
Final Report on draft RTS...
```

and:

```text
Commission Delegated Regulation (EU)...
```

I will use the second one as the main formal legal basis.

That does not mean the Final Report is useless.

In fact, it can sometimes be easier to understand than the final regulation because it explains why the standard was designed in a particular way, what feedback was received during public consultation, and why one approach was selected over another.

But it cannot replace the final regulation.

Otherwise, there is a real risk of spending a long time studying a draft, only to discover that the final published version has already changed.

## So What Is Level 3?

Level 3 does not eventually become a Commission Delegated Regulation or Commission Implementing Regulation in the same way as Level 2.

It mainly consists of supervisory guidance issued by the ESAs, such as **Guidelines**.

The purpose is not to create a new law, but to encourage national supervisory authorities and financial entities to interpret and apply the same DORA requirements in a reasonably consistent way.

The core Level 3 documents I downloaded this time include two sets of Guidelines:

- Guidelines on estimating the aggregated annual costs and losses caused by major ICT-related incidents
- Guidelines on cooperation between the ESAs and national competent authorities in the oversight of critical ICT third-party providers

Q&A documents, Decisions, Final Reports, and reporting tools are also important, but they cannot all simply be placed under Level 3.

For example, in addition to the formal regulation, the Register of Information also comes with materials such as:

- Data dictionary
- Validation rules
- Reporting templates
- Filing rules
- Technical package
- Frequently asked questions

These documents are there to help banks organise and submit the data in practice.

They are not additional pieces of legislation.

## What Are All These Websites Actually For?

After going around in circles, I finally gave myself a simple set of rules.

### EUR-Lex: Find the Formal Legal Text

2554, 2556, and the formally adopted Level 2 regulations can all be downloaded from EUR-Lex.

If I am unsure whether a document is the final legal text, I first check whether it has been published in the Official Journal of the European Union.

Formal legal citations, internal control matrices, and audit references should ultimately go back to EUR-Lex.

### European Commission: Check the Level 2 Catalogue

The European Commission website lists the delegated acts and implementing acts adopted under DORA.

It works more like a catalogue.

It tells me which Level 2 documents have been formally adopted, but the actual legal text usually links back to EUR-Lex.

### ESMA or EIOPA: See the Overall DORA Map

The DORA pages maintained by ESMA and EIOPA bring together Level 1, Level 2, and Level 3 documents and arrange them by topic.

They are slightly more human-friendly.

We previously used the ESMA DORA page to download the full set of core documents.

### EBA: Find Banking and Reporting Materials

DORA does not only apply to banks, but because I currently work in a bank, many EBA materials are closer to my daily work.

In particular:

- Register of Information
- Reporting tools
- Q&A
- Data dictionaries
- Validation rules
- Oversight of critical ICT third-party providers

However, EBA is not the one and only “official DORA website”.

### CSSF: Find Out How It Is Implemented in Luxembourg

EU legislation tells banks:

> What must be done.

CSSF then provides more detail for financial entities in Luxembourg, including:

- Who receives the submission
- Whether eDesk or another channel must be used
- Which procedure applies
- Which local Circulars are relevant
- How incident reporting works in practice
- How the Register of Information should be submitted

For example, DORA requires major ICT-related incidents to be reported, but a financial entity in Luxembourg still has to follow the local process defined by CSSF.

These websites are therefore not competing to become “the official DORA website”.

They are simply working at different levels.

The version I now use for myself is:

```text
Formal legal text: EUR-Lex

Level 2 catalogue: European Commission

Overall DORA map: ESMA / EIOPA

Banking and reporting materials: EBA

Local implementation in Luxembourg: CSSF
```

## How to Quickly Identify What Kind of Document I Am Reading

The next time I open a DORA document, I will first look at the title.

If it says:

> Regulation (EU) 2022/2554

or:

> Commission Delegated Regulation (EU) 2024/1774

and it can be found on EUR-Lex and in the Official Journal, then it is a formal legal text.

If the title says:

> Final Report on draft RTS...

then it is normally a technical-standard draft and explanatory document submitted by the ESAs. It cannot replace the final Commission Regulation.

If the title includes:

> Joint Guidelines  
> JC/GL/2024/34

then it is supervisory guidance.

If the title says:

> Circular CSSF...

then it is a local supervisory document issued by the Luxembourg authority.

If it says:

> Data Dictionary  
> Validation Rules  
> Reporting Package  
> Filing Rules

then it is normally technical reporting material, not another new law.

## Why DORA Cannot Be Reduced to One PDF

At this point, I finally understand why I only wanted to download DORA but ended up with more than ten files.

2554 is responsible for constructing the main structure of the building.

How incidents are classified, when reports must be submitted, which templates must be used, how suppliers should be managed, how TLPT should be performed, and how the Register of Information should be maintained are all developed in separate rooms built by the later regulations.

The Guidelines, Q&A documents, technical tools, and CSSF materials then explain how the building is actually supposed to be used.

I was genuinely confused by all of this at the beginning.

The main task this time was simply to understand the documents themselves.

Otherwise, I could spend a long time studying DORA without even knowing whether I was reading a formal regulation, supervisory guidance, a draft, or a reporting instruction.

For now, the rule I have given myself is simple:

**For legal conclusions, go back to EUR-Lex. For implementation questions, look at the ESAs and CSSF.**

 Some Important Documents to Keep

- [Regulation (EU) 2022/2554 — EUR-Lex](https://eur-lex.europa.eu/eli/reg/2022/2554/oj/eng)
- [Directive (EU) 2022/2556 — EUR-Lex](https://eur-lex.europa.eu/eli/dir/2022/2556/oj/eng)
- [European Commission — Implementing and delegated acts under DORA](https://finance.ec.europa.eu/regulation-and-supervision/financial-services-legislation/implementing-and-delegated-acts/digital-operational-resilience-regulation_en)
- [ESMA — Digital Operational Resilience Act](https://www.esma.europa.eu/esmas-activities/digital-finance-and-innovation/digital-operational-resilience-act-dora)
- [EIOPA — Digital Operational Resilience Act](https://www.eiopa.europa.eu/digital-operational-resilience-act-dora_en)
- [EBA — Digital Operational Resilience Act](https://www.eba.europa.eu/activities/direct-supervision-and-oversight/digital-operational-resilience-act)
- [CSSF — ICT and cyber risk for DORA entities](https://www.cssf.lu/en/ict-and-cyber-risk-for-dora-entities/)