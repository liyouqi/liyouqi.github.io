---
title: "Learning DORA 03 | If the System Is Outsourced to a Cloud Provider, Whose Problem Is It When Things Go Wrong?"
date: 2026-04-29
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
---

Today AI started by asking me a question.

Suppose a bank hosts its entire online banking system on AWS. One day AWS has an outage, and the bank's online banking service goes down for several hours.

Whose responsibility is it?

My instinct was: the bank's.

If you are the customer buying the service, you still have to take responsibility for what you chose to buy. You cannot simply push everything onto the supplier.

The bank chose AWS. The customers are the bank's customers. If the system goes down, the bank obviously cannot just tell the regulator:

> "This is an AWS problem. Nothing to do with us."

But as I kept learning, I realised that this very intuitive answer actually leads straight into one of the more important ideas in DORA:

You can outsource the service, but you cannot outsource the responsibility.

But, but, but... it is not quite that simple either.

## DORA is not only about banks

After finally figuring out those ten-plus DORA documents in the previous post, today I moved on to the question of who DORA actually applies to.

At first I thought this would be easy too.

DORA is a financial regulation, so banks, brokers, funds, insurance companies... something like that.

Then Article 2 gives you this huge list.

Honestly, I had no idea what to do with it. ("No idea what to do" has recently become a catchphrase of a Tsinghua intern around me.)

The list includes banks, payment institutions, electronic money institutions, investment firms, financial market infrastructures, fund managers, insurance companies, pension institutions, credit rating agencies, crypto-asset service providers, crowdfunding platforms, and quite a few others.

Even ICT third-party service providers are pulled into the wider DORA framework.

So DORA is not really a "banking law".

It feels more like DORA draws a big circle around the whole financial ecosystem.

```text
Banks
Payment institutions
Securities and investment firms
Funds
Insurance
Pensions
Crypto
Crowdfunding
Financial market infrastructures
        ↓
all increasingly depend on ICT
        ↓
Cloud / SaaS / Data / Network / Outsourcing Providers
```

Modern financial institutions can hardly operate independently from external technology companies anymore.

Banks use cloud services, buy SaaS products, outsource systems, and sometimes have critical business functions that depend heavily on third parties.

Which leads to the obvious question:

If the technology is actually being provided by someone else, who is responsible when something breaks?

## The system can run on AWS, but the responsibility still belongs to the bank

DORA is actually quite direct about this.

Using an ICT third-party provider does not remove the financial institution's responsibility for complying with DORA.

That logic makes sense to me.

Suppose the bank's online banking platform runs on AWS:

```text
Customer
   ↓
Bank online banking
   ↓
Bank application
   ↓
AWS Cloud
   ↓
Other technical dependencies behind AWS
```

If AWS has an outage, AWS obviously has responsibilities of its own.

For example:

- Why did the service go down?
- Was the SLA met?
- How long did recovery take?
- Did AWS notify the bank in time?
- Did it provide enough information?
- What was the root cause?
- What will be fixed afterwards?

But the bank has a completely different set of questions to answer:

- How many customers were affected?
- Which transactions failed?
- Was a critical or important function affected?
- Did the incident meet the threshold for a major ICT-related incident?
- Does it need to be reported to CSSF?
- Why was this supplier selected in the first place?
- Was the risk assessed properly?
- Is there an alternative?
- If AWS cannot recover for a long time, what is the bank going to do?

So from a regulatory point of view, "AWS went down" does not automatically become an excuse for the bank.

The question can easily come back the other way:

If this supplier is so important to you, why was this risk not identified and managed earlier?

This is probably the sentence I remember most from today:

> Outsourcing the service does not outsource the responsibility.

The service is outsourced. The regulatory responsibility is not.

Of course, this is not a binary answer where the responsibility has to be either 100% yours or 100% mine.

A company like AWS is not just an ordinary supplier.

And that brings me to another acronym I had seen many times but never really understood:

CTPP — Critical ICT Third-Party Provider.

A critical ICT third-party service provider.

A provider does not automatically become a CTPP simply because it is large.

It has to be formally assessed and designated by the three ESAs — EBA, ESMA and EIOPA — according to the criteria set out under DORA.

Why build another layer like this?

After thinking about it, it actually makes sense.

Suppose only one bank uses a small provider:

```text
Bank A
  ↓
Provider X
```

If Provider X goes down, Bank A is probably the main institution affected.

That is mainly Bank A's own third-party risk management problem.

But now imagine this:

```text
Bank A ─┐
Bank B ─┤
Bank C ─┤
Bank D ─┤
Insurer ├────→ One major Cloud Provider
Fund    ─┤
Broker  ─┘
```

Now the problem looks very different.

If dozens, or even hundreds, of important financial institutions across Europe depend on a small number of technology providers, then a serious outage at one of those providers may no longer affect just one bank.

It may start affecting the operational resilience of the financial system itself.

At that point, relying only on each bank to manage the supplier individually is probably not enough.

So DORA adds another layer on top:

```text
EBA / ESMA / EIOPA
        ↓
   Lead Overseer
        ↓
      CTPP
        ↓
provides ICT services
        ↓
many financial institutions
```

Once an ICT provider is formally designated as a CTPP, it enters the EU-level oversight framework created by DORA.

A Lead Overseer can assess its ICT risk management, governance, incident handling and business continuity arrangements, and can also request information, conduct investigations and carry out inspections.

In November 2025, the ESAs officially published the first list of designated CTPPs.

I looked through it, and quite a few of the names were very familiar.

### List of designated CTPPs

<!-- Insert screenshot of the List of designated CTPPs here -->

What I found interesting about that list is the change in perspective.

When I used to look at companies such as AWS, Google Cloud or Microsoft, I mainly thought of them as cloud providers or technology platforms.

But from DORA's point of view, once a large number of financial institutions build important business functions on top of their infrastructure, these companies are no longer just ordinary IT suppliers.

They start becoming part of the operational resilience of the financial system itself.

## If a CTPP is already supervised by the EU, does that make life easier for the bank?

Not really.

If AWS or another major provider is already directly supervised as a CTPP, it is tempting to think that the bank can relax a little.

But these are actually two parallel responsibilities.

```text
EU supervisory authorities
        ↓
supervise whether the CTPP itself is resilient enough

at the same time

Bank
        ↓
continues to manage the risks created by its own use of that CTPP
```

The bank still needs to know:

- Which services it actually uses from the provider
- Which business functions depend on those services
- Whether any critical or important functions are involved
- Where the data is stored
- Whether subcontractors are involved
- How concentrated the dependency is
- How long the bank can survive if the service goes down
- Whether alternatives exist
- Whether the bank can actually exit the relationship

So CTPP oversight does not mean that the EU is doing third-party risk management on behalf of the bank.

These are two different questions.

One is:

> Could this large provider create systemic risk for the European financial system?

The other is:

> Why is your bank so dependent on this provider, and what are you doing about that dependency?

That is the key point.

Another part I found quite reasonable is that DORA does not treat every financial institution exactly the same.

Article 2 brings a huge range of financial institutions into the framework.

But then another question appears.

A large international bank operating in dozens of countries with hundreds or thousands of systems obviously cannot be treated exactly the same as a small financial institution with a simple business model.

A good example is the Chinese multinational bank where I currently work. Back in China it is a giant, but overseas it may only have one presence in each country's capital and mainly provide corporate banking services.

Same banking group, very different local scale and complexity.

Article 4 introduces the Proportionality Principle.

In simple terms, when implementing ICT risk management requirements, the institution should consider things such as:

- Size
- Overall risk profile
- Nature of services
- Scale
- Complexity of services, activities and operations

So the institution's size, overall risk profile, and the nature, scale and complexity of its business and operations all matter.

My understanding is that this does not mean:

> Small institutions can simply comply less.

It means:

> The regulatory objective is the same, but the way you achieve it does not have to look exactly the same.

I made a simple comparison for myself.

This is not an official DORA classification or a regulatory template. It is just my own interpretation of the proportionality principle in Article 4.

| Scenario | Large cross-border bank | Medium-sized financial institution | Smaller institution with relatively simple business |
|---|---|---|---|
| Business and systems | Multiple countries, entities, large numbers of systems and complex dependencies | More limited number of systems and business lines | Fewer business lines and a relatively simple technical architecture |
| ICT governance | May require dedicated committees and multiple risk/control functions | Can use a leaner governance structure | Organisation may be compact, but responsibilities still need to be clear |
| Assets and dependencies | Often needs mature CMDB, service mapping and dependency management | Can manage through a central asset repository and business mapping | Smaller data volume, but still needs to know critical assets and dependencies |
| Monitoring | Needs coverage across large numbers of systems, suppliers, regions and business chains | Monitoring can focus on higher-risk systems | Simpler monitoring may be enough, but "small" does not mean "no monitoring" |
| Testing | Testing programmes are usually broader and cover more scenarios and systems | Scope can be designed around critical services and risk | Scope may be smaller, but the institution still needs to demonstrate resilience |
| Third-party management | More suppliers, more concentration risk, more complex subcontracting chains | Can focus more heavily on critical suppliers | Fewer suppliers may actually mean stronger dependence on a small number of them |
| Evidence and audit | Usually needs a more institutionalised and automated evidence framework | Standardised processes may be enough | Processes may be simpler, but the institution still needs to prove that controls were actually performed |

The regulator is not asking every institution to buy the same tools or build the same size team.

The responsible functions still need to ask:

Given our own scale, risk profile and complexity, are these controls actually enough?

For example, if a small institution only has a dozen critical systems, a well-maintained asset register may be perfectly reasonable.

If a multinational bank still relies on ten different Excel sheets and a lot of manual reconciliation just to understand its assets, it becomes much harder to defend the statement:

> "But we do have an asset inventory."

The same control can look very different in different institutions.

## So who does DORA actually regulate?

After going through this, I honestly do not think there is much value in memorising the long list in Article 2.

The logic behind the list is more important.

The financial industry is no longer:

```text
Bank
builds its own systems
runs its own systems
carries its own risks
```

It increasingly looks like this:

```text
Financial institution
   ↓
large amounts of applications and data
   ↓
Cloud / SaaS / Network / Data / Outsourcing
   ↓
Subcontractors
   ↓
more infrastructure underneath
```

Financial institutions and technology providers are now deeply connected.

So DORA does two things at the same time.

On one side, it requires financial institutions to build their own digital operational resilience.

On the other side, it brings the ICT supply chain itself into the regulatory framework.

Ordinary third parties are managed by the financial institution.

Particularly critical third parties get another layer of direct EU oversight.

But whether a supplier is designated as a CTPP or not, the bank's own responsibility does not disappear.

I think I can reduce today's lesson to three simple points.

First, DORA regulates the wider financial ecosystem, not just banks.

Second, services can be outsourced, but responsibility cannot simply be outsourced with them.

Third, DORA applies the proportionality principle. Different institutions are expected to achieve the same regulatory objectives, but the way they implement controls does not have to look exactly the same.

If a regulation completely ignored institutional size and business complexity, it could easily turn into another exercise where everyone builds the same paperwork just to satisfy the rule.

At least in its design, DORA recognises something quite practical:

Risk management was never supposed to look exactly the same everywhere.

So...

## Files I touched today

- Regulation (EU) 2022/2554 — Article 2: Scope
- Regulation (EU) 2022/2554 — Article 4: Proportionality principle
- Regulation (EU) 2022/2554 — Chapter V: Managing of ICT third-party risk
- Regulation (EU) 2022/2554 — Article 31: Designation of critical ICT third-party service providers
- Commission Delegated Regulation (EU) 2024/1502 — criteria for the designation of CTPPs
- ESAs — List of designated Critical ICT Third-Party Providers