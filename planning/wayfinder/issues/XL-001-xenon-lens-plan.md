---
id: XL-001
title: Chart an implementation-ready plan for Xenon Lens
status: open
labels:
  - wayfinder:map
assignee: null
---

## Destination

An evidence-backed, implementation-ready Xenon Lens plan containing all requested planning deliverables, reviewed and explicitly approved by the human. No production implementation code belongs to this effort.

## Notes

Domain: local, harness-independent collaboration through interactive HTML artifacts. Consult `grilling` and `domain-modeling` for HITL decisions; use `research` against primary sources for external facts. Prefer a shared runtime and CLI, portable instructions, and optional thin adapters. Baselines accepted during charting: Windows/macOS/Linux; current desktop Chrome/Edge/Firefox; Lens-native artifacts; one originating agent per session; Pi first-loop validation; artifact content untrusted by default. Label final claims as verified fact, assumption, or unmeasured token-saving hypothesis. Planning only until explicit approval.

## Decisions so far

<!-- Closed ticket links and one-line gists go here. -->

- [Assess Lavish capabilities, architecture, and reuse constraints](../resolutions/XL-002.md) — Lavish already provides mature one-file HTML collaboration under MIT; complementary use has strongest public seam, while extension needs scope confirmation and standalone work needs measured justification.
- [Verify harness waiting, delivery, and resumption capabilities](../resolutions/XL-003.md) — Shared CLI works across verified harness capabilities, but indefinite automatic waits do not; use capability-selected foreground/event/manual-resume profiles, with no MVP need for MCP.
- [Establish browser isolation and local transport constraints](../resolutions/XL-004.md) — Use trusted parent chrome plus opaque-origin script sandbox, validated bridge, parent-owned queue/authority, loopback authorization, explicit state hooks, and conservative text-anchor staleness.

## Not yet specified

- Packaging, installation, and update path depend on reuse and runtime choices.
- Lavish migration or contribution mechanics, if relevant, depend on reuse recommendation and license findings.

## Out of scope

- Production implementation during this effort.
- Dedicated diagram editors, visual CSS editors, automatic session replay, multiplayer collaboration, and arbitrary production-site support.
- Cloud accounts or hosted infrastructure required for core loop.
- Browser feedback implicitly authorizing deployments, external messages, or other consequential actions.
