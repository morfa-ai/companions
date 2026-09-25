# Consultation protocol

Only `consult` and `discover` bill; only `get_answer` returns run content. Every other tool feeds an existing job, stops one (`cancel_job`), or reads the catalogue. `consult` only submits: it bills the run and returns a receipt with a `job_id`, never the answer.

## Directory

Without `view`, `list_companions` returns the complete companions list plus the visible teams with `member_count`; open one with `view="members"` and its `team_id`. The `kind` filter counts only matching members and drops teams with none; `visibility` filters by the team's own visibility. The `team_id` must be a `team_<uuid>` id; copy a row's `id` from `view="teams"`.

## Sending a history

`consult` takes either `prompt` or `messages`, never both. `messages` is a chat history of `system`, `user` and `assistant` turns, each with text, whose last turn is the user's; it is sent to the API as given. Use `messages` when the user is continuing an exchange whose turns you already hold; use `prompt` for a single question.

Treat the response status as the next action:

- `complete`: read the typed `content`, translate it for the user, and attribute it. If it reports `partial: true`, see Partial answers below.
- `pending` or `running`: still working, not lost. Keep the `job_id` and call `get_answer`. It waits up to `timeout_seconds` (default 45); call it again on a useful cadence, not in a tight loop. Never start another consultation to collect the same work — that can create another billed run.
- `requires_action`: execute every pending client-tool call, then pass exactly one result per `tool_call_id` to `submit_tool_outputs`. It returns a receipt; collect the resumed run with `get_answer`. A piecemeal `submit_tool_outputs` acknowledgement reports `requires_action` with `accepted` and `missing` call IDs and no `pending_tool_calls`; an actionable `requires_action` response carries `pending_tool_calls` to execute.
- `needs_reply`: answer the requested clarification with `submit_reply`, then collect with `get_answer`.
- `ambiguous`: show the candidates and retry with the intended stable ID after the user or context resolves it.
- `decision`: fix the reported input problem before making a new call; there may be no job to poll.
- `failed` or `error`: explain the useful error message. Do not pretend a Companion answered.
- `cancelled`: the run was stopped and is over; nothing more to collect. Tell the user; do not start a new consultation to replace it unless they ask.
- `cancel_requested`: the stop was accepted but not yet confirmed. Submit no more tool outputs or replies for this job; collect it with `get_answer`.
- `cancel_not_applied`: with `outcome: already_complete`, the run finished first — collect the answer with `get_answer`; with `outcome: retry_required`, it changed state mid-request — call `cancel_job` again.

Remote error results use `{status: "error", http_status, body}`, where `body` is the API's error response as sent. Read the error type and message inside it: `body.details` carries `error_type` and `message`, and a framework error carries `body.detail` instead (a message string or a list of validation errors).

A 422 rejection lists what the API currently accepts — relay it and adjust rather than pre-judging what is enabled. `list_params` shows the currently available modes, models, settings, and limits. `list_params` reports `pause_cycle_cap`, `max_tool_output_bytes`, `max_tool_outputs_total_bytes` and `max_tool_iters`; read their current values from `list_params`.

`local_tools` is a Portal-only `consult` field; the remote MCP does not accept it.

If credit is insufficient, tell the user before attempting another consultation.

## Partial answers

A partial answer keeps status `complete` and reports `partial: true`. `failed_nodes` holds `{node, stage_id, error_type}` objects. `skipped_nodes` reports gate skips whether or not the answer is partial. Older results omit fields they do not have; do not invent a reason. `balance_after` is the run's own ledger snapshot at settlement, not live credit; `check_balance` is the live read.
## Cancelling a job

When the user asks to stop a consultation, or it is no longer wanted, call `cancel_job` with its `job_id`. It needs no roster. A cancel stops the run at the node it arrives on and never charges for more; what was already done stays charged. Anything charged but not delivered is refunded, and a `cancelled` result carries the net `cost` and `delivered_nodes` when present. After asking to cancel, if `get_answer` returns `requires_action` or `needs_reply` for that job, call `cancel_job` again instead of executing the calls or replying — this overrides the usual handling of those states. A running `discover` is not stopped; it finishes on its own. If your tool list has no `cancel_job`, the server predates it: tell the user the job cannot be stopped from here, and never start a new consultation to replace or cancel it.
