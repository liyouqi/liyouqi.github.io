---
title: "Vulnerability Analysis: Network Security in Fintech: 2.Deploying Tenable at Scale"
date: 2026-04-21
categories:
- Cybersecurity
tags:
  - Cybersecurity
  - Network Security
  - Fintech



layout: single
author_profile: true
read_time: true
comments: false
share: true
---

This time I'm talking about what happens after last time get Tenable working in a lab. This is about deploying it in a real environment, which is a completely different thing. It's not just running a scan and reading the report. It's about keeping the platform stable, managing hundreds of assets across multiple countries, integrating it into business processes, and dealing with the inevitable edge cases.

I'm working on an internal fintech project, so I'll keep the details vague—company names, network layout, asset count, and specific policies are all off limits. What I'll cover here is the approach, the problems I ran into, and how I handled them.

If the first post was about getting a demoo scanner to work, this one is about running it in production. The difference isn't the tool—it's that you're suddenly dealing with system diversity, network segmentation, permission headaches, time sync issues, plugin updates, policy exceptions, and reporting schedules all at once.

## Why Tenable

We picked Tenable because it fit what we needed, not because it's the latest trendy tool.

We needed a platform that runs continuously, not a one-off scanner. Nessus does scanning well, and Tenable.sc lets you manage assets, policies, tasks, results, and reports in one place. When you need to track trends, organize groups, reschedule scans, and maintain audit trails, having everything in one system saves time.

What mattered to us:

- Fast onboarding. The deployment path is straightforward.
- Single pane of glass for managing everything. Good for coordinated operations.
- Plugins and policies stay fresh. You're not locked into outdated scanning rules.

The real win is that it turns scanning from a one-time task into something sustainable. You can also debug problems cleanly—when something goes wrong, you can usually pinpoint whether it's the scanner, the manager, the network, authentication, or the Linux box itself. You don't end up in a black box.

## Deployment Architecture

We're running a standard setup: one manager, two scanner nodes, plus a network of assets being gradually brought online.

The roles are clear:

- The manager handles policy distribution, result aggregation, reports, and access control.
- Scanner nodes do the actual scanning.
- Assets get onboarded gradually by country, business unit, and environment.

In theory this is straightforward. In practice, the first problems aren't with Tenable—they're with Linux and your network setup.

### Handling SELinux

We disabled it early on. Not because SELinux is bad in general, but in this setup it caused more friction than benefit. Installation kept hitting permission walls, services wouldn't start, port binding failed, directory access was inconsistent. You'd spend an hour debugging only to realize SELinux policy was the culprit all along.

The pragmatic move: confirm the system baseline, then dial back unnecessary restrictions. This lets Tenable services start consistently. Otherwise you'll waste days on permission and context issues.

### Time Sync

This one looks trivial but isn't.

When you have branches in twenty countries, time skew causes real problems. Scan scheduling gets confused, logs become useless, results sort wrong, historical reports don't make sense. If the manager says a scan happened at 10am UTC but the scanner thinks it's 2pm local time, everything falls apart.

Check:

- Is the system time actually correct?
- Is NTP or chrony running?
- Are timezones consistent?
- Is there drift between manager and scanner nodes?

Sometimes you'll see what looks like a scheduling bug when really it's just the timestamps are misaligned.

### Common Linux Problems

These are basic issues, but they eat time:

- Service won't start: check `systemctl status` and `journalctl -xe`.
- Port isn't listening: run `ss -lntp` to see what's actually bound.
- File permissions wrong: verify `chmod`, `chown`, and mount points.
- Firewall blocking: check `firewalld`, `iptables`, or your cloud security group.
- Hostname or DNS mismatch: will keep manager and scanner from finding each other.

None of this is complicated, but any one of these can cost you half a day.

## Managing the Plugin Repository

After deployment, the next big job is the plugin repository.

This isn't just importing assets. It's organizing your entire scanning universe—targets, policies, plugins, exceptions—into something you can actually maintain over time. The work is tedious and error-prone.

You're dealing with: databases, app servers, file services, office systems, middleware, legacy boxes, test environments, isolated segments. Each needs different scanning approaches.

My approach:

1. Group assets by type.
2. Layer by environment: production, test, office, edge networks.
3. Bind different scanning templates to each group.
4. Exclude targets that don't need monitoring.
5. Tailor plugin sets to asset types.

The problem is this isn't a one-time job. Assets change. Systems get updated. Networks get redrawn. What you organized last month might be half wrong by now. You need to circle back and clean it up regularly.

Plugin updates follow the same logic. Vulnerabilities are discovered constantly. If your plugins are stale, you're scanning with last month's threat intelligence. I schedule plugin updates as part of regular maintenance—non-negotiable. At minimum, log which plugin version you ran with each scan, otherwise people will ask why you missed a vulnerability.

Keep the repository clean. Bad data in means bad data out.

## Tuning Scan Policies

Here's where it gets finicky. Policy isn't "pick a template and run it." You have to actually understand your target environment—network layout, ports, protocols, services, versions, auth setup, and available scan windows. Then decide what makes sense.

The mistake I see most is "scan everything, as hard as possible." That breaks things. Older systems can't handle aggressive probing. Business windows are narrow. If scans interfere with production, the security team becomes unpopular fast.

What I do:

- Light probing on systems that can't handle stress.
- Credential scanning where it's approved.
- Maintenance windows only for disruptive checks.
- Exclude high-impact plugins where needed.
- Some targets get partial scans, not full coverage.

The experience to build is knowing "adequate" from "overkill." Full coverage is nice. Relevant coverage is essential.

Then there are regional differences. Some offices have clean network segmentation—business tier, server tier, office tier, desktop tier, all separate. Those are easy to manage. You know the attack surface.

Others are a mess of unplanned connections—everything wired together, no clear boundaries, a hundred one-off exceptions. Those are painful. Every policy decision has to account for reachability, business impact, and false positives together.

This taught me something: vulnerability management isn't really a security problem. It's a network and operations maturity problem.

## Reading Scan Results

Here's where it gets interesting. Don't just stare at the vulnerability count. Look at the fields that matter: versions, exposure, exploitation prerequisites, plugin description, remediation steps, whether credentials were needed, whether remote code execution is possible.

The real questions: What's actually exposed? What can an attacker actually reach? What's noise? What affects the business?

I focus on:

- Remote exploitable services.
- Old software with known vulnerabilities.
- Weak credentials or default config.
- Anything touching money, identity, or user data.

Experience matters here. The same medium-risk item means nothing on a test box but changes everything if it's running your transaction processing. A seemingly minor issue becomes major if it's in a critical network segment or can be used to move laterally.

So I stopped treating scan reports as technical checklists. I treat them as business risk inputs. What does the system actually do? Will fixing this affect operations? Will this impact billing, reporting, settlement? Context matters.

## Fixing and Verifying

This is the part that actually reduces risk.

How I approach it:

1. Confirm the vulnerability is real, not a false positive.
2. Decide: upgrade, patch, configure, or temporarily mitigate?
3. Coordinate with the business on timing.
4. Fix it.
5. Rescan and verify it's gone.
6. Document what changed.

The biggest trap: fixing a vulnerability then thinking you're done. Often it comes back. Configuration gets reverted. A new container gets deployed from an old image. A script overwrites your change. Patch gets rolled back during a rollout. If you don't verify, you end up thinking you fixed something that's still broken.

What matters is the loop: find -> assign -> fix -> verify -> close. All of it. If you skip verification, you're not actually managing risk.

## Reports and Distribution

The platform eventually has to produce reports that people actually use.

Different people need different things. Engineers want detail. Management wants trends. Compliance wants evidence. Auditors want an audit trail.

So reports need layers:

- Detailed findings for the technical team.
- Risk summary for leadership.
- Evidence package for compliance/audit.
- Trend reports for ongoing tracking.

Frequency matters too. Critical items need immediate escalation. Low-risk items go into weekly rolls. In-progress fixes get flagged separately. If you dump all findings at once, the important ones get lost in the noise.

## Compliance

In Europe, you can't ignore GDPR and DORA. They don't tell you how to scan. But they shape how you work.

Data retention, access control, evidence preservation, fix deadlines, supply chain audits, incident response—all of it becomes mandatory. You can't just say "we know about it." You have to show how you identified, assessed, responded, and verified.

This means process and documentation. Every step has to be auditable. Scan results aren't the end—follow-through is.

## What I've Learned

Tenable isn't hard to set up. Making it work in production is. There are a lot things to consider: architecture decisions, repository maintenance, policy tuning, result analysis, remediation coordination, reporting, compliance mapping.

Each piece isn't complex. But together they require care and consistency. The goal is to make every step repeatable, auditable, and documented. That's what keeps the platform useful.

The next step will be about turning this into a more robust workflow: how to prioritize in our multiple country teams, assign work, set SLAs, track execution, and make it something your team can actually run.
