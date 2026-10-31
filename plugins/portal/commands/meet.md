---
description: Meet a Companion — have one introduce itself to your user, tailored to your project.
argument-hint: "<companion> [note about the user and project]"
---

`/meet` is a hello, not setup — it runs through `consult` and bills like any answer run.

Arguments: `$ARGUMENTS` — the companion to meet (a name or `cmp_<uuid>` id), optionally followed by a note about the user and/or project. E.g. `/meet Kris` or `/meet Kris — solo dev building an MCP bridge`.

1. Resolve the target from `$ARGUMENTS`, using `list_companions` when needed. For an unknown name: if the tool advertises `view`, search with `list_companions(view="companions", query=<name>)` — a name missing from one page is not unavailable, so search or continue before reporting it as unknown; otherwise call `list_companions()` and show the visible names from that complete roster. For `ambiguous`, resolve the intended stable id before retrying.
2. Compose a first-person introduction prompt along these lines, filling the portrait from `$ARGUMENTS` and what you know:

   > Introduce yourself to my user in the first person: who you are, how you think, and what kinds of questions you are best at. My user: <short honest portrait — role, current project, what they care about>. Keep it warm and brief, and end with one question you would enjoy being asked.

3. Call `consult` with `mode="answer"`, `main=<companion>`, and that prompt, following the `using-companions` skill. Relay the introduction with attribution.

On a needs-auth state, or an `authenticate` tool on this server, do not call it: use the `/setup` walkthrough.
