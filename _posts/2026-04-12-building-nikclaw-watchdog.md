---
layout: post
category: AI Systems
---

## Building niklaw-watchdog: what I learned trying to inspect AI agents in production

I started building niklaw-watchdog thinking I needed an observability dashboard for AI agents.

What I learned is that observability is only part of the problem.

The real problem is investigation.

Most dashboards are built to answer questions like:

• is the system up?
• how many tokens did it use?
• how fast was it?
• did anything error?

Those are useful questions.

But when an agent does something weird in production, they’re usually not the questions you actually care about.

What you want to know is:
what happened, in what order, and inside what context?

With agent systems, that gets messy quickly.

A single output might sit inside a chain like:

cron → flow → task → session → subagent → turn → tool → message

If you flatten all of that into a timeline, it may look clean, but it becomes harder to understand.

That pushed me toward a different design choice in niklaw-watchdog:
the investigate view should be a real tree, not just a feed of events.

Because when you’re debugging agents, hierarchy matters.

You need to know:

- what spawned what
- which session a tool call belonged to
- whether something came from the main agent or a subagent
- how one action led to the next

Another thing this project clarified for me is that “turns” are useful for reading behavior, but they’re not always the right primitive for modeling it.

They’re good for interfaces.
They’re not always good enough for explaining system behavior.

The underlying structure is usually more about tasks, sessions, tools, messages, and parent-child relationships than about a neat conversational transcript.

So the main takeaway for me has been pretty simple:

if you’re building tooling for agents, don’t just optimize for monitoring.
Optimize for reconstructing what happened.

That’s the standard I’m aiming for with niklaw-watchdog.