# Consultation protocol

Only `consult` and `discover` bill; only `get_answer` returns run content. Every other tool feeds an existing job or reads the catalogue. `consult` only submits: it bills the run and returns a receipt with a `job_id`, never the answer.

## Sending a history

`consult` takes either `prompt` or `messages`, never both. `messages` is a chat history of `system`, `user` and `assistant` turns, each with text, whose last turn is the user's; it is sent to the API as given. Use `messages` when the user is continuing an exchange whose turns you already hold; use `prompt` for a single question.

## Local tools

`tools: []` only means you declare no client tools. Portal may still add its configured local tools when the workspace allows it: `list_files`, `read_file`, `grep`, `git`, `write_file`, `edit_file` and `run_command`, as enabled in Portal's configuration. Send `local_tools: false` to run that call without Portal's local tools; for a run with neither Portal's tools nor yours, send `local_tools: false` and `tools: []`. Server-side web search is a separate setting (`web_search: false`). `local_tools` needs Portal 0.9.0: check that the `consult` schema has `local_tools` before sending it. The consult receipt reports `portal.local_tools` (whether Portal attached its local tools to that call; the companion may still not run any) and `portal.client_tools` (the client tools Portal accepted from you, by the names you declared), separately from `portal.substituted` (the local tools Portal added).

## Continuing a conversation

After an `answer` or `parallel_with_main` run is collected, Portal keeps the thread locally. To continue it, call `consult` with `continue_from: <job_id>` and `prompt` as the next turn — not `messages`. It runs as `answer` with the same main; `model`, sampling and crumbs may be overridden; a different workspace, main or mode is refused. Against a Portal whose `consult` schema has no `continue_from`, start a fresh consult and carry the context in the prompt.

Treat the response status as the next action:

- `complete`: read the typed `content`, translate it for the user, and attribute it.
- `pending` or `running`: still working, not lost. Keep the `job_id` and call `get_answer`. It waits up to `timeout_seconds` (default 40); call it again on a useful cadence, not in a tight loop. Never start another consultation to collect the same work — that can create another billed run. Portal's `discover` accepts `timeout_seconds` from 0 to 40 (default 40): it waits up to 40 seconds; 0 returns the job handle at once for `get_answer`.
- `requires_action`: execute every pending client-tool call, then pass exactly one result per `tool_call_id` to `submit_tool_outputs`. It returns a receipt; collect the resumed run with `get_answer`.
- `needs_reply`: answer the requested clarification with `submit_reply`, then collect with `get_answer`.
- `ambiguous`: show the candidates and retry with the intended stable ID after the user or context resolves it.
- `decision`: fix the reported input problem before making a new call; there may be no job to poll.
- `failed` or `error`: explain the useful error message. Do not pretend a Companion answered.

A 422 rejection lists what the API currently accepts — relay it and adjust rather than pre-judging what is enabled. `list_params` shows the currently available modes, models, settings, and limits. `list_params` reports `pause_cycle_cap`, `max_tool_output_bytes`, `max_tool_outputs_total_bytes` and `max_tool_iters`; read their current values from `list_params`.

If credit is insufficient, tell the user before attempting another consultation.
