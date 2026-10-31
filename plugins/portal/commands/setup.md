---
description: Check Companions connectivity, show the roster, and suggest grounded starting points — free, spends no credit.
---

# Companions setup

Setup is a free orientation: it reads the balance and the catalogue and never spends consultation credit.

1. **Check the connection.** Call `check_balance`.
   - On success, report the balance briefly and continue.
   - This plugin has no login step in the harness. A needs-auth MCP state, or an `authenticate` tool appearing on this server, means the local Portal daemon refused the connection: do not call that tool. Run `portal status`, then reconnect in `/mcp`.
   - If a tool answers with Portal's own error, relay that message verbatim.
   - A 503 from this server, such as "Failed to connect — HTTP 503", means the daemon could not pin the connection to one process and will accept a retry; it is not an outage. Reconnect in `/mcp`, or retry the call once.
   - On another network or 5xx error, report the endpoint the error names and ask the user to confirm connectivity.
2. **Show the roster.** A free catalogue read either way.
   - If the tool advertises `view` in its schema: call `list_companions(view="companions")` and present that **one page** — each Companion's name with its one-sentence `hint`. When `has_more` is true, say that more exist and offer either to narrow (`query`, `kind`) or to continue with `next_cursor` copied unchanged; never describe a page as the whole roster, and never walk pages on your own here. Then call `list_companions(view="teams")` for the team names with their `member_count`, and `list_companions(view="members", team_id=…)` to open a team the user asks about.
   - Otherwise: call `list_companions()`. Present each visible Companion with one sentence (its `hint`), then the teams briefly — that roster is complete.
3. **Explain how to use the system.**
   - When routing is unclear, say *"discover the best companions setup for …"* — the `discover` ask flow returns a grounded setup (companions, mode, reshaped prompt) with a price band.
   - The word *"companions"* in a request activates the Companion skills.
   - Consultations and `discover` bill; directory reads (`list_companions`, `list_params`) are free.
4. **Offer grounded starting points.** Review the user's actual recent work — recent conversations, repository activity, the current project — and propose a few concrete problems or questions worth bringing to Companions. Map each to a fitting Companion from the roster or to a `discover` ask, and say in one sentence why it would help. Run nothing until the user picks one.
