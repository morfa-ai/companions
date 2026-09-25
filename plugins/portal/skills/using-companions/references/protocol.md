# Consultation protocol

Only `consult` and `discover` bill; only `get_answer` returns run content. Every other tool feeds an existing job or reads the catalogue. `consult` only submits: it bills the run and returns a receipt with a `job_id`, never the answer.

## Directory

Without `view`, `list_companions` returns the complete companions list plus the visible teams with `member_count`; open one with `view="members"` and its `team_id`. The `kind` filter counts only matching members and drops teams with none; `visibility` filters by the team's own visibility. The `team_id` must be a `team_<uuid>` id; copy a row's `id` from `view="teams"`.

## Retrying discover

Send an `idempotency_key` with a `discover` request that may be retried, and reuse that key when retrying the same request. Portal forwards a nonempty key as the `Idempotency-Key` header; an omitted or empty key sends no header.

## Sending a history

`consult` takes either `prompt` or `messages`, never both. `messages` is a chat history of `system`, `user` and `assistant` turns, each with text, whose last turn is the user's; it is sent to the API as given. Use `messages` when the user is continuing an exchange whose turns you already hold; use `prompt` for a single question.

## Local tools

`tools: []` only means you declare no client tools. Portal may still add its configured local tools when the workspace allows it: `list_files`, `read_file`, `grep`, `git`, `write_file`, `edit_file` and `run_command`, as enabled in Portal's configuration. Send `local_tools: false` to run that call without Portal's local tools; for a run with neither Portal's tools nor yours, send `local_tools: false` and `tools: []`. Server-side web search is a separate setting (`web_search: false`). `local_tools` needs Portal 0.9.0: check that the `consult` schema has `local_tools` before sending it. The consult receipt reports `portal.local_tools` (whether Portal attached its local tools to that call; the companion may still not run any) and `portal.client_tools` (the client tools Portal accepted from you, by the names you declared), separately from `portal.substituted` (the local tools Portal added).

## Continuing a conversation

After an `answer` or `parallel_with_main` run is collected, Portal keeps the thread locally. To continue it, call `consult` with `continue_from: <job_id>` and `prompt` as the next turn — not `messages`. It runs as `answer` with the same main; `model`, sampling and crumbs may be overridden; a different workspace, main or mode is refused. Against a Portal whose `consult` schema has no `continue_from`, start a fresh consult and carry the context in the prompt.

Treat the response status as the next action:

- `complete`: read the typed `content`, translate it for the user, and attribute it. If it reports `partial: true`, see Partial answers below.
- `pending` or `running`: still working, not lost. Keep the `job_id` and call `get_answer`. It waits up to `timeout_seconds` (default 40); call it again on a useful cadence, not in a tight loop. Never start another consultation to collect the same work — that can create another billed run. Portal's `discover` accepts `timeout_seconds` from 0 to 40 (default 40): it waits up to 40 seconds; 0 returns the job handle at once for `get_answer`.
- `requires_action`: execute every pending client-tool call, then pass exactly one result per `tool_call_id` to `submit_tool_outputs`. It returns a receipt; collect the resumed run with `get_answer`. A piecemeal `submit_tool_outputs` acknowledgement reports `requires_action` with `accepted` and `missing` call IDs and no `pending_tool_calls`; an actionable `requires_action` response carries `pending_tool_calls` to execute.
- `needs_reply`: answer the requested clarification with `submit_reply`, then collect with `get_answer`.
- `ambiguous`: show the candidates and retry with the intended stable ID after the user or context resolves it.
- `decision`: fix the reported input problem before making a new call; there may be no job to poll.
- `failed` or `error`: explain the useful error message. Do not pretend a Companion answered.

Errors from the Companions API use `{status: "error", reason, message, fix}`, with `fix` omitted when unavailable and `reason` always nonempty. Portal's own local refusals keep their typed status (such as `invalid`, `decision` or `root_not_admissible`) and carry at least a `message`; `reason` and `fix` may be absent. `http_status` and `job_id` remain metadata. Read `message`, and `reason` when present; never parse `body`.

A 422 rejection lists what the API currently accepts — relay it and adjust rather than pre-judging what is enabled. `list_params` shows the currently available modes, models, settings, and limits. `list_params` reports `pause_cycle_cap`, `max_tool_output_bytes`, `max_tool_outputs_total_bytes` and `max_tool_iters`; read their current values from `list_params`.

If credit is insufficient, tell the user before attempting another consultation.

## Partial answers

A partial answer keeps status `complete` and reports `partial: true`. When present, `partial_reason` is `insufficient_balance`, `pause_cycle_cap` or `node_failure`, and `delivered_nodes` lists the delivered node names in graph order. `failed_nodes` holds `{node, stage_id, error_type}` objects. `skipped_nodes` reports gate skips whether or not the answer is partial. Older results omit fields they do not have; do not invent a reason or hint.

For `insufficient_balance` the hint reads:

> Partial answer: your credit ran out before every node could run. Only the delivered nodes are billed; `cost` is their total. Nodes that did not deliver add nothing to it: any charge they had was refunded. Top up and re-run for the full answer.

For `pause_cycle_cap` the hint reads:

> Partial answer: the run hit the tool-call pause limit before every node finished. Only the delivered nodes are billed; `cost` is their total. Nodes that did not deliver add nothing to it: any charge they had was refunded. Re-run with fewer tool rounds for the full answer.

`node_failure` adds no hint. A failed result keeps its reported numeric `cost`, including `0.0` after a whole-run refund. `balance_after` is the run's own ledger snapshot at settlement, not live credit; `check_balance` is the live read.
