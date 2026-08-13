---
title: "Learning DORA 04 | A Vulnerability Stayed Unpatched for Three Months. Something Happened. Who's to Blame?"
date: 2026-04-30
categories:
- Cybersecurity
tags:
- Cybersecurity
- Fintech
- DORA
- ICT Risk
- Vulnerability Management
- Risk Governance
- Internal Audit
- Digital Operational Resilience Act
---

Let's imagine Tenable finds a high-risk vulnerability on a core system. IT confirms that it needs to be fixed, but patching it requires a two-hour outage. The business says it is settlement period and the system cannot go down.

So it gets postponed once.

Next month, still cannot stop it.

The month after that, postponed again.

Three months later, the vulnerability is actually exploited and turns into a security incident. So who should be blamed?

My first reaction was: IT definitely has responsibility, and then management also has responsibility because they failed to supervise properly. I guess this is probably the first reaction most people would have. Normally, vulnerabilities belong to IT. If it wasn't fixed, then surely that's an IT problem.

But after going deeper into it, I realised it is not that simple. The four words "the vulnerability wasn't fixed" tell you almost nothing about what actually happened in between.

Maybe IT never knew about it. Maybe they knew and forgot. Maybe they wanted to fix it but the business refused downtime. Maybe the risk had already been assessed and management decided that taking the system down at that moment would create an even bigger business risk, so they chose to accept it temporarily.

All of these situations look exactly the same at the end:

> The vulnerability remained unpatched for three months.

But they are completely different things.

## Finding a vulnerability does not mean IT gets to decide everything

Suppose Tenable finds the vulnerability and IT and Security finish their analysis:

> High Risk. Recommended remediation within two weeks.

Normally this should be simple: arrange a maintenance window, patch it, validate it, close it.

But reality is often not that cooperative. The business may simply say:

> We cannot stop the system now.

And that can be a perfectly reasonable answer. Maybe it is month-end settlement, peak customer transaction time, a major production launch, or the system itself supports an important business service.

Then you get an interesting problem.

Fixing the vulnerability creates risk. Not fixing it also creates risk.

```text
Fix it now
→ possible business interruption

Do not fix it now
→ continue to carry the security exposure
```

So "risk management" is very often not about eliminating risk.

It is about choosing between two options that both make you uncomfortable.

IT can tell you how technically serious the vulnerability is. Security can tell you where the attack surface is.

But IT does not automatically have the authority to decide:

> In order to patch this vulnerability, we are willing to take the whole business offline for two hours.

That is no longer a purely technical decision.

## So who gets to say: let's accept this risk for now?

I had never really thought about this carefully before.

In many companies, the real-life process can be extremely casual.

Business says:

> We are busy recently. Let's patch it next month.

IT says:

> Fine. Why not take it easy if we can.

Then the Due Date in Excel moves from May to June.

June arrives.

Still no.

Move it to July.

It looks like everyone is "tracking remediation", but in reality nothing has happened.

Studying DORA made me realise that the real question may not be:

> Why hasn't this vulnerability been fixed?

The real question is:

> Who decided that it was allowed to remain unfixed?

Those are two completely different questions.

If the decision is to postpone remediation, then at the very least it should become a formal risk decision.

This is one of the management points I picked up this time and I think it is actually quite important: managing the risk decision itself.

Something like:

```text
Vulnerability identified
   ↓
Confirmed that it cannot be fixed on time
   ↓
Explain why
   ↓
Reassess the risk
   ↓
Any temporary controls?
   ↓
Who accepts the risk?
   ↓
For how long?
   ↓
When will it be reviewed again?
```

DORA obviously does not hand every bank a standard "vulnerability postponement approval form".

But it does require ICT risk management to have governance, ownership, records and oversight.

So an Exception should not mean:

> Whatever, let's look at it again next month.

It should become a real identified risk. Someone knows about it. Someone accepts it. And that acceptance should have an expiry date.

## But there is another trap here: signing off on a risk does not mean signing up to take the blame

Suppose IT has already clearly stated:

> High-risk vulnerability. Recommended remediation within two weeks.

The Business Owner formally decides:

> Because of quarterly settlement, we accept this risk for three months.

At the same time, some temporary controls are put in place: network isolation, restricted access, enhanced monitoring.

Then in the second month, the system still gets attacked.

Now think about it:

> Isn't this management failure? You signed off on the risk.

That was also my first thought.

But later I realised that this interpretation is not entirely right either.

Accepting risk basically means:

> I know this may happen, but based on the information I have now, I decide to carry this risk temporarily.

Risk management is not fortune-telling.

If every risk that eventually happens automatically proves that the original decision was wrong, then risk management becomes impossible.

It would turn into:

```text
Nothing happened
→ correct decision

Something happened
→ wrong decision
```

That is just hindsight.

So a more accurate way to put it is this:

Management or the Risk Owner is accountable for the decision, but accountability does not automatically mean failure.

And this line is genuinely difficult to manage, especially in places where the culture is basically: never be the one holding the bag. You know what I mean.

English actually makes the distinction quite nicely:

> Accountability is not the same as fault.

What should really be reviewed afterwards is: why was the risk accepted in the first place? Was it already outside the company's risk tolerance? Was three months a reasonable period? Did the temporary controls actually work? Who approved it? Did that person have the authority? Was the risk continuously monitored after acceptance?

If all of that was handled reasonably and the risk still happened, then the most accurate description is probably:

> An accepted residual risk actually materialised.

That is completely different from "nobody managed it and then something randomly blew up".

I did not have this distinction very clear before.

I actually think DORA is designed quite well in this area. I have to admit, quite a few European and American management concepts are still ahead in this kind of governance thinking.

## The Management Body is not sitting there approving CVEs every day either

We already touched the management body in Day 1 and the previous post.

DORA Article 5 puts the ultimate governance responsibility for ICT risk on the management body.

But if you interpret that as:

> Every high-risk vulnerability needs to be sent to the board for signature.

Then the board would have nothing else to do all day except approve CVEs.

The management body is not supposed to manage every individual technical issue.

It should manage the mechanism.

For example:

What kinds of risks can be accepted by a System Owner?

Which ones need Risk approval?

Which risks must be escalated once they cross a threshold?

How long can remediation be postponed?

When must something be reported to Senior Management?

Should critical or important functions have stricter requirements?

These rules should already exist before the problem happens.

So a reasonable structure might look like this:

```text
Normal vulnerability
→ IT / Security handles it

Cannot remediate on time
→ System / Business Owner

Serious risk or long-term delay
→ Risk / Security Committee

Above risk tolerance
→ Higher management
```

Whether the number is 30 days, 60 days or 90 days is not that important to me.

Different institutions can define it differently.

The important question is:

> What actually happens after the deadline passes?

This is a surprisingly difficult question.

A lot of spreadsheets have a Due Date.

But what happens after the Due Date?

Sometimes the answer is:

> Send another email.

One month later:

> Send another email.

Another month later:

> Remind them again.

That is tracking. I am not sure it is really governance.

If governance is actually working, then the longer the issue remains unresolved, the higher it should move through the organisation, instead of sitting forever in the same row of the same Excel file.

## I finally understand the Three Lines of Defence a little better

I had seen the Three Lines of Defence many times before.

To be honest, I always thought it was one of those organisational concepts that banks and auditors particularly enjoy inventing.

But once I put this vulnerability example into it, it suddenly made much more sense.

The first line is usually the business, IT and other teams that actually own and operate the risks.

IT fixes the vulnerability, the System Owner owns the system, and the business owns the business impact.

The second line is Risk, Compliance, or some form of Security oversight.

They are not there to patch servers for IT every day. They ask questions like:

> Why has this vulnerability already been overdue for 90 days?
> You said it cannot be fixed. Where is the formal justification?
> Who accepted the risk?
> Where are the temporary controls?

The third line is Internal Audit.

Audit can be even more brutal.

They may not even care what the CVE technically is.

They might randomly sample ten high-risk vulnerabilities and find something like:

```text
Vulnerability identified: January
Remediation deadline: February
Current date: August
Status: Open
Reason: Business unavailable
Risk acceptance: None
Compensating control: No evidence found
Escalation: None
Owner: Already left the company
```

At that point, there is really no need to keep studying the CVE.

The process itself already tells the story.

Audit can simply ask:

> Is your Vulnerability Management process actually working?

So Internal Audit is not there to run a second vulnerability scan for Security.

It is checking:

> Are all those controls you claim to have actually operating, or do they only exist in the policy document?

As my understanding got a little deeper, I suddenly realised something. Or maybe I already knew it, I just didn't know that I knew it:

A lot of the spreadsheets and records we maintain every day are actually evidence.

And I mean evidence, not just "register management".

This is actually very close to what I am doing at work now.

Vulnerability tracker.

PDCA.

Remediation records.

Emails.

Tickets.

Meeting minutes.

Risk acceptance.

Sometimes these things are genuinely annoying.

A technical issue exists. Why not just fix it and move on? Why do we need to keep so many things around afterwards?

But looking at it from a DORA perspective, I am starting to understand why they exist.

Because what regulators or Audit eventually ask is often not:

> Did you discuss this at the time?

It is:

> Show me the evidence.

Then the questions become:

```text
Who decided?

When?

Based on what?

For how long?

Who approved it?

What temporary controls were applied?

Was it reviewed again?

How was it finally closed?
```

You say:

> We had a meeting about it.

Doesn't mean much.

You say:

> Security already warned the business.

Where is the evidence?

The business says:

> We knew about the risk at the time.

Who accepted it?

So in the end, all these things that look annoyingly administrative are actually recording one thing:

> What happened to a risk from the moment it was identified until the moment it was finally closed?

Thinking about it now, this may be one of the real cores of GRC work.

It is not about making the Excel sheet look nicer.

It is about making sure that somebody can still reconstruct the entire decision trail later.

## So if a vulnerability remains unpatched for three months, who should be blamed?

Now if I go back to the original question, I can no longer simply answer:

> IT.

And I cannot simply answer:

> Management.

You need to pull out what happened in between.

If IT never identified it, never tracked it and never handled it, then yes, that is first of all an IT control problem.

If IT proposed remediation, but the business kept refusing, and there was no formal risk acceptance, no compensating control and no escalation, then the entire governance chain failed.

If the risk was formally assessed, accepted by someone with the right authority, had temporary controls, had an expiry date and was continuously reviewed, and then something still happened, you cannot simply work backwards from "an incident happened" and conclude that the whole risk management process must have failed.

Of course the incident still needs a post-mortem.

You may even discover that the original assessment model was wrong, the risk tolerance was too loose, or the temporary controls were not as effective as everyone thought.

But that is another question.

I now think DORA puts ultimate responsibility on the management body not because it wants to say:

> Whatever goes wrong with a system, make the board take the blame.

It is saying:

> The organisation must have a mechanism that prevents major ICT risks from sitting there for months while nobody knows who made the decision.

The sentence I remember best today is not actually from the regulation.

It is this:

> A risk existing for a long time does not automatically mean risk management has failed. What is really dangerous is when nobody knows why the risk is still there, who accepted it, or how long they intend to keep accepting it.

And another one:

> Risk acceptance is not risk elimination, and accountability is not the same as fault.

So, be informed, be accountable, and keep the evidence.