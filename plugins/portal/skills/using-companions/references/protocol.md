# Consultation protocol

Only `consult` and `discover` bill; only `get_answer` returns run content. Every other tool feeds an existing job, stops one (`cancel_job`), or reads the catalogue. `consult` only submits: it bills the run and returns a receipt with a `job_id`, never the answer.

## Sending a history

`consult` takes either `prompt` or `messages`, never both. `messages` is a chat history of `system`, `user` and `assistant` turns, each with text, whose last turn is the user's; it is sent to the API as given. Use `messages` when the user is continuing an exchange whose turns you already hold; use `prompt` for a single question.

## Continuing a conversation

After an `answer` or `parallel_with_main` run is collected, Portal keeps the thread locally. To continue it, call `consult` with `continue_from: <job_id>` and `prompt` as the next turn — not `messages`. It runs as `answer` with the same main; `model`, sampling and crumbs may be overridden; a different workspace, main or mode is refused. Against a Portal whose `consult` schema has no `continue_from`, start a fresh consult and carry the context in the prompt.

Treat the response status as the next action:

- `complete`: read the typed `content`, translate it for the user, and attribute it.
- `pending` or `running`: still working, not lost. Keep the `job_id` and call `get_answer`. It waits up to `timeout_seconds` (default 40, maximum 40); call it again on a useful cadence, not in a tight loop. Never start another consultation to collect the same work — that can create another billed run.
- `requires_action`: execute every pending client-tool call, then pass exactly one result per `tool_call_id` to `submit_tool_outputs`. It returns a receipt; collect the resumed run with `get_answer`.
- `needs_reply`: answer the requested clarification with `submit_reply`, then collect with `get_answer`.
- `ambiguous`: show the candidates and retry with the intended stable ID after the user or context resolves it.
- `decision`: fix the reported input problem before making a new call; there may be no job to poll.
- `failed` or `error`: explain the useful error message. Do not pretend a Companion answered.
- `cancelled`: the run was stopped and is over; nothing more to collect. Tell the user; do not start a new consultation to replace it unless they ask. A later `get_answer` for it returns the same result while Portal keeps it: in memory, for up to one hour after it was first returned, and only for the 256 most recent results. After that, or after a Portal restart, it answers `portal_lost_ownership` — nothing more is owed.
- `cancel_requested`: the stop was accepted but not yet confirmed. Submit no more tool outputs or replies for this job; collect it with `get_answer`.
- `cancel_not_applied`: with `outcome: already_complete`, the run finished first — collect the answer with `get_answer`; with `outcome: retry_required`, it changed state mid-request — call `cancel_job` again.

A 422 rejection lists what the API currently accepts — relay it and adjust rather than pre-judging what is enabled. `list_params` shows the currently available modes, models, settings, and limits.

If credit is insufficient, tell the user before attempting another consultation.

## Cancelling a job

When the user asks to stop a consultation, or it is no longer wanted, call `cancel_job` with its `job_id`. It needs no roster. A cancel stops the run at the node it arrives on and never charges for more; what was already done stays charged. Anything charged but not delivered is refunded, and a `cancelled` result carries the net `cost` and `delivered_nodes` when present. After asking to cancel, if `get_answer` returns `requires_action` or `needs_reply` for that job, call `cancel_job` again instead of executing the calls or replying — this overrides the usual handling of those states. A running `discover` is not stopped; it finishes on its own. If your tool list has no `cancel_job`, the server predates it: tell the user the job cannot be stopped from here, and never start a new consultation to replace or cancel it.
