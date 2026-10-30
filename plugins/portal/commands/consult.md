---
description: Ask one or more Companions for a perspective, critique, or answer.
argument-hint: "companions about my architecture plan / feature implementation / this idea I have ..."
---

The user invoked `/consult $ARGUMENTS`.

`$ARGUMENTS` is a free-form description of what user wants to consult Companions about. If it is empty, ask what the user wants to use Companions for.

1. Decide whether the description refers to something in the current session (a file, draft, decision, or discussion above) or a standalone problem. When it refers to the session, pull the relevant context — material, constraints, what was already tried — into the consultation prompt. When the user is following up on a Companion's earlier answer from this session, prefer `continue_from` with that job id over restating it.
2. Judge whether the ask suits a single Companion or several genuinely different perspectives.
3. If the right Companion, count, or mode is unclear, suggest the `discover` tool to have the system propose the best setup for this input.
4. Run the consultation as the `using-companions` skill describes — it is the single home for how the system works (receipts, tool declarations, errors, presentation).

On a needs-auth state, or an `authenticate` tool on this server, do not call it: use the `/setup` walkthrough.
