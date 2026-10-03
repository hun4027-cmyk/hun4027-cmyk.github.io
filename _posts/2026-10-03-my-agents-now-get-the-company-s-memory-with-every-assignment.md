---
layout: post
title: "Oct 3 — My agents now get the company's memory with every assignment"
date: 2026-10-03
---

I'm running an experiment: a small company staffed entirely by AI agents, built by one person. The goal is an operating system that keeps working while I sleep. This stage is about building the system and getting good at it, not revenue.

Today's thread was one question: **does the company's memory actually reach the agents doing the work, and can I measure it?**

## What shipped

**Every department assignment now carries the company's memory.** When I hand a task to a department agent, a small hook attaches the most relevant notes and past documents from a local index on my Mac, plus that department's own recent history from the server. When the agent finishes, the hook reads its final report, counts which of the attached notes it actually cited, and stores one line it writes for next time. The first live run: five notes attached, one cited, the agent's line saved. Nothing about the assignment itself is changed, and if the hook is slow or fails, the task goes out anyway.

**The hard part was the second half.** Most of my agents run in the background, so their reports don't come back where the first hook was looking. The fix reads the report from the documented events instead. Several events can arrive in any order, even at the same time, so a first draft had a dozen ways to record a run twice or never. I replaced the case-by-case fixes with one lock and a small table of states, so every order of events ends in exactly one record.

**A dashboard light that can turn green honestly.** The company's status line has three lights: server, departments, local memory. The local light has always said "unmeasured". Today the server learned to compute two of its numbers from fixed formulas: how often departments cite the memory they're given, and whether three past questions still pull up the right documents. I wrote the pass bar (at least 5 assignments, at least 50% cited) *before* looking at the data, and loosening it later needs my sign-off. The three test questions only point at documents written before the incidents they're about, so the check can't be green by hindsight. Right now the citation rate is 54%. The light stays "unmeasured" until the Mac-side runner for the second number is live.

**Faster memory search, and a mystery.** I replaced a slow pure-Python search with a vectorized one. In a direct test the search step itself went from about 650 ms to under 5 ms. On my real prompts the whole lookup still takes about 700 ms, and I don't know why. Instead of raising the time limit until it "passes", I'm adding per-stage timing so the next readout shows where the time goes.

## How the work was checked

Each piece went the same way: one AI writes the spec, a model from a **different vendor** tries to break the spec before any code exists, an AI implements it, a reviewer from the other vendor tries to break the code, and every test has to prove it can fail: I delete the guard it protects and the test must turn red.

The cross-vendor check earned its keep today. On the same spec, my usual reviewer raised 2 blocking issues; the other vendor's model raised 17, mostly about events arriving in an unexpected order. All were fixed before a line of code was written, and the final review found nothing blocking.

## Today's mistakes

- A shell variable holding a list of files was passed as one argument instead of several. Once I committed an empty patch; once I sent a review package without its test files and lost one of my two review rounds. I now build file lists inside a script and check that the output isn't empty.
- An agent reported that it had added two test rows to a spec. It hadn't — its editing script failed silently. I only caught it because I check claims against the file now, not against the report.
- Twice the implementer stopped because it ran the whole test suite on the wrong folder. The spec never said which folder. Now it does.
- My question to myself today was "is this a real fix, or am I just making the test pass?" Once, a review suggested resetting state inside a test to make a timing check pass. I fixed the product's assumption instead, and added a test that fails if that assumption comes back.

## Next

Ship the Mac-side runner, then start a 7-day window where all the lights are measured every night. After that, a plain verdict: does this memory actually help the agents do better work, yes or no.
