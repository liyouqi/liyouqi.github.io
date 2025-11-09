---
title: "Understanding Diamond (1996): Why Banks Exist — A Theory of Financial Gravity"
date: 2024-12-03
categories:
  - Financial Study Notes
tags:
  - Banking Theory
  - Financial Intermediation
  - Economic Logic
  - Fintech
layout: single
author_profile: true
read_time: true
comments: false
share: true
---


# Introduction — Why apples fall and banks exist

When an apple falls from a tree, no one is surprised.
It feels obvious, almost trivial.
But before Newton, that motion had no unified explanation — it was merely *what apples do*.
Newton changed that: he turned an everyday fact into a universal law, showing that gravity was not a mystery, but a **systemic force** that shaped the world.

Banks, too, seem obvious. They accept deposits, make loans, and sit quietly at the center of the economy.
We treat them as fixtures of capitalism, as natural as falling apples.
Yet before **Douglas Diamond’s model of delegated monitoring**, few asked *why* banks must exist at all.
Were they historical accidents, or the natural outcome of deeper economic laws?

Diamond’s 1996 teaching paper — a simplified version of his 1984 classic — does for banking what Newton did for motion.
It doesn’t just describe what banks do; it explains *why they must exist*.
It shows that when trust is costly and information is uneven, the gravitational pull of incentives and efficiency inevitably brings banks into being.

---

# 1. The natural force behind intermediation

In Diamond’s world, the invisible force is **information asymmetry**.
Borrowers know whether their projects succeed or fail; lenders do not.
Without verification, a borrower can easily say, “the project failed,” and keep the profit.
Lenders, unable to tell truth from deceit, must design contracts to protect themselves.

Imagine a single borrower needing one unit of capital.
The project pays:

* 1.4 if successful (probability 0.8)
* 1.0 if it fails (probability 0.2)

Investors require an expected repayment of 1.05 (a 5% return).
Without monitoring, they must demand a fixed promised payment ( f ):

$$
0.8 f = 1.05 \Rightarrow f = 1.3125
$$

The borrower must promise to repay 31% more than borrowed.
This high rate is not due to productive risk, but **informational risk** — the cost of distrust.
Like lifting an apple upward, it takes extra energy to fight the pull of uncertainty.

---

# 2. Monitoring: the counterforce to mistrust

Now suppose the lender can **monitor** the borrower at a cost ( K ).
Monitoring reveals whether failure was genuine or fake.
By avoiding unnecessary liquidation, the lender can save part of the wasted output — say ( S = 0.2 ).

If ( K < S ), monitoring is socially efficient.
But if every small investor monitors individually, the total cost multiplies by the number of lenders ( nK ).
This is the first key insight:

> decentralized trust is expensive; collective trust is cheaper.

At this point, the economic “gravity” becomes clear.
There’s a natural pull toward **delegation** —
to appoint one agent to monitor on behalf of all.

---

# 3. Delegated monitoring: organizing the gravitational field

Suppose investors create a single intermediary — a *bank* — that collects deposits and monitors borrowers.
Instead of each saver spending ( K ), they now share one monitoring cost.
The system economizes on effort and achieves the same discipline.

But now a new problem arises:
if depositors no longer monitor the bank, what ensures that the bank itself monitors borrowers?
Who watches the watcher?

Diamond’s elegant answer is **debt**.
If the bank finances itself with short-term deposits that can be withdrawn or trigger liquidation,
it has strong incentives to behave — to actually monitor borrowers and avoid default.

The deposit contract acts like gravity:
an invisible but persistent force keeping the bank’s behavior within orbit.
If it drifts — stops monitoring, takes hidden risks — liquidation pulls it back to discipline.

Hence the name *delegated monitoring*:
the bank monitors borrowers, and the deposit contract monitors the bank.
A self-stabilizing chain of incentives emerges — a financial solar system bound by the force of trust and threat.

---

# 4. Diversification: how mass creates stability

A single monitored loan is still risky.
Even with honest monitoring, some projects fail, and the bank may face liquidation.
But if the bank lends to **many independent borrowers**, things change dramatically.

Consider two uncorrelated loans:

* both succeed with probability ( 0.64 )
* one fails with ( 0.32 )
* both fail with ( 0.04 )

The chance of the bank’s total failure drops from 20% to 4%.
Each new loan adds diversification — a kind of **mass** that stabilizes the system.
In the limit of infinitely many loans:

$$
\text{Default probability} \to 0
$$

Now the bank becomes nearly risk-free; its survival no longer depends on random outcomes but on structure.
Monitoring costs per loan approach zero, and depositors can safely lend without watching at all.

The analogy deepens:

> diversification in finance plays the same role as mass in physics — it stabilizes through aggregation.

---

# 5. The equilibrium of financial gravity

Step back and look at the entire model.
Each element aligns like planets in a gravitational field:

| Force                 | Role                            | Economic effect     |
| --------------------- | ------------------------------- | ------------------- |
| Information asymmetry | Creates the need for monitoring | Trust deficit       |
| Monitoring            | Reduces deception               | Costly but valuable |
| Delegation            | Concentrates monitoring         | Saves social cost   |
| Debt contracts        | Discipline intermediaries       | Incentive alignment |
| Diversification       | Reduces failure probability     | Systemic stability  |

Together they form a **law of financial gravity**:
if you start with rational agents, asymmetric information, and costly monitoring,
the stable equilibrium — the system that economizes on trust — **is a banking system**.

Even without regulation or history, such institutions would spontaneously form.
The same way gravity organizes matter into orbits, these forces organize finance into intermediated structures.

---

# 6. Policy and intuition: when structure defies gravity

Gravity cannot be repealed, but it can be misread.
For decades, policymakers tried to make banks “safer” by limiting their scope —
restricting them to local lending, forbidding interstate operations,
or separating deposit-taking from investment.

Yet in Diamond’s framework, these restrictions *increase fragility*.
By reducing diversification, they make banks smaller masses, more easily displaced.
The lesson is counterintuitive:

> stability comes from scale and diversification, not isolation.

This explains why integrated banking systems — those allowed to pool risks across borrowers, sectors, or geographies —
tend to survive shocks better than fragmented ones.

It also clarifies why shadow banking and unregulated P2P lending remain inherently unstable:
they mimic the outward form of intermediation, but lack the gravitational discipline of delegated monitoring and debt-based incentives.

---

# 7. Reflection: from gravity to FinTech

Diamond’s insight has aged remarkably well.
Even in the FinTech era, where platforms promise “disintermediation,”
the core functions quietly reappear:

* **Delegation of trust** (users rely on platform credit scoring),
* **Pooling of risk** (funds spread across many borrowers),
* **Contractual discipline** (automated defaults, platform control).

In short, the gravitational law still holds —
the names change, the orbits evolve, but the underlying pull remains.

True disruption in finance would mean finding a new kind of gravity —
a new way to organize trust and monitoring without losing stability.
So far, no one has escaped it.

---

# Epilogue — Why apples fall and banks exist

Just as Newton revealed that the apple’s fall obeys a universal law,
Diamond revealed that banking is not a historical accident but a logical consequence.
Whenever monitoring is costly and information uneven,
there will be a force pulling economic agents into intermediation.

Banks exist not because someone built them,
but because the world would fall apart without them.

> **They are the equilibrium outcome of human mistrust — the gravity that keeps finance from drifting away.**

---



# Reference

Diamond, Douglas W. (1996). *Financial Intermediation as Delegated Monitoring: A Simple Example*.
Graduate School of Business, University of Chicago.

---

> **“Banks exist not by design, but by gravity.”**
> — This is my reinterpretation of Diamond’s law


