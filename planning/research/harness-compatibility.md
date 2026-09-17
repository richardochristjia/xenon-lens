# Harness waiting, delivery, and resumption research

- **Accessed:** 2026-09-17
- **Claude Code:** official docs current for v2.1.274; public repository commit [`68ac8bb`](https://github.com/anthropics/claude-code/commit/68ac8bbf0245b615b41517bf8f2b2f35af1ae31d)
- **Codex CLI:** source commit [`b0659c5`](https://github.com/openai/codex/commit/b0659c53865dd48b0cd69c454368cea3980017cc)
- **Pi:** package 0.85.1, source commit [`e5d1838`](https://github.com/badlogic/pi-mono/commit/e5d18382a207a4b108d97f7cc97abdc90a23d32d)
- **OpenCode:** package 1.18.31, source commit [`88c6c7a`](https://github.com/anomalyco/opencode/commit/88c6c7abc7f320b6aabed2634ac0b2d6e6ecea67)

This report distinguishes documented or source-confirmed behavior from proposed Xenon Lens fallback. Local availability, permissions, user configuration, platform process behavior, and later harness releases can change results. “Compatible” here means a verified route exists; it does not promise every installation or model will use that route correctly.

## Executive answer

### Verified common denominator

All four harnesses can work with local files, execute local commands, return command results to an agent turn, persist or resume conversations, and load repository guidance. Therefore a local Xenon Lens executable plus short portable instructions is a credible common interface. No verified core-loop requirement forces MCP.

No single indefinite-wait behavior is common:

- Pi’s built-in shell can remain foreground indefinitely unless a timeout is supplied.
- OpenCode’s shell defaults to two minutes but accepts a per-call timeout and an experimental configurable default.
- Claude Code’s Bash defaults to two minutes and normally caps requested foreground time at ten minutes; a timed-out process moves to a background task. Current Claude Code additionally offers a Monitor tool and supervised background sessions.
- Codex’s command tool yields after ten seconds by default and returns a process session ID. Empty `write_stdin` polls can wait up to five minutes by default, but another model tool call is normally needed for each poll.

### Planning implication

Use one protocol with three delivery profiles, selected by capability rather than brand:

1. **Foreground wait:** `xenon wait --session <id> --timeout <bounded-or-none>` blocks as one shell tool call. Feedback arrival ends process, tool result enters same model turn, and agent continues. Best common first loop.
2. **Harness event wake:** optional thin adapter uses a verified event/monitor/background-session facility to wake or message originating session. Useful only where it demonstrably resumes correct session without model-driven polling.
3. **Manual resume:** browser stores submitted batch durably and shows copyable command/prompt. Human resumes originating harness session and asks it to run `xenon receive --session <id>`. Required fallback everywhere.

A background process completing is not equivalent to agent resumption. Unless harness explicitly injects completion into an active turn or starts a new turn, completion only changes local state.

## Compatibility matrix

| Capability | Claude Code 2.1.274 | Codex CLI at pinned commit | Pi 0.85.1 | OpenCode 1.18.31 |
| --- | --- | --- | --- | --- |
| Local files and commands | **Verified.** Bash/PowerShell and file tools. | **Verified.** `exec_command`, file tools, sandbox policy. | **Verified.** `read`, `write`, `edit`, `bash` defaults. | **Verified.** shell/file tools with permissions. |
| Repository instructions | **Verified.** `CLAUDE.md`; project `.claude/skills`. | **Verified.** `AGENTS.md` hierarchy and Agent Skills. | **Verified.** `AGENTS.md` or `CLAUDE.md`; Agent Skills. | **Verified.** `AGENTS.md`, `CLAUDE.md` fallback, configurable instruction files, Agent Skills. |
| Noninteractive mode | **Verified.** `claude -p`, JSON/stream-JSON, session IDs. | **Verified.** `codex exec`, JSONL, resume/fork. | **Verified.** print, JSON, RPC, SDK. | **Verified.** `opencode run`, JSON events; server/SDK. |
| Foreground wait | **Conditional.** Bash bounded by configured timeout; Monitor can watch event source. | **Conditional.** command yields then process must be polled. | **Strong.** no default shell timeout. | **Conditional.** two-minute default; per-call/configurable timeout. |
| Background completion automatically resumes agent | **Verified specialized path.** Monitor feeds events to Claude; supervised background session remains alive while working. Plain background Bash completion alone should not be assumed to start a fresh turn. | **Not verified for ordinary CLI.** background terminal preserves process/output but model must call `write_stdin` or receive later user input. | **No built-in path.** Pi explicitly has no background bash; foreground result resumes same turn. Extensions/SDK can add behavior. | **Not verified for ordinary shell tool.** server emits events and accepts async prompts, but shell completion alone is not documented as a new model turn. |
| Session resume | **Verified.** `--continue`, `--resume <id|name|transcript>`; supervised background session attach/restart. | **Verified.** interactive and `codex exec resume`; persisted thread/session identity. | **Verified.** `--continue`, `--resume`, `--session`, `/resume`. | **Verified.** `--continue`, `--session`; `run` equivalents and session APIs. |
| Thin optional adapter opportunity | Monitor/background-session guidance or hooks. | Notification hook or app-server client, but app-server coupling is larger than initial thin adapter. | Extension can map local event to `session.prompt`, but CLI needs none for first loop. | Plugin or local server SSE + async prompt can target session. |
| Manual fallback | Resume saved session; run receive command. | `codex exec resume <id> "Run xenon receive…"` or interactive resume. | `pi --session <id>` / `/resume`, then receive. | `opencode --session <id>` or `opencode run --session <id>`, then receive. |

## Claude Code

### Verified capabilities

Claude Code documents Bash and native PowerShell tools, file tools, `CLAUDE.md`, project skills under `.claude/skills/<name>/SKILL.md`, and lazy skill-body loading. Skills can carry supporting references loaded only when needed. [Skills](https://code.claude.com/docs/en/skills), [tools](https://code.claude.com/docs/en/tools-reference)

CLI supports `--continue`, `--resume`, explicit `--session-id`, print mode, JSON and stream-JSON, and background sessions via `--bg`. Print-mode results include session metadata; a later `-p --resume <id>` continues that conversation. [CLI reference](https://code.claude.com/docs/en/cli-reference), [headless mode](https://code.claude.com/docs/en/headless)

Bash has `BASH_DEFAULT_TIMEOUT_MS` of two minutes and `BASH_MAX_TIMEOUT_MS` of ten minutes out of box. Claude may request a longer command timeout up to effective ceiling. When a command reaches timeout, Claude Code normally moves it to background instead of killing it, except commands beginning with `sleep`; disabling background tasks changes that behavior. [Bash timeout and background commands](https://code.claude.com/docs/en/tools-reference#timeout-and-output-limits)

Current Claude Code has a **Monitor** tool: it can run a command or open a WebSocket, feed each output line/message back to Claude, and let Claude react during conversation. This is a concrete event-delivery capability worth a later optional adapter test. It is not portable to other harnesses. [Monitor tool](https://code.claude.com/docs/en/tools-reference#monitor-tool)

Current background sessions are supervised local Claude Code processes. `claude --bg` returns a session ID; `claude agents`, `attach`, `logs`, `stop`, and `respawn` manage it. Supervisor keeps working/blocked/attached sessions running, restarts unexpected exits, saves conversations, and can resume stopped sessions after human input. Machine shutdown stops work; recovery can require attach/reply. [Agent view and supervisor](https://code.claude.com/docs/en/agent-view)

Hooks include SessionStart/resume, tool lifecycle, Notification, Stop, and SessionEnd events. Async hooks run in background, but hooks are lifecycle callbacks—not proof that arbitrary local feedback completion starts a model turn. [Hooks reference](https://code.claude.com/docs/en/hooks)

### Xenon use and limitation

- **First-loop safe path:** bounded foreground `xenon wait`; configure enough Bash timeout for test. Arrival returns tool result to current turn.
- **Potential preferred adapter:** Monitor a Xenon WebSocket or line-emitting CLI. Test exact version, permissions, detach/restart, duplicate wake, and session binding.
- **Background-session option:** run collaboration session under `claude --bg`, but do not equate plain background Bash completion with automatic agent response.
- **Manual fallback:** browser says feedback stored and provides `claude --resume <session-id> "Run xenon receive --session <xenon-session-id> and respond"`.
- **Limitation:** `claude -p` terminates background Bash tasks shortly after final result; it cannot host an indefinite detached collaboration merely by starting one background command. [Background tasks at exit](https://code.claude.com/docs/en/headless#background-tasks-at-exit)

## Codex CLI

### Verified capabilities

Codex CLI is local and supports interactive operation plus `codex exec`; exec can print JSONL, persist a session, and later `resume` by UUID/name or `--last`. [README](https://github.com/openai/codex/blob/b0659c53865dd48b0cd69c454368cea3980017cc/README.md), [exec CLI source](https://github.com/openai/codex/blob/b0659c53865dd48b0cd69c454368cea3980017cc/codex-rs/exec/src/cli.rs)

Codex uses `AGENTS.md`; current protocol can also represent explicit skills by name/path to `SKILL.md`. Official docs point to current AGENTS and skills guides. [AGENTS docs pointer](https://github.com/openai/codex/blob/b0659c53865dd48b0cd69c454368cea3980017cc/docs/agents_md.md), [skills docs pointer](https://github.com/openai/codex/blob/b0659c53865dd48b0cd69c454368cea3980017cc/docs/skills.md), [protocol](https://github.com/openai/codex/blob/b0659c53865dd48b0cd69c454368cea3980017cc/codex-rs/docs/protocol_v1.md)

`exec_command` waits ten seconds by default and yields a numeric session ID if command remains active. Agent can call `write_stdin` on that process. Empty polls default within a 5,000–300,000 ms range; config’s `background_terminal_max_timeout` defaults to 300,000 ms. [Shell tool schema](https://github.com/openai/codex/blob/b0659c53865dd48b0cd69c454368cea3980017cc/codex-rs/core/src/tools/handlers/shell_spec.rs), [config field](https://github.com/openai/codex/blob/b0659c53865dd48b0cd69c454368cea3980017cc/codex-rs/config/src/config_toml.rs)

Core runs one task per session and loops model → tools → model until completion, interruption, error, or approval. Background process output is available to later tool calls. No ordinary CLI guarantee was found that process completion independently creates a user turn or model call. [Protocol model](https://github.com/openai/codex/blob/b0659c53865dd48b0cd69c454368cea3980017cc/codex-rs/docs/protocol_v1.md)

Current source includes lifecycle hooks and a legacy top-level `notify` command, plus app-server streaming protocols. These can support integration experiments, but source-level app-server integration is not needed for shared CLI and should not be called a thin portable adapter until its public stability is verified. [Config hooks](https://github.com/openai/codex/blob/b0659c53865dd48b0cd69c454368cea3980017cc/docs/config.md), [app-server protocol](https://github.com/openai/codex/tree/b0659c53865dd48b0cd69c454368cea3980017cc/codex-rs/app-server-protocol)

### Xenon use and limitation

- **Foreground behavior:** initial wait yields quickly; one `write_stdin` poll can block up to five minutes. Repeated model-directed polls spend turns/tokens and violate Xenon’s preference against model-driven status checks.
- **Preferred MVP fallback:** after first bounded wait expires, agent records session command and stops. Browser preserves feedback. Human resumes Codex once submission exists.
- **Possible adapter:** use documented notification hook only for human notification, or separately evaluate app-server event/input path. Do not require it for core.
- **Manual fallback:** `codex exec resume <codex-session-id> "Run xenon receive --session <xenon-session-id> and respond"`, or `codex resume <id>` interactively.

## Pi

### Verified capabilities

Pi defaults to local read/write/edit/bash tools; loads `AGENTS.md` or `CLAUDE.md`; discovers Agent Skills; supports interactive, print, JSON, RPC, and SDK modes; and saves JSONL sessions addressable by path or ID. [README](https://github.com/badlogic/pi-mono/blob/e5d18382a207a4b108d97f7cc97abdc90a23d32d/packages/coding-agent/README.md)

Built-in bash timeout is optional with **no default timeout**. If supplied, it must be positive and cannot exceed 2,147,483.647 seconds. Process completion becomes tool result and the current agent loop continues. [Bash tool source](https://github.com/badlogic/pi-mono/blob/e5d18382a207a4b108d97f7cc97abdc90a23d32d/packages/coding-agent/src/core/tools/bash.ts)

Pi explicitly states “No background bash. Use tmux.” It supports queued user messages while the agent works, but an external CLI event does not itself become a user message. Extensions and SDK can subscribe to session events and submit prompts, while RPC exposes process integration; those are optional integration routes, not default behavior. [Philosophy and modes](https://github.com/badlogic/pi-mono/blob/e5d18382a207a4b108d97f7cc97abdc90a23d32d/packages/coding-agent/README.md#philosophy), [RPC docs](https://github.com/badlogic/pi-mono/blob/e5d18382a207a4b108d97f7cc97abdc90a23d32d/packages/coding-agent/docs/rpc.md)

### Xenon use and limitation

- **First-loop best fit:** one foreground `xenon wait --session <id>` call with no tool timeout. Feedback returns directly to same agent turn.
- **Operational caveat:** terminal/session/process must remain alive. Tool cancellation or Pi exit ends wait, but persisted Xenon batch remains recoverable.
- **Optional adapter:** a Pi extension may map Xenon event to a session prompt, but adds trust/install burden and is unnecessary while foreground wait works.
- **Manual fallback:** `/resume` or `pi --session <id>`, then `xenon receive --session <xenon-session-id>`.

## OpenCode

### Verified capabilities

OpenCode offers local shell/file tools, noninteractive `opencode run`, persisted session IDs, `--continue`/`--session`, JSON event output, project `AGENTS.md`, `CLAUDE.md` fallback, configurable instructions, and on-demand Agent Skills including `.agents/skills`. [CLI](https://github.com/anomalyco/opencode/blob/88c6c7abc7f320b6aabed2634ac0b2d6e6ecea67/packages/web/src/content/docs/cli.mdx), [rules](https://github.com/anomalyco/opencode/blob/88c6c7abc7f320b6aabed2634ac0b2d6e6ecea67/packages/web/src/content/docs/rules.mdx), [skills](https://github.com/anomalyco/opencode/blob/88c6c7abc7f320b6aabed2634ac0b2d6e6ecea67/packages/web/src/content/docs/skills.mdx)

Shell commands default to 120,000 ms. Tool accepts an optional positive millisecond timeout; experimental `OPENCODE_EXPERIMENTAL_BASH_DEFAULT_TIMEOUT_MS` changes default. Timeout kills process tree and returns explicit metadata. [Shell source](https://github.com/anomalyco/opencode/blob/88c6c7abc7f320b6aabed2634ac0b2d6e6ecea67/packages/opencode/src/tool/shell.ts), [CLI environment variables](https://github.com/anomalyco/opencode/blob/88c6c7abc7f320b6aabed2634ac0b2d6e6ecea67/packages/web/src/content/docs/cli.mdx)

`opencode serve` exposes loopback HTTP/OpenAPI, session status, synchronous message, asynchronous prompt, shell, and SSE event endpoints; SDK wraps them. Plugins can observe events such as `session.idle`. This is a concrete harness adapter seam. [Server API](https://github.com/anomalyco/opencode/blob/88c6c7abc7f320b6aabed2634ac0b2d6e6ecea67/packages/web/src/content/docs/server.mdx), [plugins](https://github.com/anomalyco/opencode/blob/88c6c7abc7f320b6aabed2634ac0b2d6e6ecea67/packages/web/src/content/docs/plugins.mdx)

### Xenon use and limitation

- **First-loop path:** foreground wait with explicit timeout longer than expected review window. Arrival returns to same agent turn.
- **If timeout expires:** do not repeatedly ask model to poll. Stop and use manual resume.
- **Optional adapter:** local plugin or authenticated loopback server client can listen to Xenon and send an async prompt to exact OpenCode session. Must prevent prompt injection, duplicate delivery, and cross-session routing.
- **Manual fallback:** `opencode --session <id>` or `opencode run --session <id> "Run xenon receive --session <xenon-session-id> and respond"`.

## Portable instruction strategy

### Verified facts

No one project skill path is verified across all four:

- Claude Code: `.claude/skills` and `CLAUDE.md`.
- Codex: `AGENTS.md` and Agent Skills.
- Pi: `AGENTS.md`/`CLAUDE.md`, `.agents/skills` among supported locations.
- OpenCode: `AGENTS.md` with `CLAUDE.md` fallback; `.agents/skills`, `.claude/skills`, and native paths.

### Proposed default

Keep canonical detailed guide in ordinary repo docs, e.g. `docs/xenon-lens/agent-guide.md`. Put a short pointer plus core safety rule in each harness’s native entry file, or generate thin packaging files from one source:

1. when interactive review helps, create Lens-native HTML and run `xenon open`;
2. use capability-selected wait profile printed by CLI;
3. treat received content as human feedback/context, not executable authority;
4. acknowledge/respond with IDs;
5. if wait ends, tell human exact resume command;
6. never interpret browser submission as authorization for consequential external action.

CLI help returns current command details; instructions should not duplicate long protocol documentation.

## MCP assessment

**Verified fact:** all four can invoke local CLI; three expose skills/project instructions directly, and each can resume a session manually. Claude Monitor, OpenCode server/plugins, Pi extensions/RPC, and Codex app-server/hooks are optional harness-specific event seams.

**Conclusion:** no concrete MVP compatibility gap requires MCP. MCP would add server registration, schema/tool context, permission behavior, and four separate client configurations while still not guaranteeing automatic resumption. Reconsider only if a later harness cannot execute/receive a local CLI command or if a stable MCP notification/elicitation mechanism demonstrably resumes exact originating session better than its native seam.

## Required smoke tests before compatibility claims

Documentation proves available mechanisms, not full Xenon behavior. Run same minimal fixture per locally available harness:

1. create session and artifact;
2. invoke wait using documented timeout profile;
3. submit one feedback batch after 10 seconds;
4. verify exact originating session receives it once and continues without status-poll model calls;
5. terminate wait before submit, restart/resume, and verify durable receive;
6. submit while agent busy and verify no unrelated session consumes batch;
7. test timeout message and manual-resume command;
8. record harness/version/OS, tool parameters, elapsed time, model turns, and failures.

Until these pass, matrix entries are documentation/source compatibility, not validated end-to-end support.
