# Changelog

All notable changes to the Companions and Portal plugins are documented in this
file.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
From 0.7.0, the shared minor identifies the API, Portal, and plugin compatibility
line. Compatible releases may advance each component's patch independently.

## [Unreleased]

### Changed

- `companions`: the remote MCP connection and OAuth resource use `api.morfa.ai`; setup instructions use the same endpoint.
- `companions` and `portal`: publisher metadata uses `morfa.ai`, and Portal installation links use `get.morfa.ai`.

## [0.9.0] - 2026-09-29

### Added

- `companions` and `portal`: the skills document `cancel_job` — when to cancel, the `cancelled`/`cancel_requested`/`cancel_not_applied` statuses, re-cancelling on a pause, and what a cancel charges (work already done stays charged, charges for undelivered work are refunded, and nothing more is billed). It needs the API/Portal release that ships `cancel_job`.
- `companions` and `portal`: the protocol reference says that `list_params` reports `pause_cycle_cap`, `max_tool_output_bytes`, `max_tool_outputs_total_bytes` and `max_tool_iters`. The Portal plugin also documents `discover`'s `timeout_seconds` (0 to 40, default 40; 0 returns the job handle at once for `get_answer`).
- `portal`: documents the Portal 0.9.0 `local_tools` consult field (`local_tools: false` turns off Portal's local tools, and with `tools: []` gives a run with no tools). The remote `companions` protocol notes that `local_tools` is Portal-only.
- `portal`: the consult receipt reports `portal.local_tools` (whether Portal's local tools were offered for that call) and `portal.client_tools` (the client tools Portal accepted, by their declared names), separately from `portal.substituted`.
- `portal`: partial answers documented in full — `partial_reason`, `delivered_nodes`, the `insufficient_balance` and `pause_cycle_cap` hints, and a failed result's `cost`. The remote `companions` plugin documents only what its MCP returns (`partial`, `failed_nodes`, `skipped_nodes`). Both say `balance_after` is the run's own ledger snapshot at settlement, not live credit, and that `check_balance` is the live read.
- `portal`: documents `discover`'s optional `idempotency_key` and reusing it on retries.
- `companions` and `portal`: how to tell a partial `submit_tool_outputs` acknowledgement (`requires_action` with `accepted` / `missing` and no `pending_tool_calls`) apart from an actionable `requires_action` that carries pending calls.

### Changed

- **Breaking:** A `list_companions` call with no arguments now returns each team's `member_count` instead of its full roster. To see a team's members, call `list_companions` with `view="members"` and its `team_id`; `team_id` must be a `team_<uuid>` id. Needs API 0.9.0 / Portal 0.9.0.
- **Breaking** (`portal` only): Errors from the Companions API now come back as `{status: "error", reason, message, fix}`, with `fix` omitted when unavailable and `reason` always nonempty. `http_status` and `job_id` stay as metadata, and the raw `body` is no longer returned. Portal's own local refusals keep their typed status and always carry a `message`. The remote `companions` plugin documents its unchanged `{status: "error", http_status, body}` shape. Needs Portal 0.9.0.

### Fixed

- `portal`: the `get_answer` wait is documented as 40 seconds (default and maximum), and the protocol states the 1 h / 256-result retention bound for a repeat `get_answer` on a delivered job.

## [0.8.1] - 2026-09-22

`continue_from` needs Portal 0.8.1; `messages` on the remote MCP needs API 0.8.1. Against older servers the skills fall back to a fresh consult with the context in the prompt.

### Added

- `companions`: the consultation protocol explains sending a history — `consult` takes `prompt` or `messages` (system, user and assistant turns ending with the user's), never both — and when to prefer `messages` over restating an exchange.
- `portal`: the same `messages` guidance, plus continuing a conversation: after an `answer` or `parallel_with_main` run, `consult` with `continue_from: <job_id>` and the next turn as `prompt` picks the stored thread up; `/consult` prefers it when the user follows up on an earlier answer.

## [0.8.0] - 2026-09-21

Installing or updating these plugins through `portal update --plugins-only`
needs Portal 0.8.0. The skills themselves work with Portal 0.7.3 and with the
remote MCP.

### Changed

- `companions`: `/setup` now shows one page of the companion directory instead of implying a complete roster, says when more results exist, and explains how to browse teams and open a team's members. The skills filter by the task first, continue only when the shortlist is insufficient or the user asks for everything, and use `refs` for the shortlisted full profiles. The guidance follows the advertised tool schema, so it still works against an older `list_companions` that offers no `view` argument.
- `portal`: the same one-page `/setup`, team and member browsing, and filter/continue/`refs` guidance for the Portal plugin — unchanged behaviour against an older Portal daemon whose `list_companions` advertises no `view`.

### Fixed

- `portal`: `brainstorm` and `perspective` now include their client-tool reference inside each skill, allowing Hermes to install the complete skills without rejecting links outside the skill folder.
- `companions`: the same self-contained `brainstorm` and `perspective` bundles keep client-tool guidance available when the skills are installed individually.

## [0.7.0] - 2026-09-11

### Added

- New `portal` plugin: the same commands and skills, connected to the local Portal daemon instead of the remote MCP. Installed by `portal setup`.
- New `/discover` command: asks the service for a recommended Companions setup and price band for a task. Discovery is metered; running the recommended consultation is a separate action.

### Changed

- Commands reduced from 12 to 5: `/setup`, `/consult`, `/discover`, `/meet`, `/balance`. The skills are now the single home for how the system works.
- **Breaking:** Consultations no longer infer Companions from saved defaults. Update saved prompts and integrations to name `main` for `answer`, `participants` for `parallel`, and both for `parallel_with_main`. Read `list_params` for mode requirements.
- `/setup` checks the connection, shows the roster and suggests starting points without spending credit. It no longer depends on the harness.
- Plugin descriptions rewritten to say what Companions are.

### Removed

- **Breaking:** Removed `/handshake`, `/preferences`, `/set-preference`, `/clear-preference`, `/list-companions`, `/list-teams`, `/list-models`, and `/describe-companions`. Use `/setup` for orientation and the skills' catalogue tools for roster and parameter reads; name Companions explicitly when consulting.
- **Breaking:** Renamed `/check-balance` to `/balance`. Update saved prompts that invoke the old command.

## [0.6.3] - 2026-07-22

### Changed

- Strengthened client-tool guidance in `using-companions`, `brainstorm` and `perspective` so hosts attach the available tool declarations on every Companion consultation by default.
- Added an instruction to use `answer` mode when attaching client tools, and to consult several Companions in parallel by running concurrent `answer` consultations.

## [0.6.2] - 2026-07-21

### Changed

- Strengthened the result-presentation guidance in `using-companions`, `brainstorm` and `perspective` so responses are more adjusted to the user.

## [0.6.1] - 2026-07-21

### Added

- `using-companions`, `brainstorm`, and `perspective` ask clarifying questions to gather missing context before a consultation.

### Changed

- Consultation guidance uses `get_job` to collect results instead of relying on `consult`'s built-in polling, and no longer instructs hosts to supply idempotency keys.

## [0.6.0] - 2026-07-17

### Added

- New `perspective` skill for structured second opinions, critiques, independent Companion views, and host synthesis.
- New `brainstorm` skill for human-steered creative exploration with independent ideas, conceptual compression, and review checkpoints.
- New `/preferences`, `/set-preference`, and `/clear-preference` commands for managing the user's everyday Companion.
- Reusable client-tool declarations and consultation continuation guidance for `pending`, `requires_action`, and `needs_reply` states.

### Changed

- Reworked the `using-companions` skill into a compact activation and routing guide for everyday questions, challenges, decisions, drafts, and ideas—not only formal expert consultations.
- Redesigned `/setup` and `/handshake` around a guided first success: introduce the system, recommend one editable everyday Companion, and offer tailored warm-up questions without automatically spending consultation credit.
- Defaulted ordinary consultations to `answer`, while choosing multiple Companions explicitly for each problem.
- Companion responses are now translated and conceptually compressed for the user while preserving distinct voices; multi-Companion results add a separate host synthesis.
- Simplified command guidance, plugin descriptions, and README onboarding around plain language and intentional second opinions.

## [0.5.0] - 2026-07-07

### Added

- New `get_job` MCP tool — collect a run by its `job-<uuid>` handle: the reader for a `pending` consult (or a paused/timed-out resume). Read-only, never bills.
- `using-companions` skill: a "Collecting a `pending` run" section teaching the submit-then-collect flow.

### Changed

- `consult` now waits a bounded courtesy-window (default 60s) then returns `{status: "pending", job_id}` instead of blocking indefinitely; the former `{status: "timeout"}` envelope is renamed `pending`. Resume verbs (`submit_reply`, `submit_tool_outputs`) default `timeout_seconds` to 60 as well.
- Working agreement bumped to v4 for the wait-policy rule (collect a `pending` run with `get_job`, never re-`consult`).

## [0.4.0] - 2026-07-06

### Added

- New `/describe-companions` command — full profiles for a shortlist of companions by name or id.
- New `/list-teams` command — teams and their member rosters.

### Changed

- Slimmed `list_companions` to a cheap directory of names, ids, and one-line hints; full profiles now come from the new `describe_companions` tool and team rosters from `list_teams`.
- Updated the `using-companions` skill for the shortlist → `describe_companions` flow.
- Working agreement bumped to v3 for the new discovery surface.

### Removed

- Dropped the deprecated `discover` MCP tool alias.

## [0.3.1] - 2026-07-06

### Fixed

- Corrected the OAuth resource URL used during authentication.
- Documented that the MCP server must be reinstalled for the Codex CLI, and added the reinstall note to the README. (`5ca531b`)

## [0.3.0] - 2026-07-03

### Changed

- Renamed the `meet` MCP tool to `handshake`, added a dedicated `/meet` command, and introduced the `using-companions` skill to guide when and how to reach for companion experts. (`664a813`)
- Reformatted indentation across JSON files for consistency. (`0ecf343`)

## [0.2.0] - 2026-06-30

### Added

- New `/meet` command. (`091342f`)
- `meet` handshake flow and `list_companions` support synced into the plugin. (`ab39352`)

### Changed

- Dropped the `companions-` prefix from command names and aligned them with the underlying MCP tool names (`companions-balance` → `check-balance`, `companions-consult` → `consult`, `companions-discover` → `list-companions`, `companions-list-models` → `list-models`, `companions-setup` → `setup`). (`091342f`)
- Renamed the `discover` capability to `list_companions`. (`ab39352`)
- CI now derives the auto-bump level (major/minor/patch) from Conventional Commit messages. (`77cb411`)

## [0.1.9] - 2026-06-15

### Fixed

- Removed agent directives from the authentication flow. (`68e76c3`)

### Added

- Added a Codex CLI installation note. (`8db41ff`)

## [0.1.8] - 2026-06-10

### Changed

- Reorganized the repository into a `plugins/companions/` structure for easier plugin installation. (`b8edd0a`)

## [0.1.7] - 2026-06-10

### Fixed

- Added missing fields to the Codex plugin manifest and fixed the version bump script. (`0baf165`)

## [0.1.6] - 2026-06-10

### Added

- Codex plugin manifest (`.codex-plugin/plugin.json`) for Codex CLI support, plus a Hermes installation note. (`2a22aed`)
- Expanded the general project description in the README. (`a6dd4e8`)
- Documented integrations in the README and configured the CI workflow to skip on specific paths. (`4441899`)

## [0.1.5] - 2026-06-02

### Changed

- Switched to OAuth-based authentication; updated `/setup` and all commands to use the new flow. (`c87df10`)

## [0.1.4] - 2026-05-31

### Added

- Tool calling via MCP, allowing companions to invoke tools during a consultation. (`3fb28fe`)

## [0.1.3] - 2026-05-27

### Added

- New `list_models` MCP command. (`1ca3b8d`)

### Changed

- Updated response shapes to match the reworked API. (`1ca3b8d`)

## [0.1.2] - 2026-05-24

### Fixed

- Renamed `mcp.json` to `.mcp.json` so Claude Code auto-discovers the MCP server. (`8deaf0f`)

## [0.1.1] - 2026-05-24

### Fixed

- `/setup` now verifies authentication with a balance check. (`17aefc5`)

## [0.1.0] - 2026-05-24

### Added

- Initial release of the Companions plugin: marketplace manifests (Claude Code and agents), plugin manifest, MCP configuration, and the `consulting-experts` skill. (`b5de196`)
- Commands: `/companions-balance`, `/companions-consult`, `/companions-discover`, `/companions-setup`. (`b5de196`)
- CI workflow for automatic version bumping and a `bump-version.sh` script. (`b5de196`)

[Unreleased]: https://github.com/neurowelt/companions/compare/main...HEAD
[0.9.0]: https://github.com/neurowelt/companions/releases/tag/v0.9.0
[0.8.1]: https://github.com/neurowelt/companions/releases/tag/v0.8.1
[0.8.0]: https://github.com/neurowelt/companions/releases/tag/v0.8.0
[0.7.0]: https://github.com/neurowelt/companions/releases/tag/v0.7.0
[0.6.3]: https://github.com/neurowelt/companions/releases/tag/v0.6.3
[0.6.2]: https://github.com/neurowelt/companions/releases/tag/v0.6.2
[0.6.1]: https://github.com/neurowelt/companions/releases/tag/v0.6.1
[0.6.0]: https://github.com/neurowelt/companions/releases/tag/v0.6.0
[0.5.0]: https://github.com/neurowelt/companions/releases/tag/v0.5.0
[0.4.0]: https://github.com/neurowelt/companions/releases/tag/v0.4.0
[0.3.1]: https://github.com/neurowelt/companions/releases/tag/v0.3.1
[0.3.0]: https://github.com/neurowelt/companions/releases/tag/v0.3.0
[0.2.0]: https://github.com/neurowelt/companions/releases/tag/v0.2.0
[0.1.9]: https://github.com/neurowelt/companions/releases/tag/v0.1.9
[0.1.8]: https://github.com/neurowelt/companions/releases/tag/v0.1.8
[0.1.7]: https://github.com/neurowelt/companions/releases/tag/v0.1.7
[0.1.6]: https://github.com/neurowelt/companions/releases/tag/v0.1.6
[0.1.5]: https://github.com/neurowelt/companions/releases/tag/v0.1.5
[0.1.4]: https://github.com/neurowelt/companions/releases/tag/v0.1.4
[0.1.3]: https://github.com/neurowelt/companions/releases/tag/v0.1.3
[0.1.2]: https://github.com/neurowelt/companions/releases/tag/v0.1.2
[0.1.1]: https://github.com/neurowelt/companions/releases/tag/v0.1.1
[0.1.0]: https://github.com/neurowelt/companions/releases/tag/v0.1.0
