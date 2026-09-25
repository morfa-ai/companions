# Consultation protocol

Only `consult` and `discover` bill; only `get_answer` returns run content. Every other tool feeds an existing job or reads the catalogue. `consult` only submits: it bills the run and returns a receipt with a `job_id`, never the answer.

## Sending a history

`consult` takes either `prompt` or `messages`, never both. `messages` is a chat history of `system`, `user` and `assistant` turns, each with text, whose last turn is the user's; it is sent to the API as given. Use `messages` when the user is continuing an exchange whose turns you already hold; use `prompt` for a single question.

Treat the response status as the next action:

- `complete`: read the typed `content`, translate it for the user, and attribute it.
- `pending` or `running`: still working, not lost. Keep the `job_id` and call `get_answer`. It waits up to `timeout_seconds` (default 45); call it again on a useful cadence, not in a tight loop. Never start another consultation to collect the same work — that can create another billed run.
- `requires_action`: execute every pending client-tool call, then pass exactly one result per `tool_call_id` to `submit_tool_outputs`. It returns a receipt; collect the resumed run with `get_answer`.
- `needs_reply`: answer the requested clarification with `submit_reply`, then collect with `get_answer`.
- `ambiguous`: show the candidates and retry with the intended stable ID after the user or context resolves it.
- `decision`: fix the reported input problem before making a new call; there may be no job to poll.
- `failed` or `error`: explain the useful error message. Do not pretend a Companion answered.

A 422 rejection lists what the API currently accepts — relay it and adjust rather than pre-judging what is enabled. `list_params` shows the currently available modes, models, settings, and limits. `list_params` reports `pause_cycle_cap`, `max_tool_output_bytes`, `max_tool_outputs_total_bytes` and `max_tool_iters`; read their current values from `list_params`.

`local_tools` is a Portal-only `consult` field; the remote MCP does not accept it.

If credit is insufficient, tell the user before attempting another consultation.
