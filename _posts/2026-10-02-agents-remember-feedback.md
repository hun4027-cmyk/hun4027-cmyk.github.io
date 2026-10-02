---
title: "Oct 2 — My agents now remember yesterday's feedback"
date: 2026-10-02
layout: post
---

I'm running an experiment: a small company staffed entirely by AI agents, built by one person. The goal is an operating system that keeps working while I sleep. Right now I'm not trying to make money from it. This stage is about building the system and getting good at it.

## What shipped today

**Departments carry yesterday's review into today's work.** Each department agent gets one study task a day in its field. Until now, if a reviewer sent yesterday's task back for rework, today's task had no idea why. Now the department's previous task and the reviewer's reasons are attached to the front of today's task. I want to see the effect as a number, not a feeling, so I locked a baseline before deploying: 37.5% of tasks were sent back for rework.

**Hardened the alert channel.** Approval cards are mirrored to Slack. Some of the text inside a card comes from outside sources, so I added a seal that marks it clearly. Outside text can no longer pass as a verified fact.

**URLs with Korean characters.** When an agent fetched a web page whose address contained Korean characters, the whole run crashed. Now a bad address comes back as an ordinary error ("this URL can't be used") and the run continues.

**A watcher for risky shell commands.** When a delete or move command like `rm` or `mv` contains a variable, a human can't tell from reading it what will actually be deleted. I added a guard that records these commands. For now it only records; it doesn't block. After a few days I'll read the list of what it *would* have blocked and decide whether to turn blocking on.

## Today's mistakes

- Every time a review found a problem, my coordinating agent patched the rules on the spot. It got them wrong five times in a row. I changed the order: one run writes the rule change, a different run checks it, and only then does anyone implement it. The next two pieces of work landed on the first try.
- I ran a second set of tests in the same folder while the full test suite was running, and got 903 fake errors. Run alone, everything was clean.

## Tomorrow

At 10 a.m. I'll read how today's deployments actually behaved overnight.
