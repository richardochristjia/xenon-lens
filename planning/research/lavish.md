# Lavish assessment for XL-002

- **Accessed:** 2026-09-17
- **Requested repository:** <https://github.com/kunchenguid/lavish>
- **Canonical repository returned by GitHub:** [`kunchenguid/lavish-axi`](https://github.com/kunchenguid/lavish-axi)
- **Inspected commit:** [`4413dcc8eff35cdc659e2035b94194d3c9be55fa`](https://github.com/kunchenguid/lavish-axi/commit/4413dcc8eff35cdc659e2035b94194d3c9be55fa)
- **Inspected release/package:** [`lavish-axi-v0.1.71`](https://github.com/kunchenguid/lavish-axi/releases/tag/lavish-axi-v0.1.71), [`lavish-axi@0.1.71`](https://www.npmjs.com/package/lavish-axi/v/0.1.71)

This report uses current upstream documentation, source, package metadata, git history, releases, CI metadata, and license text. All source permalinks pin the inspected commit. Capabilities already implemented by Lavish are prior art and comparison baselines; this report does **not** claim them as Xenon Lens innovations.

## Executive answer

### Verified facts

Lavish is a functioning local-first review loop for agent-authored HTML. An agent invokes a Node CLI with an HTML file; a local server opens browser chrome around a sandboxed artifact; a person annotates elements or text, uses artifact controls, edits supported diagrams, or writes conversation feedback; feedback remains queued until submission; and an agent receives submitted prompts through a long-running CLI poll. Lavish also implements live reload, browser conversation history, image attachments, layout diagnostics, export, optional third-party sharing, phone-oriented UI, and skill/hook/plugin discovery. [README overview and workflow](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/README.md#how-it-works)

Lavish declares a deliberately narrow domain: “one person and one agent over one local HTML file.” It does not launch or supervise the agent, is not an MCP server, and treats its CLI as the agent interface. [VISION scope](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/VISION.md#scope), [README Agent Plugin section](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/README.md#agent-plugin)

### Interpretation for Xenon Lens

Lavish is substantial prior art, a serious benchmark, and a plausible dependency or complement. It is not a drop-in implementation of Xenon Lens's present domain model. Current Lavish documentation/source do not expose a collaboration-session aggregate spanning multiple artifacts, originating-agent entitlement or explicit agent handoff, a separately modelled human acceptance event, or a versioned stable-target contract. These structural differences should drive the later reuse decision more than overlapping UI features.

## Capabilities and implementation evidence

### Annotations and target context

**Verified facts.** Lavish supports clicking elements and selecting text. Element context contains a runtime UID, generated CSS selector, tag, and bounded visible text. Table annotations can add semantic row/column context, and Mermaid clicks can become diagram-node targets. Text selection adds a `text-range` target with common-ancestor selector plus start/end node paths and offsets. [element selector/context source](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/src/artifact-sdk.js#L772-L834), [text-range source](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/src/artifact-sdk.js#L1081-L1137), [README precise-target behavior](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/README.md#how-it-works)

**Interpretation.** This is rich feedback context, but not the same guarantee as a Lens `Target`. Runtime UIDs restart per document load; generated selectors depend on DOM structure. Lavish guidance recommends stable visible IDs for diagrams and tracked input items, but no public `data-lavish-target`-style identity contract or cross-revision target migration guarantee was found. [diagram/input playbooks](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/src/playbooks.js#L9-L39)

### Feedback queue and explicit submission

**Verified facts.** `window.lavish.queuePrompt()` constructs a queued item from artifact/target context. Optional fields include caller-provided `uid`, `selector`, `tag`, `text`, typed `target`, structured context appended to prompt text, and attachment references. The SDK also exposes `sendQueuedPrompts()` and `endSession()`. [artifact API implementation](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/src/artifact-sdk.js#L1203-L1242), [published `window.lavish` surface](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/src/artifact-sdk.js#L2445-L2478)

The browser chrome stores queued prompts in per-session `sessionStorage`, assigns prompt identities, replaces pre-send updates sharing a queue key, and renders queued items separately from accepted conversation history. A free-form composer message joins the same queue. Clicking **Send to Agent**, pressing Enter in the composer, or an artifact call to `sendQueuedPrompts()` snapshots optional DOM context and posts the whole batch to `/api/:key/prompts`; **Send & End** submits one terminal batch and ends the session. Failed sends preserve the queue. [queue persistence](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/src/chrome-client.js#L1-L20), [queue sanitization/persistence](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/src/chrome-client.js#L340-L470), [submission pipeline](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/src/chrome-client.js#L1356-L1868), [send controls](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/src/chrome-client.js#L3817-L3912)

Server acceptance is atomic for attachment errors and rejects late feedback after session end. Accepted prompt shape is normalized to `uid`, `prompt`, `selector`, `tag`, `text`, optional target, and server-resolved attachment metadata. Submitted prompts are persisted and appended to bounded conversation history before a poll consumes them. [prompt route](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/src/server.js#L812-L887), [normalization](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/src/session-store.js#L743-L797)

**Important boundary.** Lavish's vision says only deliberate human browser action should become feedback. The implementation verifies that artifact messages come from the current sandboxed frame and current artifact-load token, but the artifact API itself can programmatically call both `queuePrompt()` and `sendQueuedPrompts()`; the chrome message handler does not verify a browser user-activation signal before queue/send dispatch. [VISION human-action rule](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/VISION.md#nothing-interrupts-the-human), [chrome message dispatch](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/src/chrome-client.js#L3599-L3678)

**Interpretation.** Lavish uses explicit send UX and author guidance, but Xenon Lens should not treat an artifact-originated submitted prompt as cryptographic proof of human intent when artifact scripts are untrusted. A complementary integration would need either to accept this trust model or add a Xenon-owned confirmation boundary.

### Waiting, long polling, and agent workflow

**Verified facts.** `lavish-axi poll <html-file>` resolves the canonical file path and performs an HTTP GET to `/api/poll?file=...`. Without a debug timeout it waits indefinitely, writes only wait guidance to stderr, keeps stdout for final structured output, and instructs agents to keep the poll in the foreground unless the harness has a tracked completion facility guaranteed to resume/notify the same agent. Codex receives specific foreground-poll guidance. [CLI poll implementation](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/src/cli.js#L337-L433)

The server first checks persisted feedback, then holds the response with whitespace heartbeats. It wakes for submitted feedback, session end, or browser disconnect after a grace period. Feedback delivery consumes the pending batch; if the HTTP client closes before delivery completes, the server attempts to restore the batch and wake another poll. [server poll route](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/src/server.js#L689-L805), [closed-poll restoration](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/src/server.js#L422-L480), [store delivery semantics](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/src/session-store.js#L559-L615)

Agent-facing poll states include `feedback`, `ended`, `browser_disconnected`, timeout/waiting, and missing-session error. Feedback output includes prompts, optional fatal artifact failures, optional terminal-session attribution, a DOM snapshot, and generated next-step guidance. After applying feedback, an agent can send a concise browser response with `--agent-reply` or a structured longer reply with `--agent-reply-file`, then continue polling. [poll output mapping](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/src/cli.js#L436-L528)

**Interpretation.** The CLI is harness-neutral at process level, but continuity depends on harness/supervisor behavior. Lavish does not identify or authorize an originating agent; whichever capable process invokes poll for the canonical file can receive feedback. Browser-tab “take over” and load tokens protect current browser/revision flows, not Xenon Lens agent ownership or handoff.

### Live reload and revision handling

**Verified facts.** Chokidar watches the HTML file by default. Artifacts opt into parent-directory watching for sibling assets with `data-lavish-live-reload-root` or matching meta tag. Change events are debounced and broadcast to browser clients. [watcher implementation](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/src/server.js#L2162-L2220)

The chrome begins each artifact load through a server-issued artifact revision and load token, resets iframe source, and rejects stale frame messages. Across reloads it preserves iframe scroll, open annotation text, Lavish-owned question state, queued prompts, terminal reservation, warning selections, and certain whiteboard state; application-owned arbitrary state is not preserved. [README live reload](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/README.md#how-it-works), [artifact load routes](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/src/server.js#L1183-L1261)

**Interpretation.** `artifact_revision` is useful implementation state for stale-load and diagnostic ordering. It is not documented as a durable, public Artifact Revision resource with immutable content identity or retrieval semantics.

### Artifact handling and isolation

**Verified facts.** Saved HTML remains source of truth. Server reads it and injects one `/sdk.js` tag before `</body>` (or appends it). Local sibling assets are served relative to artifact directory. Artifact asset resolution uses real paths and rejects traversal/symlink escape outside that root. [HTML transform](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/src/html-transform.js), [artifact routes](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/src/server.js#L1226-L1283), [artifact-ownership principle](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/VISION.md#the-artifact-stays-the-authors)

Browser chrome places artifact HTML in an iframe with scripts, forms, popups, popup escape, and downloads allowed, but omits `allow-same-origin`. Artifact↔chrome communication uses `postMessage`; current-load tokens reject stale messages. Host allowlisting defends against DNS rebinding, mutating routes check Origin/Referer, attachments are resolved server-side instead of trusting client paths, and the top-level review chrome denies framing. [iframe sandbox source](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/src/server.js#L2519-L2520), [session framing policy](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/src/server.js#L1141-L1177), [network/Host documentation](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/README.md#how-it-works)

Export creates one HTML file by inlining local assets, removing the annotation SDK, leaving remote references remote, applying size caps, and redacting unsafe absolute `file:` paths. Optional `share` sends the exported content to third-party `ht-ml.app`, public by default, or a compatible configured backend. [README export/share behavior](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/README.md#how-it-works), [self-hosted share contract](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/docs/self-hosting-share.md)

**Privacy caveat—verified fact.** Core artifact/feedback content remains local unless sharing is invoked, but official release builds include command-usage telemetry unless `LAVISH_AXI_TELEMETRY=0`, `false`, or `off`. Source payload includes command/event, success/error, version, platform, and architecture; it does not include artifact path, artifact content, or feedback text. [telemetry source](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/src/telemetry.js), [release build configuration](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/.github/workflows/release-please.yml#L40-L53)

### Persistence

**Verified facts.** Durable server state defaults to `~/.lavish-axi/state.json`, configurable with `LAVISH_AXI_STATE_DIR`; server log, attachments, and whiteboard sidecars live under the same state root. Session identity is a 16-hex-character prefix of SHA-256 over the canonical real path. [paths](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/src/paths.js#L127-L143), [session key](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/src/session-store.js#L718-L726)

Persisted session fields include file, URL, status, queued prompts, prompt count, layout warnings, artifact revision/failures, delivered attachment retention, DOM snapshot, bounded conversation history, and acknowledgement IDs. One in-process mutex serializes state read/modify/write and attachment lifecycle operations. State is rewritten as formatted JSON. [session construction/store](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/src/session-store.js#L48-L140), [state read/write](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/src/session-store.js#L692-L716)

Unsubmitted browser queue and drafts are browser `sessionStorage`, not yet server state; after explicit submission, accepted feedback and transcript state become server-persisted. This distinction matters for recovery across tab closure versus page reload.

### Additional collaboration capabilities

**Verified facts.** Current implementation also includes:

- conversation history and Markdown-subset agent replies;
- PNG/JPEG/WebP attachments with count, byte, disk, rate, and lifecycle limits;
- passive viewport layout-warning inbox and user-selected repair requests;
- fatal artifact-load failures that can wake an agent without user queueing;
- Mermaid-to-Excalidraw editable whiteboards with local scene/preview sidecars;
- mobile conversation sheet and optional Tailscale phone URL;
- user-ended versus agent-ended attribution and guarded reopen;
- multiple separate file-keyed sessions served by one background server.

Sources: [README detailed behavior](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/README.md#how-it-works), [server event/session architecture](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/src/server.js#L276-L388).

## Architecture

### Verified facts

```text
agent harness
  └─ lavish-axi CLI / AXI structured output
       ├─ open/resume/end/export/share/setup
       ├─ HTTP long poll for submitted feedback
       └─ optional HTTP agent reply

single local Node server
  ├─ Express HTTP JSON routes
  ├─ WebSocket live browser events (legacy SSE upgrade path)
  ├─ Chokidar live reload
  ├─ JSON session store under ~/.lavish-axi
  ├─ attachment and whiteboard sidecars
  └─ one session key per canonical HTML file path

browser chrome
  └─ sandboxed artifact iframe
       ├─ saved HTML plus one injected SDK script
       ├─ annotations, controls, snapshots, diagnostics
       ├─ window.lavish API
       └─ postMessage bridge to trusted chrome
```

The npm package is ESM JavaScript requiring Node `>=22`. It publishes a single `lavish-axi` binary plus built assets, plugin manifest, public skill, README, license, and notices. Runtime dependencies include `axi-sdk-js`, Express, WebSocket, Chokidar, `open`, and Parse5; whiteboard/design assets are bundled at build time. No package `exports` or supported library entry point is declared. [package metadata](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/package.json), [build script](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/scripts/build.js)

### Interpretation

Architecture is a deployable CLI application, not a reusable runtime package. Reusing source modules directly means depending on internal coupling among CLI, Express routes, browser chrome, serialized SDK helpers, JSON schema, and sidecar stores. A complement should prefer process-level CLI/artifact contracts. A fork can stabilize those internals but inherits their maintenance.

## Protocol inventory and stability

### Publicly documented surfaces

1. **CLI/AXI:** `open`, `poll`, `end`, `export`, `share`, `stop`, `playbook`, `design`, hook setup, plugin setup, and SDK-provided update. CLI returns structured output and embeds next-step instructions. [CLI reference](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/README.md#cli-reference)
2. **Artifact authoring:** standard HTML plus `window.lavish.queuePrompt`, `sendQueuedPrompts`, `endSession`, `setStatus`, and `snapshot`; `data-lavish-question`, `data-lavish-action`, and optional live-reload markers. [input playbook](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/src/playbooks.js#L187-L229)
3. **Discovery packages:** Agent Skill stub, SessionStart hook installers, and Agent Plugin manifest. [public skill](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/skills/lavish/SKILL.md), [plugin manifest](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/plugin.json)
4. **Share backend:** documented `POST /v1/sites` and `PUT /v1/sites/{site_id}` contract. [share contract](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/docs/self-hosting-share.md#contract)

### Internal surfaces

Express review routes, WebSocket event types, SDK↔chrome `postMessage` messages, persisted `state.json`, browser storage keys, attachment sidecars, and individual source modules are visible and extensively tested but are not published as a versioned protocol or library API. The package remains `0.1.x`; no compatibility or semantic-versioning promise was found.

### Interpretation

A direct CLI/artifact integration is supportable but still needs a pinned Lavish version and black-box contract tests. Depending on internal routes or state should be treated like maintaining a fork. Parsing prose from generated `next_step` guidance is especially brittle; integration should consume only observed structured fields and fail visibly when shape changes.

## CLI and agent integration seams

| Seam | Verified scope | Maintenance implication |
| --- | --- | --- |
| Direct CLI | Any capable agent can invoke through `npx -y lavish-axi`; file path is session identity. | Best-supported runtime seam. Requires poll process continuity and version pinning/testing. |
| Public Agent Skill | Short MIT skill redirects agent to current CLI help/design/playbooks so installed instructions do not stale. | Strong complementary/discovery seam; intentionally not a stable workflow spec itself. |
| SessionStart hooks | Opt-in installers for Claude Code, Codex, OpenCode, and GitHub Copilot CLI surface open sessions and guidance. | Useful adapters, but harness-specific configuration and no generic wake callback. |
| Agent Plugin | Registers same skill in VS Code, Cursor, and Copilot CLI; no MCP server. | Packaging/discovery only, not deeper runtime extension. |
| Artifact API | `window.lavish` and `data-lavish-*` conventions collect structured choices and review context. | Useful for generated artifacts; submission-authenticity boundary needs review for untrusted scripts. |
| Share backend | Environment variable redirects sharing to compatible service. | Extension point only for publishing, not collaboration semantics. |
| Upstream contribution | Public PR route with cross-platform CI. | Human PRs must pass `no-mistakes >=1.46.0` attestation gate. [CONTRIBUTING](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/CONTRIBUTING.md) |
| MIT fork/source copy | Full source available. | Legally permissive with notices; technically carries internal-coupling and update burden. |

## Multi-use-case claims

### Verified facts

Lavish's README and skill claim use for plans, comparisons, diagrams, tables, code views/diffs, reports, prototypes, design exploration, and browser review loops. Built-in playbooks exist for `diagram`, `table`, `comparison`, `plan`, `code`, `input`, and `slides`, and guidance says one artifact may combine multiple playbooks. [skill use cases](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/skills/lavish/SKILL.md), [playbook source](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/src/playbooks.js), [README playbook list](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/README.md#cli-reference)

### Interpretation and limits

These are documented intended use cases and authoring guidance, not comparative outcome evidence. No controlled study was found showing equal reliability or value across those use cases. “One artifact combines playbooks” also does not establish one collaboration session containing multiple independently revisioned artifacts.

## Token-efficiency evidence

### Verified facts

README claims TOON output, long polling, and contextual disclosure make Lavish “highly token efficient”; VISION says token efficiency is first-class and guidance should move behind commands only after agents follow pointers. [README AXI claim](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/README.md#quick-start), [VISION token principle](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/VISION.md#every-token-is-spent-on-purpose)

No quantitative token measurements, benchmark corpus, baseline comparison, confidence interval, or raw token logs were found in README, VISION, `docs/`, source, tests, changelog, or repository agent guidance at the inspected commit.

### Classification

**Unmeasured hypothesis:** Lavish's structured compact output, one blocking poll instead of repeated polling, and on-demand playbooks may reduce agent tokens. Direction is plausible; magnitude and total tokens to a useful accepted outcome remain unmeasured. Xenon Lens must not claim savings—or assume it beats Lavish—without the controlled benchmark planned in XL-013.

## Maintenance state

### Verified facts at access date

- Repository created 2026-05-11; pinned main commit and `0.1.71` release published 2026-09-16.
- Local git history contains 215 commits and 71 tags. Monthly commit counts: May 51, June 50, July 42, August 55, September 17 through inspected commit.
- GitHub lists 71 releases from `0.1.1` through `0.1.71`; npm latest is `0.1.71` and requires Node `>=22`.
- Pinned main commit's CI succeeded across configured Ubuntu, macOS, and Windows jobs. [CI definition](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/.github/workflows/ci.yml), [successful run](https://github.com/kunchenguid/lavish-axi/actions/runs/35132413769)
- Git history attributes 112 commits to maintainer name variants, 71 to release automation, and 32 to other author names; outside contributors exist. [contributors](https://github.com/kunchenguid/lavish-axi/graphs/contributors)
- At access, GitHub search showed 196 merged PRs, 13 open PRs, 44 closed issues, and 31 open issues. Counts are volatile snapshots, not quality measures.

### Interpretation

Project is active, rapidly changing, and maintainer-led—not abandoned and not demonstrably stable. Frequent `0.1.x` releases and absence of an explicit compatibility policy increase pinning, regression-test, and upgrade-review cost. Cross-platform CI supports Xenon's OS baseline, but no explicit Chrome/Edge/Firefox support matrix was found; browser portability still needs independent verification.

## License and legally supportable reuse paths

This section summarizes repository license evidence and is **not legal advice**.

### Verified facts

Lavish repository/package uses MIT License, copyright 2026 Kun Chen. Text permits using, copying, modifying, merging, publishing, distributing, sublicensing, selling, and permitting others to do so. Condition: copyright and permission notice must be included in all copies or substantial portions. Software is provided without warranty. [LICENSE](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/LICENSE), [package metadata](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/package.json)

Published bundle includes third-party software/assets. Notices identify Excalidraw, Mermaid converter, Mermaid, React, DaisyUI, and Tailwind assets as MIT; vendored fonts use MIT or SIL Open Font License 1.1. Upstream says notices satisfy listed attribution requirements and warns font licenses should be rechecked when vendored font set changes. [THIRD-PARTY-NOTICES](https://github.com/kunchenguid/lavish-axi/blob/4413dcc8eff35cdc659e2035b94194d3c9be55fa/THIRD-PARTY-NOTICES.md)

No CLA, trademark policy, or explicit name/logo permission was found in inspected root docs. `ht-ml.app` is explicitly a third-party service outside Lavish; repository MIT terms do not establish that service's terms.

### Supportable options from that evidence

- Invoke unmodified Lavish CLI as an external tool/dependency.
- Publish a complementary skill or adapter that invokes documented CLI/artifact surfaces.
- Modify and redistribute a fork while retaining Lavish's MIT notice and applicable third-party notices/licenses.
- Copy substantial source portions while retaining required notice and auditing dependencies/assets brought along.
- Reimplement observed behavior independently without copying Lavish code; branding, patents, and other rights remain separate questions.

**Interpretation.** MIT removes a major copyright barrier to fork/selective reuse, but not technical maintenance cost. Absence of explicit trademark terms means branding should be treated separately rather than inferred from the software license. Hosted sharing should be omitted, separately reviewed, or replaced through the documented backend contract.

## Evidence matrix for later reuse decision

No final reuse posture is selected by XL-002.

| Strategy | Verified evidence in favor | Interpretation / risk | Assumption requiring validation |
| --- | --- | --- | --- |
| **Extend Lavish upstream** | Strong capability overlap; active maintainer; contribution path; MIT. | Lowest duplication if requirements fit. Multi-artifact collaboration sessions, originating-agent ownership/handoff, explicit acceptance, and stronger target identity appear outside current declared one-file scope. | Upstream confirms those requirements fit `VISION.md` and will support needed public contracts. |
| **Complementary skill/adapter** | CLI is intentional agent interface; skill/hooks/plugins already demonstrate portable discovery; artifact API can collect structured input. | Strongest current seam: let Lavish own one-file review while Xenon owns session grouping, agent entitlement, acceptance, and orchestration. Dual lifecycle and poll resumption may confuse or fail without careful adapter boundaries. | Pinned black-box tests pass in Pi and another harness; Xenon can add its semantics without internal routes/state or misleading UX. |
| **Selective reuse or fork** | MIT permits modification/reuse with notices; source contains mature recovery, isolation, attachment, export, mobile, and diagnostic work. | No reusable package API; modules are coupled and upstream changes quickly. Fork inherits security fixes, browser compatibility, release tooling, notices, and merge burden. | Measured implementation savings exceed long-term synchronization and audit cost; exact copied dependency/asset inventory is acceptable. |
| **Standalone Xenon Lens** | Full control over collaboration-session, originating-agent, acceptance, target, and protocol contracts. | Avoids dependency/internal churn but duplicates substantial hardened behavior. “Different domain model” is not enough unless implementation/operation advantage is measured. | A thin standalone proof meets recovery/isolation requirements and compares favorably on setup, tokens, retries, quality, and maintenance. |

## Required follow-up evidence

1. Ask upstream whether multi-artifact session grouping, explicit agent ownership/handoff, stable author target identity, and human acceptance belong in Lavish scope.
2. Black-box pinned `0.1.71` in Pi: open, annotate, structured choice, queue multiple items, explicitly submit, poll, reply, live reload, interrupted delivery, user end.
3. Repeat core loop in at least one other harness and verify poll completion resumes the same originating agent.
4. Test hostile Lens-native artifact behavior around `queuePrompt`, `sendQueuedPrompts`, navigation, popups, attachments, and network access before accepting Lavish's intent boundary.
5. Prototype only Xenon-specific state outside Lavish to see whether one collaboration session can coherently group multiple file-keyed Lavish sessions.
6. Inventory every source file, dependency, built asset, font, and notice proposed for selective reuse/fork.
7. Benchmark complementary and standalone approaches against Lavish on identical tasks. Measure total tokens to useful accepted outcome, retries, context requests, result quality, setup cost, and recurring cost.

## Bottom line

**Verified fact:** Lavish already implements most mechanics of local browser-based HTML review and is active, permissively licensed, and operationally sophisticated.

**Interpretation:** Treat Lavish as benchmark and first integration candidate, not as an automatically compatible foundation. Complementary use is presently best supported by public seams. Upstream extension depends on scope confirmation. Fork/selective reuse is legally plausible but technically expensive. Standalone development needs measured justification rooted in Xenon Lens's multi-artifact session, originating-agent, stable-target, acceptance, or trust requirements.
