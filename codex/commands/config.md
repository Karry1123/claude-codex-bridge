---
description: 可选：改默认模型/强度/权限（一般不用管，默认 gpt-6.1-sol medium 全权限）
argument-hint: [show | set model=.. effort=.. sandbox=.. | reset | models]
allowed-tools: Bash(${CLAUDE_PLUGIN_ROOT}/scripts/codex_bridge.py:*)
disable-model-invocation: true
---

Run `${CLAUDE_PLUGIN_ROOT}/scripts/codex_bridge.py` with the right subcommand for: $ARGUMENTS

- No args / "show" → `config show`
- "set k=v ..." → `config set k=v ...` (keys: model, effort[low|medium|high|xhigh|max|ultra], sandbox[read-only|workspace-write|danger-full-access], timeout, profile, codex_bin, extra_args, roles, model_reasoning, model_bulk, review_model, review_effort, team_policy, max_parallel, brief_chars, subagents[none|auto|active], max_subagents, codex_memory[scoped|off|on])
- "reset" → `config reset`
- "models" → `models` (lists models and the efforts each supports; check effort is supported by the chosen model)
- Natural language ("use terra with high effort") → translate to `config set`.

Print the resulting settings. Default sandbox is danger-full-access (full permission, no confirmations) by the user's choice; read-only roles (explorer/reviewer/architect) stay read-only on purpose.

Routing defaults: `model=gpt-6.1-sol`, `effort=medium`, `model_reasoning=gpt-6-astra`, `model_bulk=gpt-6-luna`, `review_model=gpt-6-astra`, `review_effort=high`, `team_policy=true`, `max_parallel=4`. `config show` prints all of them. Model/effort combinations are checked against the installed CLI catalog before tasks launch. `max_parallel` and timeout must be positive integers; `parallel --max` overrides concurrency for one call.

Structured values must be shell-quoted:
- `config set 'roles={"worker":{"preamble":"Keep edits scoped","effort":"high"}}'`: JSON object of role objects. Allowed fields: sandbox (allowed enum above), effort (allowed enum), model (string), preamble (string). Built-in read-only roles remain read-only unless a run explicitly passes `--sandbox`.
- `config set 'extra_args=["--add-dir","/path with spaces"]'`: JSON string array, or `config set 'extra_args=--add-dir "/path with spaces"'` for shell-style argument parsing.

Invalid structured values are rejected without changing the settings file. Corrupt/unreadable settings stop show/set and execution with the filename. Explicit `config reset` moves the bad file to `settings.json.corrupt-<timestamp>` before restoring defaults. `CODEX_BRIDGE_HOME` overrides the bridge settings/jobs/cache directory; it does not change Codex's own configuration.

`codex_memory`: `scoped` (default) binds Codex's global memory to the run's workspace so look-alike projects cannot bleed into each other; `off` disables memory read and write, including interactive `attach`; `on` is Codex's native behaviour. Scoped memory follows the current Git root and its main checkout for worktrees (inside or outside the main repo); nested independent repos never inherit outer memory. Outside Git, only exact workspace paths match. `usage` prints the newest valid Codex quota reading by event time, when one was returned; quota is otherwise unavailable.

Output defaults: `brief_chars=4000` (positive integer) caps the final message at a line/paragraph boundary (never mid-line) in run/resume/review/steer, parallel, result and wait reports; the full result stays on disk and is available with `result --full` or `wait --full`. Foreground timelines are off unless `--live` is passed. Every prompt asks for a dense report as long as needed (soft target 40 lines), under Outcome; Changes (every path); Verification (each command and actual result); Problems and deviations; Open questions/decisions for the user. Failures and deviations must never be omitted for space. Mechanical activity also reports touched files, failed commands/output tails, warning/error events, timeout/interruption, observed sub-agent models/efforts, tokens and quota. `log --failed`, `log --commands` and `log --grep PATTERN` retrieve targeted evidence; a truncation marker requires fetching `result --full` before concluding.

Dispatch defaults: `subagents=auto`, `max_subagents=20` (integer 1..20). `run`, `resume` and `steer` accept `--subagents none|auto|active` and `--max-subagents N`; parallel task JSON accepts `subagents` and `max_subagents`. `none` forbids spawning; `active` requests independent parallel sub-agents, model routing and verified integration; both apply to all roles even when `team_policy=false`. `auto` keeps the optional team policy for worker/debugger/architect when enabled. Native review has no prompt and warns that none/active is ignored. Dispatch is a per-task prompt choice and does not change the session model/effort. The activity digest counts distinct observed sub-agent threads (including zero when active).
