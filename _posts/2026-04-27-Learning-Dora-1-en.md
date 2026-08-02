---
title: "Learning DORA 01 | How a System Outage Can Lead to an Entire EU Regulatory Framework"
date: 2026-04-27
categories:
- Cybersecurity
tags:
  - Cybersecurity
  - Network Security
  - Fintech
  - DORA
  - Vulnerability Management
  - Digital Operational Resilience Act

layout: single
author_profile: true
read_time: true
comments: false
share: true
---

Banking was an industry I had never worked in before.

I am currently working in information security at the European headquarters of a bank in Luxembourg. My daily work involves vulnerability management, asset inventories, security alerts, risk follow-up, and some audit-related checks.

This is also where I first heard a term that seems to come up in management discussions almost every day: **DORA**.

After looking into it briefly, I realised that this regulation is actually quite important. If I trace many of our current tasks back far enough, they somehow lead to DORA.

So I have decided to start learning it systematically.

Because my role is in cybersecurity, I initially assumed that DORA was basically an EU cybersecurity law for financial institutions.

It certainly covers cybersecurity, but its real concern seems to be much broader.

It is not only asking whether a bank can prevent an attack. It is also asking whether the bank can continue providing critical financial services when a system fails, a cyberattack occurs, a supplier becomes unavailable, or some other ICT problem happens.

Can the bank recover in time?

Are the responsibilities clear?

Are there proper processes in place?

Can the bank provide evidence showing what it actually did?

In other words, DORA is not only about “security”. It is about **digital operational resilience**.

I have also realised that, although I have already worked on several small parts of DORA-related activities, I still do not understand the framework as a whole.

For example:

- Why does the ultimate responsibility for ICT risk sit with management rather than the IT department? This feels slightly upside down to me. Anyone who has worked in a Chinese organisation probably knows what I mean.
- What kind of system outage becomes a major ICT-related incident?
- Where do vulnerability scanning, recovery exercises, and penetration testing sit within the DORA framework?
- If a bank moves a system to a cloud provider, does the related risk move to the provider as well?
- Why is a detailed supplier Excel file still not enough to satisfy the Register of Information requirements?

Clearly, memorising a few regulation numbers is not going to answer these questions.

To be honest, I could barely understand the text in the first place. My manager told me to study the regulation, but my brain more or less shut down on the first page.

That thing did not look like something written for ordinary human beings.

So I downloaded more than ten official DORA-related documents and decided to give them to AI, asking it to act as a DORA consultant and guide me through the framework step by step.

Of course, AI can also make things up.

Whether its explanation is correct still has to be checked against the actual regulatory text.

Today, I started with the main DORA Regulation.

Even finding the so-called “main regulation” took me a while to understand. At first, I thought DORA was simply one PDF. After going around in circles, I ended up finding more than ten related documents.

Eventually, I understood that **Regulation (EU) 2022/2554** is more like the main framework.

It establishes the overall structure, responsibilities, and basic requirements. A series of delegated regulations, implementing regulations, and supervisory guidelines then expand on specific areas such as incident reporting, ICT risk management, third-party risk, and resilience testing.

The task AI gave me today was relatively light: read Article 1 and Article 5 of the main regulation.

To help me understand them, it first gave me a simple scenario.

A bank’s payment system becomes unavailable for two hours because a supplier update fails. There is no cyberattack, no data breach, and the system is eventually restored.

Under my previous understanding, this might not even count as a cybersecurity incident.

It sounds more like an ordinary IT failure.

The technical team restores the system, investigates why the update failed, carries out a review, and the matter is more or less closed.

But after reading Article 1, I realised that DORA looks at the situation from a much broader angle.

Article 1 is titled **Subject matter**. It explains, at a high level, what DORA is intended to regulate.

Its concern is not limited to cyberattacks. It also covers ICT problems that may affect the operation of a financial institution.

In other words, even if there is no hacker and no data leakage, an ICT failure already falls within DORA’s area of concern if it disrupts the continued delivery of financial services.

The situation is therefore no longer just:

> “The server was down for two hours.”

More questions have to be asked:

How badly was the payment service affected?

Were customers unable to complete transactions?

Was a critical business function involved?

How long did the bank take to recover?

Could the same problem happen again?

Article 1 itself does not yet tell me whether the incident is serious enough to be reported to the regulator.

There are separate rules for incident classification and reporting, which I will study later.

But at least one point is already clear:

DORA covers a much wider area than cybersecurity in the traditional sense.

What surprised me even more was Article 5.

Article 5 places the ultimate responsibility for ICT risk at the level of the **management body**.

In a bank, this can be understood, roughly speaking, as the board or another body carrying the highest level of governance responsibility.

When I first read this, it honestly felt slightly upside down.

Anyone who has worked in a Chinese organisation will probably understand. Responsibility is usually pushed down to the operational level, and then pushed down again.

The systems are maintained by IT.

Security controls are implemented by the security team.

So why should management carry the final responsibility?

My current understanding is that the IT department can decide how to patch a vulnerability or how to restore a server, but it cannot independently decide how much risk the bank is willing to accept.

Which services must be recovered first?

How much budget should be invested?

Should the bank continue using a high-risk supplier?

How much business disruption is acceptable after a major incident?

These are not decisions that a technical department can make on its own.

Article 5 obviously does not mean that the board needs to repair servers personally.

IT and security teams still perform the technical controls and respond to incidents. Risk, compliance, and internal audit also have their own monitoring and assurance responsibilities.

But management cannot simply throw the entire subject of ICT risk to the IT department and then appear only after a serious incident to ask:

> “Why didn’t anyone discover this earlier?”

DORA requires the management body to define, approve, and oversee the ICT risk management framework, while remaining ultimately responsible for its implementation.

The work itself may be delegated.

The final governance responsibility cannot be delegated along with it.

I did not read very much on the first day—only Article 1 and Article 5.

But my main takeaway is already quite different from what I originally expected.

I initially thought DORA was mainly telling bank IT and security teams how to perform cybersecurity work.

Now it seems more accurate to say that DORA requires the entire bank to treat ICT risk as a business and governance issue, rather than merely a technical one.

After spending the last two years in Europe, I have definitely encountered this kind of governance thinking more often.

In China, organisations often seem more focused on growth.

That is enough for today.