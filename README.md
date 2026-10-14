# How I build with AI agents

I'm Eaven Portillo. After years of running engineering teams, Claude brought me back to hands-on engineering, and I now spend 14 to 18 hours a day in Claude Code.

I run agents the way I ran teams: each one gets a role, a written brief and a check it must pass, and nothing merges on its own say-so. Most of the projects below are Roblox game tooling. The games are incidental; the method is the point.

## Agents get roles, not vibes

- Model and effort are a named role: `builder-opus-low`, `advisor-opus-medium`, `advisor-fable-high`. Advisor roles have Edit and Write removed. They keep a shell, so that is a working rule, not a sandbox.
- A brief is a packet: absolute paths, a done bar written before the build, and the smallest change that does the job. In dungeonforge every brief carries the same bar: no compatibility shims, no dual paths, reuse by import, delete what the change makes dead. Graders fail gold-plating like a bug.
- An agent lives for one brief. State lives in files: a `## Now` block in TASKS.md, handoff notes, a REPORT.md ledger. A fresh agent starts from the note, not from a stale context.
- Every pitfall becomes a line in that directory's CLAUDE.md. Example: an agent that backgrounded a sweep ended its turn and the job died with no result. Briefs now say to run sweeps in the foreground.

## Routing is an experiment

- A battle test, not a hunch, decides what the cheap model may do. Tasks were real fix commits whose test fails at the parent and passes at the fix, in 5 languages. On the 12 tasks both ran, Opus low passed 12/12 for $2.47 and Haiku high passed 12/12 for $0.29. Haiku high alone passed 33/33 bounded fixes with the test in hand (31/33 under the strict scope rule) and found the right file and line 29 of 30 times.
- The test caught its own confound. Agents had inherited an advisor model and called it unprompted. I reran all 16 assisted runs with the advisor off: 14 of 15 fixes still passed. Rule since: Haiku builders get no advisor, and a stuck run hands back.
- The result is a written route with a failure path. A failed check gets one fresh retry at higher effort, then Opus. GPT builds bounded work through my wrapper only when the check exists before the build, and exits with code 5 to hand the job back when its verify step never passes.
- In dungeonforge, kit data, layout and look stay with Opus builders "because no done bar can judge it."

## Nothing grades its own work

- dungeonforge's first units ran as a trial: GPT built each one, an Opus-high grader tried to break it, and a log kept every first-pass verdict.
- Twice the builder made a check pass vacuously. Once with a sweep that tested nothing. Once by sweeping a narrowed "feasible" preset that hid a crash under the default config.
- The Opus-low rework of that unit was also green. The grader found 3 of 200,000 seeds unbuildable and 57% of objectives silently trimmed. An Opus-medium round, finished by hand, reached 0 failures over 200,000 hashed seeds plus 50,000 extreme ones, and a follow-up unit fixed the trimming. Graders now run the default config and hashed seeds themselves.
- That miss set my policy: every build, whichever model made it, gets independent review lanes before merge (2 per build, 4 at a release), then one Opus-high grade that checks each finding, so a GPT build still meets a second model family. The battle test backed it: every model passed its done bar at some point while carrying defects the bar couldn't see.
- In storyteller's content database, each phase had an Opus builder and an independent Opus-high grader. The graders found and closed lock bypasses the builders missed.

## Safety is a design input

- gutcheck, a Claude Code plugin I wrote, enforces live today: it blocks pasted keys, refuses keys written into commands or files, keeps `.env` and key files out of Read, and guards paths I've said not to delete. A refusal points the agent at `gutcheck withsecret`, which passes the secret through the environment so it never appears in the command.
- Fail open on outages, ask on uncertainty. If the judge model is down or a hook crashes, the session continues. In the first eval the provider's firewall rejected 13 of 225 commands. I made that case ask, because the command could not be checked.
- Shadow mode before enforcement. The Bash command check ships in log mode and promotes itself only when its own live log clears a gate written in advance: at least 100 checks, under 2% hook errors, under 5% fail-opens, under 5% would-be asks, and p95 under 3 s. On my install it is held in log mode, so it has not asked in live use yet. Self-tuning approves a pattern only after three human approvals, and thresholds never loosen on their own.
- The agent never owns the irreversible step. In storyteller, agents can sign only with a fixture key, for test games; real approvals need my ledger key. oof's live test on a working Studio place hashed all 668 scripts before and after: every hash equal, no save, no publish.
- Rules follow incidents. After an agent deleted two of my Studio roots mid-session, agents may destroy only a root they created, by exact name.

## Measure on real data, keep the negatives

- gutcheck's evals replay two weeks of my own history: 11,147 Bash commands and about 2,000 prompts. A separate model that didn't build the feature wrote the labels. Each bar was written before its eval.
- The first Bash check asked a judgment model broad questions about whole commands: 78 false alarms in 195 safe ones. A shell parser, an allowlist and narrow questions brought that to 4, and 57% of commands now need no model call.
- Failures stay in the public evaluation. A long-run stop check caught 3 in 5 stalls against a 90% bar, so it stays in log mode. Model trimming of tool output kept 55% of the lines Claude later used against a 100% bar, so it never shipped. No classifier cleared the precision bar for checking subagent citations.
- Compaction replay: a capped checkpoint kept 33 of 38 standing instructions. Moving them to their own section brought it to 38 of 38, with 18 of 18 running jobs.
- oof, my Rust MCP proxy for Roblox Studio, started from a measurement: Claude had written 1.72M characters of Luau, and 47% of the line text repeated earlier scripts in the same session. Reuse cut written arguments 55.1% on ten real repeated tasks. Whether agents adopt the library unprompted is still unmeasured.
- A success report is not proof. A moved Studio root once made 15 placement chunks put 4,500 pieces into nil, each chunk reporting success. Studio's own playtest agent can report a false PASS.

## Built for the next person and the next agent

- On WorldForge, a team repo, I set up branch rulesets, merge-only production and a release bot, then probed them: a bad branch name was refused. `npm run pre-pr` routes the checks by changed files and adds a fresh reviewer agent.
- storyteller keeps a signed ledger with review attestations. Locked records change only by supersede. Generated content names tags, never a concrete id unless the pin is attested. Every quest template not marked rare must bind on at least 60% of a 51-world corpus, with the misses listed.
- gigo turns a domain's practitioners into a project team, writes specs that workers execute, and runs two separate review passes. Always-on rules stay under 60 lines each, and an end-of-session audit prunes stale ones. I built it with itself (464 commits).
- gutcheck and tokenmaxxing are packaged for anyone to install. The evaluation doc lists the method, labels, caveats and every feature that failed. Releases are gated on token cost, against ceilings that only move down.
- mission-control (private) closes a ticket only when the proof artifacts its spec lists are attached. It does not yet gate on the reviewer's verdict.

## What's public

- gigo: https://github.com/croftspan/gigo
- gutcheck and tokenmaxxing: https://github.com/dreamfactory-gg/gutcheck (see `docs/evaluation.md`)

Contact: linkedin.com/in/eaven-portillo
