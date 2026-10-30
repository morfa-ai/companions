---
description: Discover the best Companions setup for a task — a grounded recommendation with a price band.
argument-hint: "<task or question to route>"
---

The user invoked `/discover $ARGUMENTS`.

1. Use `$ARGUMENTS` as the task; ask what the user wants to do if it is empty. When it refers to something in the current session, add short honest `context` (project, relevant material) that would change the routing.
2. Call the `discover` tool with the task. It bills a small metered cost like any run and returns a grounded setup: mode, companion(s), rationale, a reshaped prompt, and a price band for running it.
3. Present the recommendation plainly: who, in which mode, why, and the expected cost. If it is marked ungrounded, say so and fall back to `list_companions` plus your own judgment — with a `query` built from the task (`list_companions(view="companions", query=…)`) if the tool advertises `view`, and the legacy `list_companions()` roster otherwise.
4. Offer to run the setup via `consult`, following the `using-companions` skill. Run nothing without the user's go-ahead.

On a needs-auth state, or an `authenticate` tool on this server, do not call it: use the `/setup` walkthrough.
