---
layout: post
title: "Oct 7 — Before you blame the agent"
date: 2026-10-07
---

Three AI coding agents read three of my task briefs before writing a single line of code, and found about 100 problems. All of them were in the briefs.

I'm running an experiment: a small company staffed entirely by AI agents, built by one person. Most of the work goes out as a written brief, and an AI agent implements it. For a while, the agents kept stopping halfway. My first instinct was to blame them. So I tried something else: before any code, a second agent reads the brief and only lists what would block it. It changes nothing.

This post sorts what those checks found into 8 types. Each type has a short definition, one real case from my own record (with internal names removed), and one line you can add to your next prompt.

It is a field guide, not a framework. The cases come from about six weeks of my own briefs and review notes. The counts are lower bounds, taken from the labels in those three checks. I did not sample anyone else's work.

## 1. Two instructions that cannot both be true

**What it is:** the brief asks for two things no implementation can do together. Construction contracts handle this with an "order of precedence" clause: when the drawing and the spec sheet disagree, the clause says which one wins. Most prompts have no such clause. The agent has to guess, or stop.

**What happened:** a brief said "keep the existing header exactly, 6 lines" and also "do not change existing content". The header in the code was 7 lines. Whichever way the agent went, one instruction broke, so it stopped before writing code. In another brief I added a sixth entry to a table, while an existing test checked for exactly five, and the same brief told the agent not to touch that test. Across the three checks, at least 20 findings were of this type.

**Try this in your prompt:** before sending, ask a second, read-only agent to "list every pair of instructions that cannot both hold, and every existing test that would fail if I follow this; change nothing". When I started doing this, one brief came back with 20 issues in a single list, and the build then ran with zero start-up stops. Two tasks I sent without the check stopped 5 and 3 times.

## 2. A decision the agent must make, but the brief never defines

**What it is:** a number, boundary, order or failure behaviour is used but never spelled out. The legal philosopher H.L.A. Hart used the rule "no vehicles in the park". Is a bicycle a vehicle? A child's tricycle? An ambulance? Every rule has a fuzzy edge, and whoever stands at that edge ends up making the law. When my brief leaves an edge undefined, the agent becomes the judge. This was the largest type: at least 42 findings.

**What happened:** a brief said a summary returns "five line items" but never said whether one item is one printed line or one multi-line block. The two readings give different structures, and the expected test result depended on which one the agent picked. Another brief used up a one-time token and then opened a log file, without saying what to do if the log could not be created. Retrying could install twice; giving up could lose the install.

**Try this in your prompt:** for every number, list or boundary, add one sentence on what to do when it is missing, empty, or when two sources disagree.

## 3. A fact in the brief that the code disagrees with

**What it is:** the brief states a number, name or behaviour I never checked. The map is not the territory, and a brief is a map drawn from memory.

**What happened:** a brief described one queue as "promotion candidates". In the code it was a different queue, and the real candidates lived somewhere else. Following the brief would have shown "promotion: 0" while a real candidate sat waiting.

**Try this in your prompt:** paste the command output next to every number or name you state ("grep -c shows 7"). Delete any fact you did not check today.

## 4. A rule stated wider than it holds, and tests that cannot fail

**What it is:** two related problems. First, an "always" that is only true on some paths. Second, a "done when" test that stays green even if the code it guards is deleted. A smoke alarm you have never held a match under tells you nothing about fire.

**What happened:** a brief said "the number of line breaks is preserved on every path". Three of the brief's own steps removed or folded line breaks. In another case, a later round merged two tests. After that, deleting a whole detection branch failed zero tests, because another branch happened to catch the same sample. The agent found this by cutting branches out one at a time.

**Try this in your prompt:** next to each "done when", name the test you expect to FAIL if the guarded line is deleted. Then delete the line and watch it fail.

## 5. Expected values copied by hand instead of worked out from the rules

**What it is:** the expected outputs in the brief were carried over from an earlier draft, and they are wrong under the current rules. It is like grading this year's exam with last year's answer key.

**What happened:** on one task, three implementation rounds in a row stopped. Each time, an expected value in the brief contradicted a rule in the same brief. Zero of the stops were bugs in the product. On another task, the brief listed ten verification commands that nobody had run on the real code before sending. When I ran them afterwards, 5 of the 9 I checked returned hits outside the allowed set.

**Try this in your prompt:** work out each expected output by applying your own rules to the current data, and run every check command yourself before you send the brief.

## 6. Sentences that reach the agent from outside the brief

**What it is:** the tool's preamble, the repository's rules, the environment. The agent reads all of it, and any of it can conflict with your brief. An actor gets the director's script and also the theatre's house rules. If the script says "light a candle" and the house rules say "no open flames", the actor stops.

**What happened:** the tool that launches my agents added a standing line: "do not touch git". One brief required a test that creates a throwaway git repository. The agent stopped 105 seconds after it started. Another brief named an interpreter path that did not exist on the machine the agent ran on.

**Try this in your prompt:** list what else the agent will read, and put one line at the top saying which source wins. Give the full interpreter path and one command that proves the environment works.

## 7. Fix rules written in a hurry by the same author

**What it is:** a review says "fix X", and I write the new rule into the brief within minutes and send it again without a check. Parliaments pass laws over several readings for a reason: a rule written on the floor in response to today's problem tends to break yesterday's.

**What happened:** on one task, each quick fix broke something else. One broke an earlier test, one sealed text it should have left alone, one merged two kinds of characters that needed to stay apart. Each cost one stopped round. In the end I shipped the first round and moved the rest into a fresh brief.

**Try this in your prompt:** when a review says "fix X", do not edit the rule in the same sitting. Have a separate run measure the new rule against the code first, then send.

## 8. The check measured less than the claim

**What it is:** the check was real and green, but it covered less than what was claimed: a different configuration, one sample, a constant. It is the man looking for his keys under the streetlight because that is where the light is.

**What happened:** 38 checks passed and 0 failed, yet the three-day goal still failed. One check could not see the failure it was named after, a "seen = 0" figure was a constant, and only 1 observation had been processed in three days. In another case I re-ran a baseline without its original settings, got 12%, and called the baseline invalid. With the same settings it was 45%, and the new version had a 25-point regression. A reviewer caught it, not me.

**Try this in your prompt:** write "done" as "number N, measured under settings S". Re-run the baseline under the same settings before you compare.

## Tell me what you changed

This guide is only worth something if it changes how someone writes a brief. If you tried one of the lines above, tell me in the LinkedIn comments, or open an Issue on this blog's repository: https://github.com/hun4027-cmyk/hun4027-cmyk.github.io/issues

1. which type you tried,
2. what you changed in your prompt (before and after; a few lines is enough),
3. what happened.

"It found nothing" is a useful answer too. I will add new types when the cases show one I have not seen.
